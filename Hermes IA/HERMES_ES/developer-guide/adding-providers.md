<!-- source: website/docs/guia-desarrollador/adding-proveedors.md -->
# Adding Proveedors

# Adding Proveedors

Hermes can already talk to any OpenAI-compatible endpoint through the custom proveedor path. Do not add a built-in proveedor unless you want first-class UX for that service:

- proveedor-specific auth or token refresh
- a curated modelo catalog
- setup / `hermes modelo` menu entries
- proveedor aliases for `proveedor:modelo` syntax
- a non-OpenAI API shape that needs an adapter

If the proveedor is just "another OpenAI-compatible base URL and API key", a named custom proveedor may be enough.

## The mental modelo

A built-in proveedor has to line up across a few layers:

1. `hermes_cli/auth.py` decides how credentials are found.
2. `hermes_cli/runtime_proveedor.py` turns that into runtime data:
   - `proveedor`
   - `api_mode`
   - `base_url`
   - `api_key`
   - `source`
3. `run_agente.py` uses `api_mode` to decide how requests are built and sent.
4. `hermes_cli/modelos.py` and `hermes_cli/main.py` make the proveedor show up in the CLI. (`hermes_cli/setup.py` delegates to `main.py` automatically — no changes needed there.)
5. `agente/auxiliary_client.py` and `agente/modelo_metadata.py` keep side tasks and token budgeting working.

The important abstraction is `api_mode`.

- Most proveedors use `chat_completions`.
- Codex uses `codex_responses`.
- Anthropic uses `anthropic_messages`.
- A new non-OpenAI protocol usually means adding a new adapter and a new `api_mode` branch.

## Choose the implementation path first

### Path A — OpenAI-compatible proveedor

Use this when the proveedor accepts standard chat-completions style requests.

Typical work:

- add auth metadata
- add modelo catalog / aliases
- add runtime resolution
- add CLI menu wiring
- add aux-modelo defaults
- add tests and user docs

You usually do not need a new adapter or a new `api_mode`.

### Path B — Native proveedor

Use this when the proveedor does not behave like OpenAI chat completions.

Examples in-tree today:

- `codex_responses`
- `anthropic_messages`

This path includes everything from Path A plus:

- a proveedor adapter in `agente/`
- `run_agente.py` branches for request building, dispatch, usage extraction, interrupt handling, and response normalization
- adapter tests

## File checklist

### Required for every built-in proveedor

1. `hermes_cli/auth.py`
2. `hermes_cli/modelos.py`
3. `hermes_cli/runtime_proveedor.py`
4. `hermes_cli/main.py`
5. `agente/auxiliary_client.py`
6. `agente/modelo_metadata.py`
7. tests
8. user-facing docs under `website/docs/`

:::tip
`hermes_cli/setup.py` does **not** need changes. The setup wizard delegates proveedor/modelo selection to `select_proveedor_and_modelo()` in `main.py` — any proveedor added there is automatically available in `hermes setup`.
:::

### Additional for native / non-OpenAI proveedors

10. `agente/<proveedor>_adapter.py`
11. `run_agente.py`
12. `pyproject.toml` if a proveedor SDK is required

## Fast path: Simple API-key proveedors

If your proveedor is just an OpenAI-compatible endpoint that authenticates with a single API key, you do not need to touch `auth.py`, `runtime_proveedor.py`, `main.py`, or any of the other files in the full checklist below.

All you need is:

1. A complemento directory under `complementos/modelo-proveedors/<your-proveedor>/` containing:
   - `__init__.py` — calls `register_proveedor(perfil)` at module-level
   - `complemento.yaml` — manifest (name, kind: modelo-proveedor, version, description)
2. That's it. Proveedor complementos auto-load the first time anything calls `get_proveedor_perfil()` or `list_proveedors()` — bundled complementos (this repo) and user complementos at `$HERMES_HOME/complementos/modelo-proveedors/` both get picked up.

When you add a complemento and it calls `register_proveedor()`, the following wire up automatically:

1. `PROVIDER_REGISTRY` entry in `auth.py` (credential resolution, env-var lookup)
2. `api_mode` set to `chat_completions`
3. `base_url` sourced from the config or the declared env var
4. `env_vars` checked in priority order for the API key
5. `fallback_modelos` list registered for the proveedor
6. `--proveedor` CLI flag accepts the proveedor id
7. `hermes modelo` menu includes the proveedor
8. `hermes setup` wizard delegates to `main.py` automatically
9. `proveedor:modelo` alias syntax works
10. Runtime resolver returns the correct `base_url` and `api_key`
11. `--proveedor <name>` CLI flag accepts the proveedor id
12. Fallback modelo activation can switch into the proveedor cleanly

User complementos at `$HERMES_HOME/complementos/modelo-proveedors/<name>/` override bundled complementos of the same name (last-writer-wins in `register_proveedor()`) — so third parties can monkey-patch or replace any built-in perfil without editing the repo.

See `complementos/modelo-proveedors/nvidia/` or `complementos/modelo-proveedors/gmi/` as a template, and the full [Modelo Proveedor Complemento guide](/guia-desarrollador/modelo-proveedor-complemento) for field reference, hook idioms, and end-to-end examples.

## Full path: OAuth and complex proveedors

Use the full checklist below when your proveedor needs any of the following:

- OAuth or token refresh (Nous Portal, Codex, Qwen Portal, Copilot)
- A non-OpenAI API shape that requires a new adapter (Anthropic Messages, Codex Responses)
- Custom endpoint detection or multi-region probing (z.ai, Kimi)
- A curated static modelo catalog or live `/modelos` fetch
- Proveedor-specific `hermes modelo` menu entries with bespoke auth flows

## Step 1: Pick one canonical proveedor id

Choose a single proveedor id and use it everywhere.

Examples from the repo:

- `openai-codex`
- `kimi-coding`
- `minimax-cn`

That same id should appear in:

- `PROVIDER_REGISTRY` in `hermes_cli/auth.py`
- `_PROVIDER_LABELS` in `hermes_cli/modelos.py`
- `_PROVIDER_ALIASES` in both `hermes_cli/auth.py` and `hermes_cli/modelos.py`
- CLI `--proveedor` choices in `hermes_cli/main.py`
- setup / modelo selection branches
- auxiliary-modelo defaults
- tests

If the id differs between those files, the proveedor will feel half-wired: auth may work while `/modelo`, setup, or runtime resolution silently misses it.

## Step 2: Add auth metadata in `hermes_cli/auth.py`

For API-key proveedors, add a `ProveedorConfig` entry to `PROVIDER_REGISTRY` with:

- `id`
- `name`
- `auth_type="api_key"`
- `inference_base_url`
- `api_key_env_vars`
- optional `base_url_env_var`

Also add aliases to `_PROVIDER_ALIASES`.

Use the existing proveedors as templates:

- simple API-key path: Z.AI, MiniMax
- API-key path with endpoint detection: Kimi, Z.AI
- native token resolution: Anthropic
- OAuth / auth-store path: Nous, OpenAI Codex

Questions to answer here:

- What env vars should Hermes check, and in what priority order?
- Does the proveedor need base-URL overrides?
- Does it need endpoint probing or token refresh?
- What should the auth error say when credentials are missing?

If the proveedor needs something more than "look up an API key", add a dedicated credential resolver instead of shoving logic into unrelated branches.

## Step 3: Add modelo catalog and aliases in `hermes_cli/modelos.py`

Update the proveedor catalog so the proveedor works in menus and in `proveedor:modelo` syntax.

Typical edits:

- `_PROVIDER_MODELS`
- `_PROVIDER_LABELS`
- `_PROVIDER_ALIASES`
- proveedor display order inside `list_available_proveedors()`
- `proveedor_modelo_ids()` if the proveedor supports a live `/modelos` fetch

If the proveedor exposes a live modelo list, prefer that first and keep `_PROVIDER_MODELS` as the static fallback.

This file is also what makes inputs like these work:

```text
anthropic:claude-sonnet-4-6
kimi:modelo-name
```

If aliases are missing here, the proveedor may authenticate correctly but still fail in `/modelo` parsing.

## Step 4: Resolve runtime data in `hermes_cli/runtime_proveedor.py`

`resolve_runtime_proveedor()` is the shared path used by CLI, puerta de enlace, cron, ACP, and helper clients.

Add a branch that returns a dict with at least:

```python
{
    "proveedor": "your-proveedor",
    "api_mode": "chat_completions",  # or your native mode
    "base_url": "https://...",
    "api_key": "...",
    "source": "env|portal|auth-store|explicit",
    "requested_proveedor": requested_proveedor,
}
```

If the proveedor is OpenAI-compatible, `api_mode` should usually stay `chat_completions`.

Be careful with API-key precedence. Hermes already contains logic to avoid leaking an OpenRouter key to unrelated endpoints. A new proveedor should be equally explicit about which key goes to which base URL.

## Step 5: Wire the CLI in `hermes_cli/main.py`

A proveedor is not discoverable until it shows up in the interactive `hermes modelo` flow.

Update these in `hermes_cli/main.py`:

- `proveedor_labels` dict
- `proveedors` list in `select_proveedor_and_modelo()`
- proveedor dispatch (`if selected_proveedor == ...`)
- `--proveedor` argument choices
- login/logout choices if the proveedor supports those flows
- a `_modelo_flow_<proveedor>()` function, or reuse `_modelo_flow_api_key_proveedor()` if it fits

:::tip
`hermes_cli/setup.py` does not need changes — it calls `select_proveedor_and_modelo()` from `main.py`, so your new proveedor appears in both `hermes modelo` and `hermes setup` automatically.
:::

## Step 6: Keep auxiliary calls working

Two files matter here:

### `agente/auxiliary_client.py`

Add a cheap / fast default aux modelo to `_API_KEY_PROVIDER_AUX_MODELS` if this is a direct API-key proveedor.

Auxiliary tasks include things like:

- vision summarization
- web extraction summarization
- contexto compression summaries
- session-search summaries
- memoria flushes

If the proveedor has no sensible aux default, side tasks may fall back badly or use an expensive main modelo unexpectedly.

### `agente/modelo_metadata.py`

Add contexto lengths for the proveedor's modelos so token budgeting, compression thresholds, and limits stay sane.

## Step 7: If the proveedor is native, add an adapter and `run_agente.py` support

If the proveedor is not plain chat completions, isolate the proveedor-specific logic in `agente/<proveedor>_adapter.py`.

Keep `run_agente.py` focused on orchestration. It should call adapter helpers, not hand-build proveedor payloads inline all over the file.

A native proveedor usually needs work in these places:

### New adapter file

Typical responsibilities:

- build the SDK / HTTP client
- resolve tokens
- convert OpenAI-style conversation messages to the proveedor's request format
- convert herramienta schemas if needed
- normalize proveedor responses back into what `run_agente.py` expects
- extract usage and finish-reason data

### `run_agente.py`

Search for `api_mode` and audit every switch point. At minimum, verify:

- `__init__` chooses the new `api_mode`
- client construction works for the proveedor
- `_build_api_kwargs()` knows how to format requests
- `_interruptible_api_call()` dispatches to the right client call
- interrupt / client rebuild paths work
- response validation accepts the proveedor's shape
- finish-reason extraction is correct
- token-usage extraction is correct
- fallback-modelo activation can switch into the new proveedor cleanly
- summary-generation and memoria-flush paths still work

Also search `run_agente.py` for `self.client.`. Any code path that assumes the standard OpenAI client exists can break when a native proveedor uses a different client object or `self.client = None`.

### Prompt caching and proveedor-specific request fields

Prompt caching and proveedor-specific knobs are easy to regress.

Examples already in-tree:

- Anthropic has a native prompt-caching path
- OpenRouter gets proveedor-routing fields
- not every proveedor should receive every request-side option

When you add a native proveedor, double-check that Hermes is only sending fields that proveedor actually understands.

## Step 8: Tests

At minimum, touch the tests that guard proveedor wiring.

Common places:

- `tests/hermes_cli/test_runtime_proveedor_resolution.py`
- `tests/cli/test_cli_proveedor_resolution.py`
- `tests/hermes_cli/test_modelo_switch_custom_proveedors.py` (and adjacent `tests/hermes_cli/test_modelo_switch_*.py`)
- `tests/hermes_cli/test_setup_modelo_proveedor.py`
- `tests/run_agente/test_proveedor_parity.py`
- `tests/run_agente/test_run_agente.py`
- `tests/test_<proveedor>_adapter.py` for a native proveedor

For docs-only examples, the exact file set may differ. The point is to cover:

- auth resolution
- CLI menu / proveedor selection
- runtime proveedor resolution
- agente execution path
- proveedor:modelo parsing
- any adapter-specific message conversion

Run tests with xdist disabled:

```bash
source venv/bin/activate
python -m pytest tests/hermes_cli/test_runtime_proveedor_resolution.py tests/cli/test_cli_proveedor_resolution.py tests/hermes_cli/test_setup_modelo_proveedor.py tests/run_agente/test_proveedor_parity.py -n0 -q
```

For deeper changes, run the full suite before pushing:

```bash
source venv/bin/activate
python -m pytest tests/ -n0 -q
```

## Step 9: Live verification

After tests, run a real smoke test.

```bash
source venv/bin/activate
python -m hermes_cli.main chat -q "Say hello" --proveedor your-proveedor --modelo your-modelo
```

Also test the interactive flows if you changed menus:

```bash
source venv/bin/activate
python -m hermes_cli.main modelo
python -m hermes_cli.main setup
```

For native proveedors, verify at least one herramienta call too, not just a plain text response.

## Step 10: Update user-facing docs

If the proveedor is meant to ship as a first-class option, update the user docs too:

- `website/docs/inicio-rapido/quickstart.md`
- `website/docs/guia-usuario/configuración.md`
- `website/docs/reference/environment-variables.md`

A developer can wire the proveedor perfectly and still leave users unable to discover the required env vars or setup flow.

## OpenAI-compatible proveedor checklist

Use this if the proveedor is standard chat completions.

- [ ] `ProveedorConfig` added in `hermes_cli/auth.py`
- [ ] aliases added in `hermes_cli/auth.py` and `hermes_cli/modelos.py`
- [ ] modelo catalog added in `hermes_cli/modelos.py`
- [ ] runtime branch added in `hermes_cli/runtime_proveedor.py`
- [ ] CLI wiring added in `hermes_cli/main.py` (setup.py inherits automatically)
- [ ] aux modelo added in `agente/auxiliary_client.py`
- [ ] contexto lengths added in `agente/modelo_metadata.py`
- [ ] runtime / CLI tests updated
- [ ] user docs updated

## Native proveedor checklist

Use this when the proveedor needs a new protocol path.

- [ ] everything in the OpenAI-compatible checklist
- [ ] adapter added in `agente/<proveedor>_adapter.py`
- [ ] new `api_mode` supported in `run_agente.py`
- [ ] interrupt / rebuild path works
- [ ] usage and finish-reason extraction works
- [ ] fallback path works
- [ ] adapter tests added
- [ ] live smoke test passes

## Common pitfalls

### 1. Adding the proveedor to auth but not to modelo parsing

That makes credentials resolve correctly while `/modelo` and `proveedor:modelo` inputs fail.

### 2. Forgetting that `config["modelo"]` can be a string or a dict

A lot of proveedor-selection code has to normalize both forms.

### 3. Assuming a built-in proveedor is required

If the service is just OpenAI-compatible, a custom proveedor may already solve the user problem with less maintenance.

### 4. Forgetting auxiliary paths

The main chat path can work while summarization, memoria flushes, or vision helpers fail because aux routing was never updated.

### 5. Native-proveedor branches hiding in `run_agente.py`

Search for `api_mode` and `self.client.`. Do not assume the obvious request path is the only one.

### 6. Sending OpenRouter-only knobs to other proveedors

Fields like proveedor routing belong only on the proveedors that support them.

### 7. Updating `hermes modelo` but not `hermes setup`

Both flows need to know about the proveedor.

## Good search targets while implementing

If you are hunting for all the places a proveedor touches, search these symbols:

- `PROVIDER_REGISTRY`
- `_PROVIDER_ALIASES`
- `_PROVIDER_MODELS`
- `resolve_runtime_proveedor`
- `_modelo_flow_`
- `select_proveedor_and_modelo`
- `api_mode`
- `_API_KEY_PROVIDER_AUX_MODELS`
- `self.client.`

## Related docs

- [Proveedor Runtime Resolution](./proveedor-runtime.md)
- [Architecture](./architecture.md)
- [Contributing](./contributing.md)

---