<!-- source: website/docs/guia-usuario/features/fallback-proveedors.md -->
# guia-usuario/features/fallback-proveedors

# Fallback Proveedors

Hermes Agente has three layers of resilience that keep your sessions running when proveedors hit issues:

1. **[Credential pools](./credential-pools.md)** — rotate across multiple API keys for the *same* proveedor (tried first)
2. **Primary modelo fallback** — automatically switches to a *different* proveedor:modelo when your main modelo fails
3. **Auxiliary task fallback** — independent proveedor resolution for side tasks like vision, compression, and web extraction

Credential pools handle same-proveedor rotation (e.g., multiple OpenRouter keys). This page covers cross-proveedor fallback. Both are optional and work independently.

## Primary Modelo Fallback

When your main LLM proveedor encounters errors — rate limits, server overload, auth failures, connection drops — Hermes can automatically switch to a backup proveedor:modelo pair mid-session without losing your conversation.

### Configuración

The easiest path is the interactive manager:

```bash
hermes fallback
```

`hermes fallback` reuses the proveedor picker from `hermes modelo` — same proveedor list, same credential prompts, same validation. Use the subcommands `add`, `list` (alias `ls`), `remove` (alias `rm`), and `clear` to manage the chain. Changes persist under the top-level `fallback_proveedors:` list in `config.yaml`.

If you'd rather edit the YAML directly, add a top-level `fallback_proveedors` list to `~/.hermes/config.yaml`:

```yaml
fallback_proveedors:
  - proveedor: openrouter
    modelo: anthropic/claude-sonnet-4
```

Each entry requires both `proveedor` and `modelo`. Entries missing either field are ignored.

:::note `fallback_modelo` vs `fallback_proveedors`
`fallback_proveedors` (plural, list) is the current config shape and supports multiple fallbacks tried in order. `fallback_modelo` (singular) is the legacy single-fallback key — Hermes still honors it for back-compat, but `hermes fallback` writes the current `fallback_proveedors` key and migrates legacy config on write. When both are set, `fallback_proveedors` takes priority.
:::

### Supported Proveedors

| Proveedor | Value | Requirements |
|----------|-------|-------------|
| OpenRouter | `openrouter` | `OPENROUTER_API_KEY` |
| Nous Portal | `nous` | `hermes setup --portal` (fresh) or `hermes auth add nous` (OAuth) |
| OpenAI Codex | `openai-codex` | `hermes modelo` (ChatGPT OAuth) |
| GitHub Copilot | `copilot` | `COPILOT_GITHUB_TOKEN`, `GH_TOKEN`, or `GITHUB_TOKEN` |
| GitHub Copilot ACP | `copilot-acp` | External process (editor integration) |
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` or Claude Code credentials |
| z.ai / GLM | `zai` | `GLM_API_KEY` |
| Kimi / Moonshot | `kimi-coding` | `KIMI_API_KEY` |
| MiniMax | `minimax` | `MINIMAX_API_KEY` |
| MiniMax (China) | `minimax-cn` | `MINIMAX_CN_API_KEY` |
| DeepSeek | `deepseek` | `DEEPSEEK_API_KEY` |
| NVIDIA NIM | `nvidia` | `NVIDIA_API_KEY` (optional: `NVIDIA_BASE_URL`) |
| GMI Cloud | `gmi` | `GMI_API_KEY` (optional: `GMI_BASE_URL`) |
| StepFun | `stepfun` | `STEPFUN_API_KEY` (optional: `STEPFUN_BASE_URL`) |
| Ollama Cloud | `ollama-cloud` | `OLLAMA_API_KEY` |
| Google AI Studio | `gemini` | `GOOGLE_API_KEY` (alias: `GEMINI_API_KEY`) |
| xAI (Grok) | `xai` (alias `grok`) | `XAI_API_KEY` (optional: `XAI_BASE_URL`) |
| xAI Grok OAuth (SuperGrok) | `xai-oauth` (alias `grok-oauth`) | `hermes modelo` → xAI Grok OAuth (browser login; SuperGrok subscription) |
| AWS Bedrock | `bedrock` | Standard boto3 auth (`AWS_REGION` + `AWS_PROFILE` or `AWS_ACCESS_KEY_ID`) |
| Qwen Portal (OAuth) | `qwen-oauth` | `hermes modelo` (Qwen Portal OAuth; optional: `HERMES_QWEN_BASE_URL`) |
| MiniMax (OAuth) | `minimax-oauth` | `hermes modelo` (MiniMax portal OAuth) |
| OpenCode Zen | `opencode-zen` | `OPENCODE_ZEN_API_KEY` |
| OpenCode Go | `opencode-go` | `OPENCODE_GO_API_KEY` |
| Kilo Code | `kilocode` | `KILOCODE_API_KEY` |
| Xiaomi MiMo | `xiaomi` | `XIAOMI_API_KEY` |
| Arcee AI | `arcee` | `ARCEEAI_API_KEY` |
| GMI Cloud | `gmi` | `GMI_API_KEY` |
| Alibaba / DashScope | `alibaba` | `DASHSCOPE_API_KEY` |
| Alibaba Coding Plan | `alibaba-coding-plan` | `ALIBABA_CODING_PLAN_API_KEY` (falls back to `DASHSCOPE_API_KEY`) |
| Kimi / Moonshot (China) | `kimi-coding-cn` | `KIMI_CN_API_KEY` |
| StepFun | `stepfun` | `STEPFUN_API_KEY` |
| Tencent TokenHub | `tencent-tokenhub` | `TOKENHUB_API_KEY` |
| Microsoft Foundry | `azure-foundry` | `AZURE_FOUNDRY_API_KEY` + `AZURE_FOUNDRY_BASE_URL` |
| LM Studio (local) | `lmstudio` | `LM_API_KEY` (or none for local) + `LM_BASE_URL` |
| Hugging Face | `huggingface` | `HF_TOKEN` |
| Custom endpoint | `custom` | `base_url` + `key_env` (see below) |

### Custom Endpoint Fallback

For a custom OpenAI-compatible endpoint, add `base_url` and optionally `key_env`:

```yaml
fallback_proveedors:
  - proveedor: custom
    modelo: my-local-modelo
    base_url: http://localhost:8000/v1
    key_env: MY_LOCAL_KEY            # env var name containing the API key
```

### When Fallback Triggers

The fallback activates automatically when the primary modelo fails with:

- **Rate limits** (HTTP 429) — after exhausting retry attempts
- **Server errors** (HTTP 500, 502, 503) — after exhausting retry attempts
- **Auth failures** (HTTP 401, 403) — immediately (no point retrying)
- **Not found** (HTTP 404) — immediately
- **Invalid responses** — when the API returns malformed or empty responses repeatedly

When triggered, Hermes:

1. Resolves credentials for the fallback proveedor
2. Builds a new API client
3. Swaps the modelo, proveedor, and client in-place
4. Resets the retry counter and continues the conversation

The switch is seamless — your conversation history, herramienta calls, and contexto are preserved. The agente continues from exactly where it left off, just using a different modelo.

:::info Per-Turn, Not Per-Session
Fallback is **turn-scoped**: each new user message starts with the primary modelo restored. If the primary fails mid-turn, fallback activates for that turn only. On the next message, Hermes tries the primary again. Within a single turn, fallback activates at most once — if the fallback also fails, normal error handling takes over (retries, then error message). This prevents cascading failover loops within a turn while giving the primary modelo a fresh chance every turn.
:::

### Ejemplos

**OpenRouter as fallback for Anthropic native:**
```yaml
modelo:
  proveedor: anthropic
  default: claude-sonnet-4-6

fallback_proveedors:
  - proveedor: openrouter
    modelo: anthropic/claude-sonnet-4
```

**Nous Portal as fallback for OpenRouter:**
```yaml
modelo:
  proveedor: openrouter
  default: anthropic/claude-opus-4

fallback_proveedors:
  - proveedor: nous
    modelo: nous-hermes-3
```

**Local modelo as fallback for cloud:**
```yaml
fallback_proveedors:
  - proveedor: custom
    modelo: llama-3.1-70b
    base_url: http://localhost:8000/v1
    key_env: LOCAL_API_KEY
```

**Codex OAuth as fallback:**
```yaml
fallback_proveedors:
  - proveedor: openai-codex
    modelo: gpt-5.3-codex
```

### Where Fallback Works

| Contexto | Fallback Supported |
|---------|-------------------|
| CLI sessions | ✔ |
| Messaging puerta de enlace (Telegram, Discord, etc.) | ✔ |
| Subagente delegation | ✔ (subagentes inherit the parent fallback chain) |
| Cron jobs | ✔ (cron agentes inherit configured fallback proveedors) |
| Auxiliary tasks on `proveedor: auto` | ✔ (try per-task fallback, then the main fallback chain before built-in aux discovery) |

:::tip
There are no environment variables for the primary fallback chain — configure it exclusively through `config.yaml` or `hermes fallback`. This is intentional: fallback configuración is a deliberate choice, not something a stale shell export should override.
:::

---

## Auxiliary Task Fallback

Hermes uses separate lightweight modelos for side tasks. Each task has its own proveedor resolution chain that acts as a built-in fallback system.

### Tasks with Independent Proveedor Resolution

| Task | What It Does | Config Key |
|------|-------------|-----------|
| Vision | Image analysis, browser screenshots | `auxiliary.vision` |
| Web Extract | Web page summarization | `auxiliary.web_extract` |
| Compression | Contexto compression summaries | `auxiliary.compression` |
| Habilidads Hub | Habilidad search and discovery | `auxiliary.habilidads_hub` |
| MCP | MCP helper operations | `auxiliary.mcp` |
| Approval | Smart command-approval classification | `auxiliary.approval` |
| Title Generation | Session title summaries | `auxiliary.title_generation` |
| Triage Specifier | `hermes kanban specify` / panel ✨ button — fleshes out a one-liner triage task into a real spec | `auxiliary.triage_specifier` |

### Auto-Detection Chain

When a task's proveedor is set to `"auto"` (the default), Hermes first tries the main proveedor + main modelo for that auxiliary task. If that route is unavailable or later fails with a capacity-style error, Hermes now honors user-configured fallback policy before using the built-in discovery chain:

```text
Main proveedor + main modelo → auxiliary.<task>.fallback_chain →
fallback_proveedors / fallback_modelo → built-in auxiliary discovery chain
```

The task-specific chain is most precise and wins when present. The top-level `fallback_proveedors` chain is the same policy the main agente uses, so free-only or same-proveedor fallback rules apply to auxiliary tasks on `auto` as well.

**Built-in text discovery chain (compression, web extract, title generation, etc.):**

```text
OpenRouter → Nous Portal → Custom endpoint → Codex OAuth →
API-key proveedors (z.ai, Kimi, MiniMax, Xiaomi MiMo, Hugging Face, Anthropic) → give up
```

**Built-in vision discovery chain:**

```text
Main proveedor (if vision-capable) → OpenRouter → Nous Portal →
Codex OAuth → Anthropic → Custom endpoint → give up
```

Those built-in chains are a convenience fallback for users who have not declared a task-specific or main fallback policy.

### Configuring Auxiliary Proveedors

Each task can be configured independently in `config.yaml`:

```yaml
auxiliary:
  vision:
    proveedor: "auto"              # auto | openrouter | nous | codex | main | anthropic
    modelo: ""                     # e.g. "openai/gpt-4o"
    base_url: ""                  # direct endpoint (takes precedence over proveedor)
    api_key: ""                   # API key for base_url

  web_extract:
    proveedor: "auto"
    modelo: ""

  compression:
    proveedor: "auto"
    modelo: ""
    fallback_chain:              # optional, task-specific fallback policy
      - proveedor: openrouter
        modelo: inclusionai/ring-2.6-1t:free

  habilidads_hub:
    proveedor: "auto"
    modelo: ""

  mcp:
    proveedor: "auto"
    modelo: ""
```

Every task above follows the same **proveedor / modelo / base_url** pattern. Each task can also declare its own `fallback_chain`; if omitted, `proveedor: auto` uses the top-level `fallback_proveedors` chain before Hermes' built-in auxiliary discovery chain.

Contexto compression is configured under `auxiliary.compression`:

```yaml
auxiliary:
  compression:
    proveedor: main                                    # Same proveedor options as other auxiliary tasks
    modelo: google/gemini-3-flash-preview
    base_url: null                                    # Custom OpenAI-compatible endpoint
```

And the primary fallback chain uses:

```yaml
fallback_proveedors:
  - proveedor: openrouter
    modelo: anthropic/claude-sonnet-4
    # base_url: http://localhost:8000/v1             # Optional custom endpoint
```

All three — auxiliary, compression, fallback — work the same way: set `proveedor` to pick who handles the request, `modelo` to pick which modelo, and `base_url` to point at a custom endpoint (overrides proveedor).

### Proveedor Options for Auxiliary Tasks

These options apply to `auxiliary:`, `compression:`, and `fallback_proveedors:` entries only — `"main"` is **not** a valid value for your top-level `modelo.proveedor`. For custom endpoints, use `proveedor: custom` in your `modelo:` section (see [AI Proveedors](/integrations/proveedors)).

| Proveedor | Description | Requirements |
|----------|-------------|-------------|
| `"auto"` | Try proveedors in order until one works (default) | At least one proveedor configured |
| `"openrouter"` | Force OpenRouter | `OPENROUTER_API_KEY` |
| `"nous"` | Force Nous Portal | `hermes auth` |
| `"codex"` | Force Codex OAuth | `hermes modelo` → Codex |
| `"main"` | Use whatever proveedor the main agente uses (auxiliary tasks only) | Active main proveedor configured |
| `"anthropic"` | Force Anthropic native | `ANTHROPIC_API_KEY` or Claude Code credentials |

### Direct Endpoint Override

For any auxiliary task, setting `base_url` bypasses proveedor resolution entirely and sends requests directly to that endpoint:

```yaml
auxiliary:
  vision:
    base_url: "http://localhost:1234/v1"
    api_key: "local-key"
    modelo: "qwen2.5-vl"
```

`base_url` takes precedence over `proveedor`. Hermes uses the configured `api_key` for autenticación, falling back to `OPENAI_API_KEY` if not set. It does **not** reuse `OPENROUTER_API_KEY` for custom endpoints.

---

## Auxiliary Capacity-Error Fallback

When you set an explicit auxiliary proveedor (e.g. `auxiliary.vision.proveedor: glm`), Hermes treats that as your preferred choice — but if the proveedor literally cannot serve the request because of a **capacity error** (HTTP 402 payment required, HTTP 429 daily-quota exhaustion, connection failure), Hermes falls back through a layered chain instead of failing silently:

1. **Primary aux proveedor** — the one you configured (tried first, always)
2. **`auxiliary.<task>.fallback_chain`** — your per-task override list, if you wrote one
3. **Main agente proveedor + modelo** — last-resort safety net (always tried, even if you didn't write a chain)
4. **Warn + re-raise** — if every layer fails, Hermes logs `Auxiliary <task>: ... all fallbacks exhausted` at WARNING level and re-raises the original error

Transient HTTP 429 rate limits (`Retry-After: ...`) are treated as request constraints, not capacity problems — they respect your explicit proveedor choice and do **not** trigger the fallback ladder. Only daily/monthly quota exhaustion, payment errors, and connection failures bypass the explicit-proveedor gate.

For users on `proveedor: auto` (no explicit aux proveedor), the existing auto-detection chain runs in place of steps 2–3. Its first step is already the main agente modelo, so `auto` users get the same outcome with zero config.

### Optional: per-task fallback chain

If you want a different fallback ordering than "main agente modelo first", configure `fallback_chain` explicitly. Each entry needs at least `proveedor`; `modelo`, `base_url`, and `api_key` are optional.

```yaml
auxiliary:
  vision:
    proveedor: glm
    modelo: glm-4v-flash
    fallback_chain:
      - proveedor: openrouter
        modelo: google/gemini-3-flash-preview
      - proveedor: nous
        modelo: anthropic/claude-sonnet-4

  compression:
    proveedor: openrouter
    fallback_chain:
      - proveedor: openai
        modelo: gpt-4o-mini
```

You do **not** need to configure `fallback_chain` to get fallback — the main-agente safety net runs regardless. Use it only when you specifically want a different order than the default.

### Proveedor quota errors that trigger fallback

Hermes recognizes these as capacity-equivalent to 402 credit exhaustion (not transient rate limits):

- Bedrock / LiteLLM: `Too many tokens per day`, `daily limit`, `tokens per day`
- Vertex AI / GCP: `quota exceeded`, `resource exhausted`, `RESOURCE_EXHAUSTED`
- Generic: `daily quota`, `quota_exceeded`

If your proveedor returns a different phrase for daily-quota exhaustion and Hermes doesn't trigger fallback, that's a bug — open an issue with the exact error string.

---

## Contexto Compression Fallback

Contexto compression uses the `auxiliary.compression` config block to control which modelo and proveedor handles summarization:

```yaml
auxiliary:
  compression:
    proveedor: "auto"                              # auto | openrouter | nous | main
    modelo: "google/gemini-3-flash-preview"
```

:::info Legacy migration
Older configs with `compression.summary_modelo` / `compression.summary_proveedor` / `compression.summary_base_url` are automatically migrated to `auxiliary.compression.*` on first load (config version 17).
:::

If no proveedor is available for compression, Hermes drops middle conversation turns without generating a summary rather than failing the session.

---

## Delegation Proveedor Override

Subagentes spawned by `delegate_task` inherit the parent agente's primary fallback chain. You can still route subagentes to a different primary proveedor:modelo pair for cost optimization:

```yaml
delegation:
  proveedor: "openrouter"                      # override proveedor for all subagentes
  modelo: "google/gemini-3-flash-preview"      # override modelo
  # base_url: "http://localhost:1234/v1"      # or use a direct endpoint
  # api_key: "local-key"
```

See [Subagente Delegation](/guia-usuario/features/delegation) for full configuración details.

---

## Cron Job Proveedors

Cron jobs inherit your configured `fallback_proveedors` chain (or legacy `fallback_modelo`) when they create an agente. To use a different primary proveedor for a cron job, configure `proveedor` and `modelo` overrides on the cron job itself:

```python
cronjob(
    action="create",
    schedule="every 2h",
    prompt="Check server status",
    proveedor="openrouter",
    modelo="google/gemini-3-flash-preview"
)
```

See [Scheduled Tasks (Cron)](/guia-usuario/features/cron) for full configuración details.

---

## Summary

| Feature | Fallback Mechanism | Config Location |
|---------|-------------------|----------------|
| Main agente modelo | `fallback_proveedors` in config.yaml — per-turn failover on errors (primary restored each turn) | `fallback_proveedors:` (top-level list) |
| Auxiliary tasks (any) — auto users | Full auto-detection chain (main agente modelo first, then proveedor chain) on capacity errors | `auxiliary.<task>.proveedor: auto` |
| Auxiliary tasks (any) — explicit proveedor | `fallback_chain` (if set) → main agente modelo → warn + raise, on capacity errors only | `auxiliary.<task>.fallback_chain` |
| Vision | Layered (see above) + internal OpenRouter retry | `auxiliary.vision` |
| Web extraction | Layered (see above) + internal OpenRouter retry | `auxiliary.web_extract` |
| Contexto compression | Layered (see above); degrades to no-summary if all layers unavailable | `auxiliary.compression` |
| Habilidads hub | Layered (see above) | `auxiliary.habilidads_hub` |
| MCP helpers | Layered (see above) | `auxiliary.mcp` |
| Approval classification | Layered (see above) | `auxiliary.approval` |
| Title generation | Layered (see above) | `auxiliary.title_generation` |
| Triage specifier | Layered (see above) | `auxiliary.triage_specifier` |
| Delegation | Proveedor override only (no automatic fallback) | `delegation.proveedor` / `delegation.modelo` |
| Cron jobs | Per-job proveedor override only (no automatic fallback) | Per-job `proveedor` / `modelo` |

---