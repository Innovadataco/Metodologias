<!-- source: website/docs/guia-desarrollador/memoria-proveedor-complemento.md -->
# Memoria Proveedor Complementos

# Building a Memoria Proveedor Complemento

Memoria proveedor complementos give Hermes Agente persistent, cross-session knowledge beyond the built-in MEMORY.md and USER.md. This guide covers how to build one.

:::tip
Memoria proveedors are one of two **proveedor complemento** types. The other is [Contexto Engine Complementos](/guia-desarrollador/contexto-engine-complemento), which replace the built-in contexto compressor. Both follow the same pattern: single-select, config-driven, managed via `hermes complementos`.
:::

## Directory Structure

Each memoria proveedor lives in `complementos/memoria/<name>/`:

```
complementos/memoria/my-proveedor/
├── __init__.py      # MemoriaProveedor implementation + register() entry point
├── complemento.yaml      # Metadata (name, description, hooks)
└── README.md        # Setup instructions, config reference, herramientas
```

## The MemoriaProveedor ABC

Your complemento implements the `MemoriaProveedor` abstract base class from `agente/memoria_proveedor.py`:

```python
from agente.memoria_proveedor import MemoriaProveedor

class MyMemoriaProveedor(MemoriaProveedor):
    @property
    def name(self) -> str:
        return "my-proveedor"

    def is_available(self) -> bool:
        """Check if this proveedor can activate. NO network calls."""
        return bool(os.environ.get("MY_API_KEY"))

    def initialize(self, session_id: str, **kwargs) -> None:
        """Called once at agente startup.

        kwargs always includes:
          hermes_home (str): Active HERMES_HOME path. Use for storage.
        """
        self._api_key = os.environ.get("MY_API_KEY", "")
        self._session_id = session_id

    # ... implement remaining methods
```

## Required Methods

### Core Lifecycle

| Method | When Called | Must Implement? |
|--------|-----------|-----------------|
| `name` (property) | Always | **Yes** |
| `is_available()` | Agente init, before activation | **Yes** — no network calls |
| `initialize(session_id, **kwargs)` | Agente startup | **Yes** |
| `get_herramienta_schemas()` | After init, for herramienta injection | **Yes** |
| `handle_herramienta_call(herramienta_name, args, **kwargs)` | When agente uses your herramientas | **Yes** (if you have herramientas) |

### Config

| Method | Purpose | Must Implement? |
|--------|---------|-----------------|
| `get_config_schema()` | Declare config fields for `hermes memoria setup` | **Yes** |
| `save_config(values, hermes_home)` | Write non-secret config to native location | **Yes** (unless env-var-only) |

### Optional Hooks

| Method | When Called | Use Case |
|--------|-----------|----------|
| `system_prompt_block()` | System prompt assembly | Static proveedor info |
| `prefetch(query, *, session_id="")` | Before each API call | Return recalled contexto |
| `queue_prefetch(query)` | After each turn | Pre-warm for next turn |
| `sync_turn(user, assistant, *, session_id="")` | After each completed turn | Persist conversation |
| `on_session_end(messages)` | Conversation ends | Final extraction/flush |
| `on_pre_compress(messages)` | Before contexto compression | Save insights before discard |
| `on_memoria_write(action, target, content)` | Built-in memoria writes | Mirror to your backend |
| `shutdown()` | Process exit | Clean up connections |

## Config Schema

`get_config_schema()` returns a list of field descriptors used by `hermes memoria setup`:

```python
def get_config_schema(self):
    return [
        {
            "key": "api_key",
            "description": "My Proveedor API key",
            "secret": True,           # → written to .env
            "required": True,
            "env_var": "MY_API_KEY",   # explicit env var name
            "url": "https://my-proveedor.com/keys",  # where to get it
        },
        {
            "key": "region",
            "description": "Server region",
            "default": "us-east",
            "choices": ["us-east", "eu-west", "ap-south"],
        },
        {
            "key": "project",
            "description": "Project identifier",
            "default": "hermes",
        },
    ]
```

Fields with `secret: True` and `env_var` go to `.env`. Non-secret fields are passed to `save_config()`.

:::tip Minimal vs Full Schema
Every field in `get_config_schema()` is prompted during `hermes memoria setup`. Proveedors with many options should keep the schema minimal — only include fields the user **must** configure (API key, required credentials). Document optional settings in a config file reference (e.g. `$HERMES_HOME/myproveedor.json`) rather than prompting for them all during setup. This keeps the setup wizard fast while still supporting advanced configuración. See the Supermemoria proveedor for an example — it only prompts for the API key; all other options live in `supermemoria.json`.
:::

## Save Config

```python
def save_config(self, values: dict, hermes_home: str) -> None:
    """Write non-secret config to your native location."""
    import json
    from pathlib import Path
    config_path = Path(hermes_home) / "my-proveedor.json"
    config_path.write_text(json.dumps(values, indent=2))
```

For env-var-only proveedors, leave the default no-op.

## Complemento Entry Point

```python
def register(ctx) -> None:
    """Called by the memoria complemento discovery system."""
    ctx.register_memoria_proveedor(MyMemoriaProveedor())
```

## complemento.yaml

```yaml
name: my-proveedor
version: 1.0.0
description: "Short description of what this proveedor does."
hooks:
  - on_session_end    # list hooks you implement
```

## Threading Contract

**`sync_turn()` MUST be non-blocking.** If your backend has latency (API calls, LLM processing), run the work in a daemon thread:

```python
def sync_turn(self, user_content, assistant_content, *, session_id="", messages=None):
    def _sync():
        try:
            self._api.ingest(user_content, assistant_content, session_id=session_id, messages=messages)
        except Exception as e:
            logger.warning("Sync failed: %s", e)

    if self._sync_thread and self._sync_thread.is_alive():
        self._sync_thread.join(timeout=5.0)
    self._sync_thread = threading.Thread(target=_sync, daemon=True)
    self._sync_thread.start()
```

`messages` is optional OpenAI-style conversation contexto as of the completed
turn. When present, it includes user/assistant messages, assistant herramienta calls,
and herramienta result messages. Proveedors that do not need raw turn contexto can omit
the `messages` parameter; Hermes will continue calling them with the legacy
signature.

Cloud proveedors should document what parts of `messages` are sent off-device.
Herramienta calls and herramienta results may contain file paths, command output, or other
workspace data.

## Perfil Isolation

All storage paths **must** use the `hermes_home` kwarg from `initialize()`, not hardcoded `~/.hermes`:

```python
# CORRECT — perfil-scoped
from hermes_constants import get_hermes_home
data_dir = get_hermes_home() / "my-proveedor"

# WRONG — shared across all perfils
data_dir = Path("~/.hermes/my-proveedor").expanduser()
```

## Testing

See `tests/agente/test_memoria_proveedor.py` and adjacent memoria tests (`tests/agente/test_memoria_session_switch.py`, `tests/agente/test_memoria_user_id.py`, `tests/run_agente/test_memoria_proveedor_init.py`) for end-to-end patterns.

```python
from agente.memoria_manager import MemoriaManager

mgr = MemoriaManager()
mgr.add_proveedor(my_proveedor)
mgr.initialize_all(session_id="test-1", platform="cli")

# Test herramienta routing
result = mgr.handle_herramienta_call("my_herramienta", {"action": "add", "content": "test"})

# Test lifecycle
mgr.sync_all("user msg", "assistant msg")
mgr.on_session_end([])
mgr.shutdown_all()
```

## Adding CLI Commands

Memoria proveedor complementos can register their own CLI subcommand tree (e.g. `hermes my-proveedor status`, `hermes my-proveedor config`). This uses a convention-based discovery system — no changes to core files needed.

### How it works

1. Add a `cli.py` file to your complemento directory
2. Define a `register_cli(subparser)` function that builds the argparse tree
3. The memoria complemento system discovers it at startup via `discover_complemento_cli_commands()`
4. Your commands appear under `hermes <proveedor-name> <subcommand>`

**Active-proveedor gating:** Your CLI commands only appear when your proveedor is the active `memoria.proveedor` in config. If a user hasn't configured your proveedor, your commands won't show in `hermes --help`.

### Example

```python
# complementos/memoria/my-proveedor/cli.py

def my_command(args):
    """Handler dispatched by argparse."""
    sub = getattr(args, "my_command", None)
    if sub == "status":
        print("Proveedor is active and connected.")
    elif sub == "config":
        print("Showing config...")
    else:
        print("Usage: hermes my-proveedor <status|config>")

def register_cli(subparser) -> None:
    """Build the hermes my-proveedor argparse tree.

    Called by discover_complemento_cli_commands() at argparse setup time.
    """
    subs = subparser.add_subparsers(dest="my_command")
    subs.add_parser("status", help="Show proveedor status")
    subs.add_parser("config", help="Show proveedor config")
    subparser.set_defaults(func=my_command)
```

### Referencia implementation

See `complementos/memoria/honcho/cli.py` for a full example with 13 subcommands, cross-perfil management (`--target-perfil`), and config read/write.

### Directory structure with CLI

```
complementos/memoria/my-proveedor/
├── __init__.py      # MemoriaProveedor implementation + register()
├── complemento.yaml      # Metadata
├── cli.py           # register_cli(subparser) — CLI commands
└── README.md        # Setup instructions
```

## Single Proveedor Rule

Only **one** external memoria proveedor can be active at a time. If a user tries to register a second, the MemoriaManager rejects it with a warning. This prevents herramienta schema bloat and conflicting backends.

---