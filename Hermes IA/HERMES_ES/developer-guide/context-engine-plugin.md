<!-- source: website/docs/guia-desarrollador/contexto-engine-complemento.md -->
# Contexto Engine Complementos

# Building a Contexto Engine Complemento

Contexto engine complementos replace the built-in `ContextoCompressor` with an alternative strategy for managing conversation contexto. For example, a Lossless Contexto Management (LCM) engine that builds a knowledge DAG instead of lossy summarization.

## How it works

The agente's contexto management is built on the `ContextoEngine` ABC (`agente/contexto_engine.py`). The built-in `ContextoCompressor` is the default implementation. Complemento engines must implement the same interface.

Only **one** contexto engine can be active at a time. Selection is config-driven:

```yaml
# config.yaml
contexto:
  engine: "compressor"    # default built-in
  engine: "lcm"           # activates a complemento engine named "lcm"
```

Complemento engines are **never auto-activated** — the user must explicitly set `contexto.engine` to the complemento's name.

## Directory structure

Each contexto engine lives in `complementos/contexto_engine/<name>/`:

```
complementos/contexto_engine/lcm/
├── __init__.py      # exports the ContextoEngine subclass
├── complemento.yaml      # metadata (name, description, version)
└── ...              # any other modules your engine needs
```

## The ContextoEngine ABC

Your engine must implement these **required** methods:

```python
from agente.contexto_engine import ContextoEngine

class LCMEngine(ContextoEngine):

    @property
    def name(self) -> str:
        """Short identifier, e.g. 'lcm'. Must match config.yaml value."""
        return "lcm"

    def update_from_response(self, usage: dict) -> None:
        """Called after every LLM call with the usage dict.

        Update self.last_prompt_tokens, self.last_completion_tokens,
        self.last_total_tokens from the response.
        """

    def should_compress(self, prompt_tokens: int = None) -> bool:
        """Return True if compaction should fire this turn."""

    def compress(self, messages: list, current_tokens: int = None,
                 focus_topic: str = None) -> list:
        """Compact the message list and return a new (possibly shorter) list.

        The returned list must be a valid OpenAI-format message sequence.

        ``focus_topic`` is an optional topic string from manual
        ``/compress <focus>``; engines that support guided compression should
        prioritise preserving information related to it, others may ignore it.
        """
```

### Class attributes your engine must maintain

The agente reads these directly for display and logging:

```python
last_prompt_tokens: int = 0
last_completion_tokens: int = 0
last_total_tokens: int = 0
threshold_tokens: int = 0        # when compression triggers
contexto_length: int = 0          # modelo's full contexto window
compression_count: int = 0       # how many times compress() has run
```

### Optional methods

These have sensible defaults in the ABC. Override as needed:

| Method | Default | Override when |
|--------|---------|--------------|
| `on_session_start(session_id, **kwargs)` | No-op | You need to load persisted state (DAG, DB) |
| `on_session_end(session_id, messages)` | No-op | You need to flush state, close connections |
| `on_session_reset()` | Resets token counters | You have per-session state to clear |
| `update_modelo(modelo, contexto_length, ...)` | Updates contexto_length + threshold | You need to recalculate budgets on modelo switch |
| `get_herramienta_schemas()` | Returns `[]` | Your engine provides agente-callable herramientas (e.g., `lcm_grep`) |
| `handle_herramienta_call(name, args, **kwargs)` | Returns error JSON | You implement herramienta handlers |
| `should_compress_preflight(messages)` | Returns `False` | You can do a cheap pre-API-call estimate |
| `get_status()` | Standard token/threshold dict | You have custom metrics to expose |

## Engine herramientas

Contexto engines can expose herramientas the agente calls directly. Return schemas from `get_herramienta_schemas()` and handle calls in `handle_herramienta_call()`:

```python
def get_herramienta_schemas(self):
    return [{
        "name": "lcm_grep",
        "description": "Search the contexto knowledge graph",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"}
            },
            "required": ["query"],
        },
    }]

def handle_herramienta_call(self, name, args, **kwargs):
    if name == "lcm_grep":
        results = self._search_dag(args["query"])
        return json.dumps({"results": results})
    return json.dumps({"error": f"Unknown herramienta: {name}"})
```

Engine herramientas are injected into the agente's herramienta list at startup and dispatched automatically — no registry registration needed.

## Registration

### Via directory (recommended)

Place your engine in `complementos/contexto_engine/<name>/`. The `__init__.py` must export a `ContextoEngine` subclass. The discovery system finds and instantiates it automatically.

### Via general complemento system

A general complemento can also register a contexto engine:

```python
def register(ctx):
    engine = LCMEngine(contexto_length=200000)
    ctx.register_contexto_engine(engine)
```

Only one engine can be registered. A second complemento attempting to register is rejected with a warning.

## Lifecycle

```
1. Engine instantiated (complemento load or directory discovery)
2. on_session_start() — conversation begins
3. update_from_response() — after each API call
4. should_compress() — checked each turn
5. compress() — called when should_compress() returns True
6. on_session_end() — session boundary (CLI exit, /reset, puerta de enlace expiry)
```

`on_session_reset()` is called on `/new` or `/reset` to clear per-session state without a full shutdown.

## Configuración

Users select your engine via `hermes complementos` → Proveedor Complementos → Contexto Engine, or by editing `config.yaml`:

```yaml
contexto:
  engine: "lcm"   # must match your engine's name property
```

The `compression` config block (`compression.threshold`, `compression.protect_last_n`, etc.) is specific to the built-in `ContextoCompressor`. Your engine should define its own config format if needed, reading from `config.yaml` during initialization.

## Testing

```python
from agente.contexto_engine import ContextoEngine

def test_engine_satisfies_abc():
    engine = YourEngine(contexto_length=200000)
    assert isinstance(engine, ContextoEngine)
    assert engine.name == "your-name"

def test_compress_returns_valid_messages():
    engine = YourEngine(contexto_length=200000)
    msgs = [{"role": "user", "content": "hello"}]
    result = engine.compress(msgs)
    assert isinstance(result, list)
    assert all("role" in m for m in result)
```

See `tests/agente/test_contexto_engine.py` for the full ABC contract test suite.

## See also

- [Contexto Compression and Caching](/guia-desarrollador/contexto-compression-and-caching) — how the built-in compressor works
- [Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento) — analogous single-select complemento system for memoria
- [Complementos](/guia-usuario/features/complementos) — general complemento system overview

---