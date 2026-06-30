<!-- source: website/docs/guia-desarrollador/proveedor-runtime.md -->
# Proveedor Runtime Resolution

# Proveedor Runtime Resolution

Hermes has a shared proveedor runtime resolver used across:

- CLI
- puerta de enlace
- cron jobs
- ACP
- auxiliary modelo calls

Primary implementation:

- `hermes_cli/runtime_proveedor.py` — credential resolution, `_resolve_custom_runtime()`
- `hermes_cli/auth.py` — proveedor registry, `resolve_proveedor()`
- `hermes_cli/modelo_switch.py` — shared `/modelo` switch pipeline (CLI + puerta de enlace)
- `agente/auxiliary_client.py` — auxiliary modelo routing
- `proveedors/` — ABC + registry entry points (`ProveedorPerfil`, `register_proveedor`, `get_proveedor_perfil`, `list_proveedors`)
- `complementos/modelo-proveedors/<name>/` — per-proveedor complementos (bundled) that declare `api_mode`, `base_url`, `env_vars`, `fallback_modelos` and register themselves into the registry on first access. User complementos at `$HERMES_HOME/complementos/modelo-proveedors/<name>/` override bundled ones of the same name.

`get_proveedor_perfil()` in `proveedors/` returns a `ProveedorPerfil` for a given proveedor id. `runtime_proveedor.py` calls this at resolution time to get the canonical `base_url`, `env_vars` priority list, `api_mode`, and `fallback_modelos` without needing to duplicate that data in multiple files. Adding a new complemento under `complementos/modelo-proveedors/<your-proveedor>/` (or `$HERMES_HOME/complementos/modelo-proveedors/<your-proveedor>/`) that calls `register_proveedor()` is enough for `runtime_proveedor.py` to pick it up — no branch needed in the resolver itself.

If you are trying to add a new first-class inference proveedor, read [Adding Proveedors](./adding-proveedors.md) and the [Modelo Proveedor Complemento guide](./modelo-proveedor-complemento.md) alongside this page.

## Resolution precedence

At a high level, proveedor resolution uses:

1. explicit CLI/runtime request
2. `config.yaml` modelo/proveedor config
3. environment variables
4. proveedor-specific defaults or auto resolution

That ordering matters because Hermes treats the saved modelo/proveedor choice as the source of truth for normal runs. This prevents a stale shell export from silently overriding the endpoint a user last selected in `hermes modelo`.

## Proveedors

Current proveedor families include (see `complementos/modelo-proveedors/` for the complete bundled set):

- OpenRouter
- Nous Portal
- OpenAI Codex
- Copilot / Copilot ACP
- Anthropic (native)
- Google / Gemini (`gemini`)
- Alibaba / DashScope (`alibaba`, `alibaba-coding-plan`)
- DeepSeek
- Z.AI
- Kimi / Moonshot (`kimi-coding`, `kimi-coding-cn`)
- MiniMax (`minimax`, `minimax-cn`, `minimax-oauth`)
- Kilo Code
- Hugging Face
- OpenCode Zen / OpenCode Go
- AWS Bedrock
- Azure Foundry
- NVIDIA NIM
- xAI (Grok)
- Arcee
- GMI Cloud
- StepFun
- Qwen OAuth
- Xiaomi
- Ollama Cloud
- LM Studio
- Tencent TokenHub
- Custom (`proveedor: custom`) — first-class proveedor for any OpenAI-compatible endpoint
- Named custom proveedors (`custom_proveedors` list in config.yaml)

## Output of runtime resolution

The runtime resolver returns data such as:

- `proveedor`
- `api_mode`
- `base_url`
- `api_key`
- `source`
- proveedor-specific metadata like expiry/refresh info

## Why this matters

This resolver is the main reason Hermes can share auth/runtime logic between:

- `hermes chat`
- puerta de enlace message handling
- cron jobs running in fresh sessions
- ACP editor sessions
- auxiliary modelo tasks

## OpenRouter and custom OpenAI-compatible base URLs

Hermes contains logic to avoid leaking the wrong API key to a custom endpoint when multiple proveedor keys exist (e.g. `OPENROUTER_API_KEY` and `OPENAI_API_KEY`).

Each proveedor's API key is scoped to its own base URL:

- `OPENROUTER_API_KEY` is only sent to `openrouter.ai` endpoints
- `OPENAI_API_KEY` is used for custom endpoints and as a fallback

Hermes also distinguishes between:

- a real custom endpoint selected by the user
- the OpenRouter fallback path used when no custom endpoint is configured

That distinction is especially important for:

- local modelo servers
- non-OpenRouter OpenAI-compatible APIs
- switching proveedors without re-running setup
- config-saved custom endpoints that should keep working even when `OPENAI_BASE_URL` is not exported in the current shell

## Native Anthropic path

Anthropic is not just "via OpenRouter" anymore.

When proveedor resolution selects `anthropic`, Hermes uses:

- `api_mode = anthropic_messages`
- the native Anthropic Messages API
- `agente/anthropic_adapter.py` for translation

Credential resolution for native Anthropic now prefers refreshable Claude Code credentials over copied env tokens when both are present. In practice that means:

- Claude Code credential files are treated as the preferred source when they include refreshable auth
- manual `ANTHROPIC_TOKEN` / `CLAUDE_CODE_OAUTH_TOKEN` values still work as explicit overrides
- Hermes preflights Anthropic credential refresh before native Messages API calls
- Hermes still retries once on a 401 after rebuilding the Anthropic client, as a fallback path

## OpenAI Codex path

Codex uses a separate Responses API path:

- `api_mode = codex_responses`
- dedicated credential resolution and auth store support

## Auxiliary modelo routing

Auxiliary tasks such as:

- vision
- web extraction summarization
- contexto compression summaries
- habilidads hub operations
- MCP helper operations
- memoria flushes

can use their own proveedor/modelo routing rather than the main conversational modelo.

When an auxiliary task is configured with proveedor `main`, Hermes resolves that through the same shared runtime path as normal chat. In practice that means:

- env-driven custom endpoints still work
- custom endpoints saved via `hermes modelo` / `config.yaml` also work
- auxiliary routing can tell the difference between a real saved custom endpoint and the OpenRouter fallback

## Fallback modelos

Hermes supports a configured fallback proveedor chain — a list of `(proveedor, modelo)` entries tried in order when the primary modelo encounters errors. The legacy single-pair `fallback_modelo` dict is still accepted for back-compat (and migrated on first write).

### How it works internally

1. **Storage**: `AIAgente.__init__` stores the `fallback_modelo` dict and sets `_fallback_activated = False`.

2. **Trigger points**: `_try_activate_fallback()` is called from three places in the main retry loop in `run_agente.py`:
   - After max retries on invalid API responses (None choices, missing content)
   - On non-retryable client errors (HTTP 401, 403, 404)
   - After max retries on transient errors (HTTP 429, 500, 502, 503)

3. **Activation flow** (`_try_activate_fallback`):
   - Returns `False` immediately if already activated or not configured
   - Calls `resolve_proveedor_client()` from `auxiliary_client.py` to build a new client with proper auth
   - Determines `api_mode`: `codex_responses` for openai-codex, `anthropic_messages` for anthropic, `chat_completions` for everything else
   - Swaps in-place: `self.modelo`, `self.proveedor`, `self.base_url`, `self.api_mode`, `self.client`, `self._client_kwargs`
   - For anthropic fallback: builds a native Anthropic client instead of OpenAI-compatible
   - Re-evaluates prompt caching (enabled for Claude modelos on OpenRouter)
   - Sets `_fallback_activated = True` — prevents firing again
   - Resets retry count to 0 and continues the loop

4. **Config flow**:
   - CLI: `cli.py` reads `CLI_CONFIG["fallback_modelo"]` → passes to `AIAgente(fallback_modelo=...)`
   - Puerta de enlace: `puerta de enlace/run.py._load_fallback_modelo()` reads `config.yaml` → passes to `AIAgente`
   - Validation: both `proveedor` and `modelo` keys must be non-empty, or fallback is disabled

### What does NOT support fallback

- **Subagente delegation** (`herramientas/delegate_herramienta.py`): subagentes inherit the parent's proveedor but not the fallback config
- **Auxiliary tasks**: use their own independent proveedor auto-detection chain (see Auxiliary modelo routing above)

Cron jobs **do** support fallback: `run_job()` reads `fallback_proveedors` (or legacy `fallback_modelo`) from `config.yaml` and passes it to `AIAgente(fallback_modelo=...)`, matching the puerta de enlace's `_load_fallback_modelo()` pattern. See [Cron Internals](./cron-internals.md).

### Test coverage

Fallback behavior is exercised across several suites:

- `tests/run_agente/test_fallback_credential_isolation.py` — credential isolation between primary and fallback
- `tests/hermes_cli/test_fallback_cmd.py` — the `/fallback` CLI command
- `tests/puerta de enlace/test_fallback_eviction.py` — puerta de enlace eviction of failed proveedors

## Related docs

- [Agente Loop Internals](./agente-loop.md)
- [ACP Internals](./acp-internals.md)
- [Contexto Compression & Prompt Caching](./contexto-compression-and-caching.md)

---