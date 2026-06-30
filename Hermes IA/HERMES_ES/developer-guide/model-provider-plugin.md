<!-- source: website/docs/guia-desarrollador/modelo-proveedor-complemento.md -->
# Modelo Proveedor Complementos

# Building a Modelo Proveedor Complemento

Modelo proveedor complementos declare an inference backend — an OpenAI-compatible endpoint, an Anthropic Messages server, a Codex-style Responses API, or a Bedrock-native surface — that Hermes can route `AIAgente` calls through. Every built-in proveedor (OpenRouter, Anthropic, GMI, DeepSeek, Nvidia, …) ships as one of these complementos. Third parties can add their own by dropping a directory under `$HERMES_HOME/complementos/modelo-proveedors/` with zero changes to the repo.

:::tip
Modelo proveedor complementos are the third kind of **proveedor complemento**. The others are [Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento) (cross-session knowledge) and [Contexto Engine Complementos](/guia-desarrollador/contexto-engine-complemento) (contexto compression strategies). All three follow the same "drop a directory, declare a perfil, no repo edits" pattern.
:::

## How discovery works

`proveedors/__init__.py._discover_proveedors()` runs lazily the first time any code calls `get_proveedor_perfil()` or `list_proveedors()`. Discovery order:

1. **Bundled complementos** — `<repo>/complementos/modelo-proveedors/<name>/` — ship with Hermes
2. **User complementos** — `$HERMES_HOME/complementos/modelo-proveedors/<name>/` — drop in any directory; no restart required for subsequent sessions
3. **Legacy single-file** — `<repo>/proveedors/<name>.py` — back-compat for out-of-tree editable installs

**User complementos override bundled complementos of the same name** because `register_proveedor()` is last-writer-wins. Drop a `$HERMES_HOME/complementos/modelo-proveedors/gmi/` directory to replace the built-in GMI perfil without touching the repo.

## Directory structure

```
complementos/modelo-proveedors/my-proveedor/
├── __init__.py       # Calls register_proveedor(perfil) at module-level
├── complemento.yaml       # kind: modelo-proveedor + metadata (optional but recommended)
└── README.md         # Setup instructions (optional)
```

The only required file is `__init__.py`. `complemento.yaml` is used by `hermes complementos` for introspection and by the general ComplementoManager to route the complemento to the right loader; without it, the general loader falls back to a source-text heuristic.

## Minimal example — a simple API-key proveedor

```python
# complementos/modelo-proveedors/acme-inference/__init__.py
from proveedors import register_proveedor
from proveedors.base import ProveedorPerfil

acme = ProveedorPerfil(
    name="acme-inference",
    aliases=("acme",),
    display_name="Acme Inference",
    description="Acme — OpenAI-compatible direct API",
    signup_url="https://acme.example.com/keys",
    env_vars=("ACME_API_KEY", "ACME_BASE_URL"),
    base_url="https://api.acme.example.com/v1",
    auth_type="api_key",
    default_aux_modelo="acme-small-fast",
    fallback_modelos=(
        "acme-large-v3",
        "acme-medium-v3",
        "acme-small-fast",
    ),
)

register_proveedor(acme)
```

```yaml
# complementos/modelo-proveedors/acme-inference/complemento.yaml
name: acme-inference
kind: modelo-proveedor
version: 1.0.0
description: Acme Inference — OpenAI-compatible direct API
author: Your Name
```

That's it. After dropping these two files, the following **auto-wire** with no other edits:

| Integration | Where | What it gets |
|---|---|---|
| Credential resolution | `hermes_cli/auth.py` | `PROVIDER_REGISTRY["acme-inference"]` populated from perfil |
| `--proveedor` CLI flag | `hermes_cli/main.py` | Accepts `acme-inference` |
| `hermes modelo` picker | `hermes_cli/modelos.py` | Appears in `CANONICAL_PROVIDERS`, modelo list fetched from `{base_url}/modelos` |
| `hermes doctor` | `hermes_cli/doctor.py` | Health check for `ACME_API_KEY` + `{base_url}/modelos` probe |
| `hermes setup` | `hermes_cli/config.py` | `ACME_API_KEY` appears in `OPTIONAL_ENV_VARS` and the setup wizard |
| URL reverse-mapping | `agente/modelo_metadata.py` | Hostname → proveedor name for auto-detection |
| Auxiliary modelo | `agente/auxiliary_client.py` | Uses `default_aux_modelo` for compression / summarization |
| Runtime resolution | `hermes_cli/runtime_proveedor.py` | Returns correct `base_url`, `api_key`, `api_mode` |
| Transport | `agente/transports/chat_completions.py` | Perfil path generates kwargs via `prepare_messages` / `build_extra_body` / `build_api_kwargs_extras` |

## ProveedorPerfil fields

Full definition in `proveedors/base.py`. The most useful ones:

| Field | Type | Purpose |
|---|---|---|
| `name` | str | Canonical id — matches `modelo.proveedor` in `config.yaml` and the `--proveedor` flag |
| `aliases` | `tuple[str, ...]` | Alternative names resolved by `get_proveedor_perfil()` (e.g. `grok` → `xai`) |
| `api_mode` | str | `chat_completions` \| `codex_responses` \| `anthropic_messages` \| `bedrock_converse` |
| `display_name` | str | Human label shown in `hermes modelo` picker |
| `description` | str | Picker subtitle |
| `signup_url` | str | Shown during first-run setup ("get an API key here") |
| `env_vars` | `tuple[str, ...]` | API-key env vars in priority order; a final `*_BASE_URL` entry is used as the user base-URL override |
| `base_url` | str | Default inference endpoint |
| `modelos_url` | str | Explicit catalog URL (falls back to `{base_url}/modelos`) |
| `auth_type` | str | `api_key` \| `oauth_device_code` \| `oauth_external` \| `copilot` \| `aws_sdk` \| `external_process` |
| `fallback_modelos` | `tuple[str, ...]` | Curated list shown when live catalog fetch fails |
| `default_headers` | `dict[str, str]` | Sent on every request (e.g. Copilot's `Editor-Version`) |
| `fixed_temperature` | Any | `None` = use caller's value; `OMIT_TEMPERATURE` sentinel = don't send temperature at all (Kimi) |
| `default_max_tokens` | `int \| None` | Proveedor-level max_tokens cap (Nvidia: 16384) |
| `default_aux_modelo` | str | Cheap modelo for auxiliary tasks (compression, vision, summarization) |

## Overridable hooks

Subclass `ProveedorPerfil` for non-trivial quirks:

```python
from typing import Any
from proveedors.base import ProveedorPerfil

class AcmePerfil(ProveedorPerfil):
    def prepare_messages(self, messages: list[dict[str, Any]]) -> list[dict[str, Any]]:
        """Proveedor-specific message preprocessing. Runs after codex
        sanitization, before developer-role swap. Default: pass-through."""
        # Example: Qwen normalizes plain-text content to a list-of-parts
        # array and injects cache_control; Kimi rewrites herramienta-call JSON
        return messages

    def build_extra_body(self, *, session_id=None, **contexto) -> dict:
        """Proveedor-specific extra_body fields merged into the API call.
        Contexto includes: session_id, proveedor_preferences, modelo, base_url,
        reasoning_config. Default: empty dict."""
        # Example: OpenRouter's proveedor-preferences block,
        # Gemini's thinking_config translation.
        return {}

    def build_api_kwargs_extras(self, *, reasoning_config=None, **contexto):
        """Returns (extra_body_additions, top_level_kwargs). Needed when some
        fields go top-level (Kimi's reasoning_effort, OpenRouter's verbosity for
        adaptive Anthropic modelos) and some go in extra_body (OpenRouter's
        reasoning dict). Default: ({}, {})."""
        return {}, {}

    def fetch_modelos(self, *, api_key=None, timeout=8.0) -> list[str] | None:
        """Live catalog fetch. Default hits {modelos_url or base_url}/modelos with
        Bearer auth. Override for: custom auth (Anthropic), no REST endpoint
        (Bedrock → None), or public/unauthenticated catalogs (OpenRouter)."""
        return super().fetch_modelos(api_key=api_key, timeout=timeout)
```

## Hook reference examples

Look at these bundled complementos for idioms:

| Complemento | Why look |
|---|---|
| `complementos/modelo-proveedors/openrouter/` | Aggregator with proveedor preferences, public modelo catalog |
| `complementos/modelo-proveedors/gemini/` | `thinking_config` translation (native + OpenAI-compat nested forms) |
| `complementos/modelo-proveedors/kimi-coding/` | `OMIT_TEMPERATURE`, `extra_body.thinking`, top-level `reasoning_effort` |
| `complementos/modelo-proveedors/qwen-oauth/` | Message normalization, `cache_control` injection, VL high-res |
| `complementos/modelo-proveedors/nous/` | Attribution tags, "omit reasoning when disabled" |
| `complementos/modelo-proveedors/custom/` | Ollama `num_ctx` + `think: false` quirks |
| `complementos/modelo-proveedors/bedrock/` | `api_mode="bedrock_converse"`, `fetch_modelos` returns None (no REST endpoint) |

## User overrides — replace a built-in without editing the repo

Say you want to point `gmi` at your private staging endpoint for testing. Create `~/.hermes/complementos/modelo-proveedors/gmi/__init__.py`:

```python
from proveedors import register_proveedor
from proveedors.base import ProveedorPerfil

register_proveedor(ProveedorPerfil(
    name="gmi",
    aliases=("gmi-cloud", "gmicloud"),
    env_vars=("GMI_API_KEY",),
    base_url="https://gmi-staging.internal.example.com/v1",
    auth_type="api_key",
    default_aux_modelo="google/gemini-3.1-flash-lite-preview",
))
```

Next session, `get_proveedor_perfil("gmi").base_url` returns the staging URL. No repo patch, no rebuild. Because user complementos are discovered after bundled ones, the user `register_proveedor()` call wins.

## api_mode selection

Four values are recognized. Hermes picks one based on:

1. User explicit override (`config.yaml` `modelo.api_mode` when set)
2. OpenCode's per-modelo dispatch (`opencode_modelo_api_mode` for Zen and Go)
3. URL auto-detection — `/anthropic` suffix → `anthropic_messages`, `api.openai.com` → `codex_responses`, `api.x.ai` → `codex_responses`, `/coding` on Kimi domains → `chat_completions`
4. **Perfil `api_mode`** as a fallback when URL detection finds nothing
5. Default `chat_completions`

Set `perfil.api_mode` to match the default your proveedor ships — it acts as a hint. User URL overrides still win.

## Auth types

| `auth_type` | Meaning | Who uses it |
|---|---|---|
| `api_key` | Single env var carries a static API key | Most proveedors |
| `oauth_device_code` | Device-code OAuth flow | — |
| `oauth_external` | User signs in elsewhere, tokens land in `auth.json` | Anthropic OAuth, MiniMax OAuth, Qwen Portal, Nous Portal |
| `copilot` | GitHub Copilot token refresh cycle | `copilot` complemento only |
| `aws_sdk` | AWS SDK credential chain (IAM role, perfil, env) | `bedrock` complemento only |
| `external_process` | Auth handled by a subprocess the agente spawns | `copilot-acp` complemento only |

`auth_type` gates which codepaths treat your proveedor as a "simple api-key proveedor" — if it's not `api_key`, the ComplementoManager still records the manifest but Hermes' CLI-level automation (doctor checks, `--proveedor` flag, setup wizard delegation) may skip over it.

## Discovery timing

Proveedor discovery is **lazy** — triggered by the first `get_proveedor_perfil()` or `list_proveedors()` call in the process. In practice this happens early at startup (`auth.py` module load extends `PROVIDER_REGISTRY` eagerly). If you need to verify your complemento loaded, run:

```bash
hermes doctor
```

— a successful `auth_type="api_key"` perfil appears under the Proveedor Connectivity section with a `/modelos` probe.

For programmatic inspection:

```python
from proveedors import list_proveedors
for p in list_proveedors():
    print(p.name, p.base_url, p.api_mode)
```

## Testing your complemento

Point `HERMES_HOME` at a temp directory so you don't pollute your real config:

```bash
export HERMES_HOME=/tmp/hermes-complemento-test
mkdir -p $HERMES_HOME/complementos/modelo-proveedors/my-proveedor
cat > $HERMES_HOME/complementos/modelo-proveedors/my-proveedor/__init__.py <<'EOF'
from proveedors import register_proveedor
from proveedors.base import ProveedorPerfil
register_proveedor(ProveedorPerfil(
    name="my-proveedor",
    env_vars=("MY_API_KEY",),
    base_url="https://api.my-proveedor.example.com/v1",
    auth_type="api_key",
))
EOF

export MY_API_KEY=your-test-key
hermes -z "hello" --proveedor my-proveedor -m some-modelo
```

## General ComplementoManager integration

The general `ComplementoManager` (the thing `hermes complementos` operates on) **sees** modelo-proveedor complementos but does not import them — `proveedors/__init__.py` owns their lifecycle. The manager records the manifest for introspection and categorizes by `kind: modelo-proveedor`. When you drop an unlabeled user complemento into `$HERMES_HOME/complementos/` that happens to call `register_proveedor` with a `ProveedorPerfil`, the manager auto-coerces it to `kind: modelo-proveedor` via a source-text heuristic — so the complemento still routes correctly even without `complemento.yaml`.

## Distribute via pip

Like any Hermes complemento, modelo proveedors can ship as a pip package. Add an entry point to your `pyproject.toml`:

```toml
[project.entry-points."hermes_agente.complementos"]
acme-inference = "acme_hermes_complemento:register"
```

…where `acme_hermes_complemento:register` is a function that calls `register_proveedor(perfil)`. The general ComplementoManager picks up entry-point complementos during `discover_and_load()`. For `kind: modelo-proveedor` pip complementos, you still need to declare the kind in your manifest (or rely on the source-text heuristic).

See [Building a Hermes Complemento](/guides/build-a-hermes-complemento#distribute-via-pip) for the full entry-points setup.

## Related pages

- [Proveedor Runtime](/guia-desarrollador/proveedor-runtime) — resolution precedence + where each layer reads the perfil
- [Adding Proveedors](/guia-desarrollador/adding-proveedors) — end-to-end checklist for new inference backends (covers both the fast complemento path and the full CLI/auth integration)
- [Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento)
- [Contexto Engine Complementos](/guia-desarrollador/contexto-engine-complemento)
- [Building a Hermes Complemento](/guides/build-a-hermes-complemento) — general complemento authoring

---