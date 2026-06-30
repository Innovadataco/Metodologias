<!-- source: website/docs/guides/build-a-hermes-complemento.md -->
# Build a Hermes Complemento

# Build a Hermes Complemento

This guide walks through building a complete Hermes complemento from scratch. By the end you'll have a working complemento with multiple herramientas, lifecycle hooks, shipped data files, and a bundled habilidad — everything the complemento system supports.

:::info Not sure which guide you need?
Hermes has several distinct pluggable interfaces — some use Python `register_*` APIs, others are config-driven or drop-in directories. Use this map first:

| If you want to add… | Read |
|---|---|
| Custom herramientas, hooks, slash commands, habilidads, or CLI subcommands | **This guide** (the general complemento surface) |
| An **LLM / inference backend** (new proveedor) | [Modelo Proveedor Complementos](/guia-desarrollador/modelo-proveedor-complemento) |
| A **puerta de enlace channel** (Discord/Telegram/IRC/Teams/etc.) | [Adding Platform Adapters](/guia-desarrollador/adding-platform-adapters) |
| A **memoria backend** (Honcho/Mem0/Supermemoria/etc.) | [Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento) |
| A **contexto-compression engine** | [Contexto Engine Complementos](/guia-desarrollador/contexto-engine-complemento) |
| An **image-generation backend** | [Image Generation Proveedor Complementos](/guia-desarrollador/image-gen-proveedor-complemento) |
| A **video-generation backend** | [Video Generation Proveedor Complementos](/guia-desarrollador/video-gen-proveedor-complemento) |
| A **TTS backend** (any CLI — Piper, VoxCPM, Kokoro, voice cloning, …) | [TTS custom command proveedors](/guia-usuario/features/tts#custom-command-proveedors) — config-driven, no Python needed |
| An **STT backend** (custom whisper / ASR CLI) | [Voice Message Transcription](/guia-usuario/features/tts#voice-message-transcription-stt) — set `HERMES_LOCAL_STT_COMMAND` to a shell template |
| **External herramientas via MCP** (filesystem, GitHub, Linear, any MCP server) | [MCP](/guia-usuario/features/mcp) — declare `mcp_servers.<name>` in `config.yaml` |
| **Puerta de enlace event hooks** (fire on startup, session events, commands) | [Event Hooks](/guia-usuario/features/hooks#puerta de enlace-event-hooks) — drop `HOOK.yaml` + `handler.py` into `~/.hermes/hooks/<name>/` |
| **Shell hooks** (run a shell command on events) | [Shell Hooks](/guia-usuario/features/hooks#shell-hooks) — declare under `hooks:` in `config.yaml` |
| **Additional habilidad sources** (custom GitHub repos, private habilidad indexes) | [Habilidads](/guia-usuario/features/habilidads) — `hermes habilidads tap add <repo>` · [Publishing a tap](/guia-usuario/features/habilidads#publishing-a-custom-habilidad-tap) |
| A first-class **core** inference proveedor (not a complemento) | [Adding Proveedors](/guia-desarrollador/adding-proveedors) |

See the full [Pluggable interfaces table](/guia-usuario/features/complementos#pluggable-interfaces--where-to-go-for-each) for a consolidated view of every extension surface including config-driven (TTS, STT, MCP, shell hooks) and drop-in directory (puerta de enlace hooks) styles.
:::

## What you're building

A **calculator** complemento with two herramientas:
- `calculate` — evaluate math expressions (`2**16`, `sqrt(144)`, `pi * 5**2`)
- `unit_convert` — convert between units (`100 F → 37.78 C`, `5 km → 3.11 mi`)

Plus a hook that logs every herramienta call, and a bundled habilidad file.

## Step 1: Create the complemento directory

```bash
mkdir -p ~/.hermes/complementos/calculator
cd ~/.hermes/complementos/calculator
```

## Step 2: Write the manifest

Create `complemento.yaml`:

```yaml
name: calculator
version: 1.0.0
description: Math calculator — evaluate expressions and convert units
provides_herramientas:
  - calculate
  - unit_convert
provides_hooks:
  - post_herramienta_call
```

This tells Hermes: "I'm a complemento called calculator, I provide herramientas and hooks." The `provides_herramientas` and `provides_hooks` fields are lists of what the complemento registers.

Optional fields you could add:
```yaml
author: Your Name
requires_env:          # gate loading on env vars; prompted during install
  - SOME_API_KEY       # simple format — complemento disabled if missing
  - name: OTHER_KEY    # rich format — shows description/url during install
    description: "Key for the Other service"
    url: "https://other.com/keys"
    secret: true
```

## Step 3: Write the herramienta schemas

Create `schemas.py` — this is what the LLM reads to decide when to call your herramientas:

```python
"""Herramienta schemas — what the LLM sees."""

CALCULATE = {
    "name": "calculate",
    "description": (
        "Evaluate a mathematical expression and return the result. "
        "Supports arithmetic (+, -, *, /, **), functions (sqrt, sin, cos, "
        "log, abs, round, floor, ceil), and constants (pi, e). "
        "Use this for any math the user asks about."
    ),
    "parameters": {
        "type": "object",
        "properties": {
            "expression": {
                "type": "string",
                "description": "Math expression to evaluate (e.g., '2**10', 'sqrt(144)')",
            },
        },
        "required": ["expression"],
    },
}

UNIT_CONVERT = {
    "name": "unit_convert",
    "description": (
        "Convert a value between units. Supports length (m, km, mi, ft, in), "
        "weight (kg, lb, oz, g), temperature (C, F, K), data (B, KB, MB, GB, TB), "
        "and time (s, min, hr, day)."
    ),
    "parameters": {
        "type": "object",
        "properties": {
            "value": {
                "type": "number",
                "description": "The numeric value to convert",
            },
            "from_unit": {
                "type": "string",
                "description": "Source unit (e.g., 'km', 'lb', 'F', 'GB')",
            },
            "to_unit": {
                "type": "string",
                "description": "Target unit (e.g., 'mi', 'kg', 'C', 'MB')",
            },
        },
        "required": ["value", "from_unit", "to_unit"],
    },
}
```

**Why schemas matter:** The `description` field is how the LLM decides when to use your herramienta. Be specific about what it does and when to use it. The `parameters` define what arguments the LLM passes.

## Step 4: Write the herramienta handlers

Create `herramientas.py` — this is the code that actually executes when the LLM calls your herramientas:

```python
"""Herramienta handlers — the code that runs when the LLM calls each herramienta."""

import json
import math

# Safe globals for expression evaluation — no file/network access
_SAFE_MATH = {
    "abs": abs, "round": round, "min": min, "max": max,
    "pow": pow, "sqrt": math.sqrt, "sin": math.sin, "cos": math.cos,
    "tan": math.tan, "log": math.log, "log2": math.log2, "log10": math.log10,
    "floor": math.floor, "ceil": math.ceil,
    "pi": math.pi, "e": math.e,
    "factorial": math.factorial,
}


def calculate(args: dict, **kwargs) -> str:
    """Evaluate a math expression safely.

    Rules for handlers:
    1. Receive args (dict) — the parameters the LLM passed
    2. Do the work
    3. Return a JSON string — ALWAYS, even on error
    4. Accept **kwargs for forward compatibility
    """
    expression = args.get("expression", "").strip()
    if not expression:
        return json.dumps({"error": "No expression provided"})

    try:
        result = eval(expression, {"__builtins__": {}}, _SAFE_MATH)
        return json.dumps({"expression": expression, "result": result})
    except ZeroDivisionError:
        return json.dumps({"expression": expression, "error": "Division by zero"})
    except Exception as e:
        return json.dumps({"expression": expression, "error": f"Invalid: {e}"})


# Conversion tables — values are in base units
_LENGTH = {"m": 1, "km": 1000, "mi": 1609.34, "ft": 0.3048, "in": 0.0254, "cm": 0.01}
_WEIGHT = {"kg": 1, "g": 0.001, "lb": 0.453592, "oz": 0.0283495}
_DATA = {"B": 1, "KB": 1024, "MB": 1024**2, "GB": 1024**3, "TB": 1024**4}
_TIME = {"s": 1, "ms": 0.001, "min": 60, "hr": 3600, "day": 86400}


def _convert_temp(value, from_u, to_u):
    # Normalize to Celsius
    c = {"F": (value - 32) * 5/9, "K": value - 273.15}.get(from_u, value)
    # Convert to target
    return {"F": c * 9/5 + 32, "K": c + 273.15}.get(to_u, c)


def unit_convert(args: dict, **kwargs) -> str:
    """Convert between units."""
    value = args.get("value")
    from_unit = args.get("from_unit", "").strip()
    to_unit = args.get("to_unit", "").strip()

    if value is None or not from_unit or not to_unit:
        return json.dumps({"error": "Need value, from_unit, and to_unit"})

    try:
        # Temperature
        if from_unit.upper() in {"C","F","K"} and to_unit.upper() in {"C","F","K"}:
            result = _convert_temp(float(value), from_unit.upper(), to_unit.upper())
            return json.dumps({"input": f"{value} {from_unit}", "result": round(result, 4),
                             "output": f"{round(result, 4)} {to_unit}"})

        # Ratio-based conversions
        for table in (_LENGTH, _WEIGHT, _DATA, _TIME):
            lc = {k.lower(): v for k, v in table.items()}
            if from_unit.lower() in lc and to_unit.lower() in lc:
                result = float(value) * lc[from_unit.lower()] / lc[to_unit.lower()]
                return json.dumps({"input": f"{value} {from_unit}",
                                 "result": round(result, 6),
                                 "output": f"{round(result, 6)} {to_unit}"})

        return json.dumps({"error": f"Cannot convert {from_unit} → {to_unit}"})
    except Exception as e:
        return json.dumps({"error": f"Conversion failed: {e}"})
```

**Key rules for handlers:**
1. **Signature:** `def my_handler(args: dict, **kwargs) -> str`
2. **Return:** Always a JSON string. Success and errors alike.
3. **Never raise:** Catch all exceptions, return error JSON instead.
4. **Accept `**kwargs`:** Hermes may pass additional contexto in the future.

## Step 5: Write the registration

Create `__init__.py` — this wires schemas to handlers:

```python
"""Calculator complemento — registration."""

import logging

from . import schemas, herramientas

logger = logging.getLogger(__name__)

# Track herramienta usage via hooks
_call_log = []

def _on_post_herramienta_call(herramienta_name, args, result, task_id, **kwargs):
    """Hook: runs after every herramienta call (not just ours)."""
    _call_log.append({"herramienta": herramienta_name, "session": task_id})
    if len(_call_log) > 100:
        _call_log.pop(0)
    logger.debug("Herramienta called: %s (session %s)", herramienta_name, task_id)


def register(ctx):
    """Wire schemas to handlers and register hooks."""
    ctx.register_herramienta(name="calculate",    herramientaset="calculator",
                      schema=schemas.CALCULATE,    handler=herramientas.calculate)
    ctx.register_herramienta(name="unit_convert", herramientaset="calculator",
                      schema=schemas.UNIT_CONVERT, handler=herramientas.unit_convert)

    # This hook fires for ALL herramienta calls, not just ours
    ctx.register_hook("post_herramienta_call", _on_post_herramienta_call)
```

**What `register()` does:**
- Called exactly once at startup
- `ctx.register_herramienta()` puts your herramienta in the registry — the modelo sees it immediately
- `ctx.register_hook()` subscribes to lifecycle events
- `ctx.register_cli_command()` registers a CLI subcommand (e.g. `hermes my-complemento <subcommand>`)
- `ctx.register_command()` registers an in-session slash command (e.g. `/mycomplemento <args>` inside CLI / puerta de enlace chat) — see [Register slash commands](#register-slash-commands) below
- `ctx.dispatch_herramienta(name, arguments)` — call any other herramienta (built-in or from another complemento) with the parent agente's contexto (approvals, credentials, task_id) wired up automatically. Useful from slash-command handlers that need to invoke `terminal`, `read_file`, or any other herramienta as if the modelo had called it directly.
- If this function crashes, the complemento is disabled but Hermes continues fine

**`dispatch_herramienta` example — a slash command that runs a herramienta:**

```python
def handle_scan(ctx, raw_args: str):
    """Implement /scan by invoking the terminal herramienta through the registry."""
    result = ctx.dispatch_herramienta("terminal", {"command": f"find . -name '{raw_args}'"})
    return result  # returned to the caller's chat UI

def register(ctx):
    # Handlers receive a single raw_args string; close over ctx via a lambda.
    ctx.register_command(
        "scan",
        lambda raw: handle_scan(ctx, raw),
        description="Find files matching a glob",
    )
```

The dispatched herramienta goes through the normal approval, redaction, and budget pipelines — it's a real herramienta invocation, not a shortcut around them.

## Step 6: Test it

Start Hermes:

```bash
hermes
```

You should see `calculator: calculate, unit_convert` in the banner's herramienta list.

Try these prompts:
```
What's 2 to the power of 16?
Convert 100 fahrenheit to celsius
What's the square root of 2 times pi?
How many gigabytes is 1.5 terabytes?
```

Check complemento status:
```
/complementos
```

Output:
```
Complementos (1):
  ✓ calculator v1.0.0 (2 herramientas, 1 hooks)
```

### Debugging complemento discovery

If your complemento doesn't show up — or shows up but isn't loading — set `HERMES_PLUGINS_DEBUG=1` to get verbose discovery logs on stderr:

```bash
HERMES_PLUGINS_DEBUG=1 hermes complementos list
```

You'll see, for every complemento source (bundled, user, project, entry-points):

- which directories were scanned and how many manifests each yielded
- per manifest: resolved key, name, kind, source, on-disk path
- skip reasons: `disabled via config`, `not enabled in config`, `exclusive complemento`, `no complemento.yaml, depth cap reached`
- on load: the complemento being imported, plus a one-line summary of what `register(ctx)` registered (herramientas, hooks, slash commands, CLI commands)
- on parse failure: a full traceback for the exception (YAML scanner errors, etc.)
- on `register()` failure: a full traceback pointing at the line in your `__init__.py` that raised

The same logs are always written to `~/.hermes/logs/agente.log` at WARNING level (failures only) and DEBUG level (everything) when the env var is set. So if you can't run with the env var (e.g. from inside the puerta de enlace), tail the log file instead:

```bash
hermes logs --level WARNING | grep -i complemento
```

Common reasons a complemento doesn't appear:

- **Not enabled in config** — complementos are opt-in. Run `hermes complementos enable <name>` (the name comes from the `complementos list` output, which can be `<category>/<complemento>` for nested layouts).
- **Wrong directory layout** — must be `~/.hermes/complementos/<complemento-name>/complemento.yaml` (flat) or `~/.hermes/complementos/<category>/<complemento-name>/complemento.yaml` (one level of category nesting, max). Anything deeper is ignored.
- **Missing `__init__.py`** — the complemento directory needs both `complemento.yaml` and `__init__.py` with a `register(ctx)` function.
- **Wrong `kind`** — puerta de enlace adapters need `kind: platform` in their manifest. Memoria proveedors are auto-detected as `kind: exclusive` and routed through the `memoria.proveedor` config instead of `complementos.enabled`.

## Your complemento's final structure

```
~/.hermes/complementos/calculator/
├── complemento.yaml      # "I'm calculator, I provide herramientas and hooks"
├── __init__.py      # Wiring: schemas → handlers, register hooks
├── schemas.py       # What the LLM reads (descriptions + parameter specs)
└── herramientas.py         # What runs (calculate, unit_convert functions)
```

Four files, clear separation:
- **Manifest** declares what the complemento is
- **Schemas** describe herramientas for the LLM
- **Handlers** implement the actual logic
- **Registration** connects everything

## What else can complementos do?

### Ship data files

Put any files in your complemento directory and read them at import time:

```python
# In herramientas.py or __init__.py
from pathlib import Path

_PLUGIN_DIR = Path(__file__).parent
_DATA_FILE = _PLUGIN_DIR / "data" / "languages.yaml"

with open(_DATA_FILE) as f:
    _DATA = yaml.safe_load(f)
```

### Bundle habilidads

Complementos can ship habilidad files that the agente loads via `habilidad_view("complemento:habilidad")`. Register them in your `__init__.py`:

```
~/.hermes/complementos/my-complemento/
├── __init__.py
├── complemento.yaml
└── habilidads/
    ├── my-workflow/
    │   └── SKILL.md
    └── my-checklist/
        └── SKILL.md
```

```python
from pathlib import Path

def register(ctx):
    habilidads_dir = Path(__file__).parent / "habilidads"
    for child in sorted(habilidads_dir.iterdir()):
        habilidad_md = child / "SKILL.md"
        if child.is_dir() and habilidad_md.exists():
            ctx.register_habilidad(child.name, habilidad_md)
```

The agente can now load your habilidads with their namespaced name:

```python
habilidad_view("my-complemento:my-workflow")   # → complemento's version
habilidad_view("my-workflow")              # → built-in version (unchanged)
```

**Key properties:**
- Complemento habilidads are **read-only** — they don't enter `~/.hermes/habilidads/` and can't be edited via `habilidad_manage`.
- Complemento habilidads are **not** listed in the system prompt's `<available_habilidads>` index — they're opt-in explicit loads.
- Bare habilidad names are unaffected — the namespace prevents collisions with built-in habilidads.
- When the agente loads a complemento habilidad, a bundle contexto banner is prepended listing sibling habilidads from the same complemento.

:::tip Legacy pattern
The old `shutil.copy2` pattern (copying a habilidad into `~/.hermes/habilidads/`) still works but creates name collision risk with built-in habilidads. Prefer `ctx.register_habilidad()` for new complementos.
:::

### Gate on environment variables

If your complemento needs an API key:

```yaml
# complemento.yaml — simple format (backwards-compatible)
requires_env:
  - WEATHER_API_KEY
```

If `WEATHER_API_KEY` isn't set, the complemento is disabled with a clear message. No crash, no error in the agente — just "Complemento weather disabled (missing: WEATHER_API_KEY)".

When users run `hermes complementos install`, they're **prompted interactively** for any missing `requires_env` variables. Values are saved to `.env` automatically.

For a better install experience, use the rich format with descriptions and signup URLs:

```yaml
# complemento.yaml — rich format
requires_env:
  - name: WEATHER_API_KEY
    description: "API key for OpenWeather"
    url: "https://openweathermap.org/api"
    secret: true
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Environment variable name |
| `description` | No | Shown to user during install prompt |
| `url` | No | Where to get the credential |
| `secret` | No | If `true`, input is hidden (like a password field) |

Both formats can be mixed in the same list. Already-set variables are skipped silently.

### Lazy-install optional Python dependencies

If your complemento wraps an SDK that not every user will have installed (a vendor SDK, a heavy ML lib, a platform-specific package), don't `import` it at the top of the module. Use the `herramientas.lazy_deps.ensure(...)` helper inside the herramienta handler — Hermes will install the package on first use, gated by the user's `security.allow_lazy_installs` config.

```python
# herramientas.py
from herramientas.lazy_deps import ensure, FeatureUnavailable

def my_herramienta_handler(args, **kwargs):
    try:
        ensure("my-complemento.my-backend")   # key must be in LAZY_DEPS
    except FeatureUnavailable as exc:
        return {"error": str(exc)}

    import my_backend_sdk   # safe now
    ...
```

Two rules from the security modelo in `herramientas/lazy_deps.py`:

| Rule | Why |
|---|---|
| Your feature key must appear in the in-tree `LAZY_DEPS` allowlist | Prevents a malicious config from coaxing Hermes into installing arbitrary packages — only specs Hermes itself ships are eligible |
| Specs are PyPI-by-name only | No `--index-url`, `git+https://`, or file: paths. Pin versions with PEP 440 (`"my-sdk>=1.2,<2"`) inside the allowlist entry |

For third-party complementos distributed via pip, declare the optional deps as `[project.optional-dependencies]` extras in your own `pyproject.toml` and tell users to `pip install your-complemento[backend]` — that path doesn't go through `lazy_deps`. The lazy-install dance is most useful for **bundled** complementos where shipping a hard dependency on every install would bloat the base Hermes footprint.

When `security.allow_lazy_installs: false` is set globally, `ensure()` raises `FeatureUnavailable` immediately with a remediation hint — your complemento should catch it and degrade gracefully (return an error result, not crash the herramienta loop).



### Thread-safe lazy singletons

Complementos often cache an expensive object — an SDK client, an HTTP session, a connection pool — in a module-level variable built on first use:

```python
_client = None

def get_client():
    global _client
    if _client is not None:
        return _client
    _client = ExpensiveClient(...)   # ← TOCTOU race
    return _client
```

This is a footgun. Hermes runs multiple threads in one process (delegated herramienta calls, background workers, the self-improvement fork), so two threads can hit `get_client()` before `_client` is set, **both** pass the `is not None` check, **both** run the expensive build, and the second write clobbers the first — leaking whatever resource the loser opened (connection, file handle, background thread).

Don't hand-roll the lock. Use the helpers in `complementos/complemento_utils.py`:

```python
from complementos.complemento_utils import lazy_singleton, SingletonSlot

# Zero-arg accessor → decorate it:
@lazy_singleton
def get_client():
    return ExpensiveClient(load_config())   # runs exactly once

client = get_client()    # safe across threads
get_client.reset()       # drop the instance (tests / teardown)


# Accessor that takes a build argument → use a slot:
_slot: SingletonSlot = SingletonSlot()

def get_client(config=None):
    return _slot.get(lambda: ExpensiveClient(resolve(config)))

def reset_client():
    _slot.reset()
```

Both serialize concurrent first calls with double-checked locking and run the factory at most once. If the factory raises, nothing is cached and the next call retries. The honcho memoria complemento (`complementos/memoria/honcho/client.py`) is the reference consumer.

> Rule of thumb: any time you write `global _something` followed by a `is None` check and a build, reach for one of these instead.



### Conditional herramienta availability

For herramientas that depend on optional libraries:

```python
ctx.register_herramienta(
    name="my_herramienta",
    schema={...},
    handler=my_handler,
    check_fn=lambda: _has_optional_lib(),  # False = herramienta hidden from modelo
)
```

### Overriding a built-in herramienta

To replace a built-in herramienta with your own implementation (e.g. swap the
default browser herramienta for a headed-Chrome CDP backend, or replace
`web_search` with a custom corporate index), pass `override=True`:

```python
def register(ctx):
    ctx.register_herramienta(
        name="browser_navigate",             # same name as the built-in
        herramientaset="complemento_my_browser",         # your own herramientaset namespace
        schema={...},
        handler=my_custom_navigate,
        override=True,                       # explicit opt-in
    )
```

Without `override=True`, the registry rejects any registration that would
shadow an existing herramienta from a different herramientaset — this prevents
accidental overwrites. The override is logged at INFO level so it's
auditable in `~/.hermes/logs/agente.log`. Complementos load after built-in
herramientas, so the registration order is correct: your handler replaces the
built-in one.

### Register multiple hooks

```python
def register(ctx):
    ctx.register_hook("pre_herramienta_call", before_any_herramienta)
    ctx.register_hook("post_herramienta_call", after_any_herramienta)
    ctx.register_hook("pre_llm_call", inject_memoria)
    ctx.register_hook("on_session_start", on_new_session)
    ctx.register_hook("on_session_end", on_session_end)
```

### Hook reference

Each hook is documented in full on the **[Event Hooks reference](/guia-usuario/features/hooks#complemento-hooks)** — callback signatures, parameter tables, exactly when each fires, and examples. Here's the summary:

| Hook | Fires when | Callback signature | Returns |
|------|-----------|-------------------|---------|
| [`pre_herramienta_call`](/guia-usuario/features/hooks#pre_herramienta_call) | Before any herramienta executes | `herramienta_name: str, args: dict, task_id: str` | ignored |
| [`post_herramienta_call`](/guia-usuario/features/hooks#post_herramienta_call) | After any herramienta returns | `herramienta_name: str, args: dict, result: str, task_id: str, duration_ms: int` | ignored |
| [`pre_llm_call`](/guia-usuario/features/hooks#pre_llm_call) | Once per turn, before the herramienta-calling loop | `session_id: str, user_message: str, conversation_history: list, is_first_turn: bool, modelo: str, platform: str` | [contexto injection](#pre_llm_call-contexto-injection) |
| [`post_llm_call`](/guia-usuario/features/hooks#post_llm_call) | Once per turn, after the herramienta-calling loop (successful turns only) | `session_id: str, user_message: str, assistant_response: str, conversation_history: list, modelo: str, platform: str` | ignored |
| [`on_session_start`](/guia-usuario/features/hooks#on_session_start) | New session created (first turn only) | `session_id: str, modelo: str, platform: str` | ignored |
| [`on_session_end`](/guia-usuario/features/hooks#on_session_end) | End of every `run_conversation` call + CLI exit | `session_id: str, completed: bool, interrupted: bool, modelo: str, platform: str` | ignored |
| [`on_session_finalize`](/guia-usuario/features/hooks#on_session_finalize) | CLI/puerta de enlace tears down an active session | `session_id: str \| None, platform: str` | ignored |
| [`on_session_reset`](/guia-usuario/features/hooks#on_session_reset) | Puerta de enlace swaps in a new session key (`/new`, `/reset`) | `session_id: str, platform: str` | ignored |
| `kanban_task_claimed` | A kanban task is claimed (dispatcher process, before the worker spawns) | `task_id: str, board: str \| None, assignee: str \| None, run_id: int \| None, perfil_name: str` | ignored |
| `kanban_task_completed` | A kanban task completes (worker process) | `task_id, board, assignee, run_id, perfil_name, summary: str \| None` | ignored |
| `kanban_task_blocked` | A kanban task is blocked (worker process) | `task_id, board, assignee, run_id, perfil_name, reason: str \| None` | ignored |

Most hooks are fire-and-forget observers — their return values are ignored. The exception is `pre_llm_call`, which can inject contexto into the conversation.

All callbacks should accept `**kwargs` for forward compatibility. If a hook callback crashes, it's logged and skipped. Other hooks and the agente continue normally.

The kanban lifecycle hooks fire **after** the board DB change commits, so a callback always sees durable state and can never hold the SQLite write lock. Because kanban workers run as separate `hermes -p <perfil> chat -q` subprocesses, `kanban_task_claimed` fires in the **dispatcher** process while `kanban_task_completed` / `kanban_task_blocked` fire in the **worker** process — hook in the dispatcher to observe every transition centrally, or in the worker for per-task in-session contexto.

### `pre_llm_call` contexto injection

This is the only hook whose return value matters. When a `pre_llm_call` callback returns a dict with a `"contexto"` key (or a plain string), Hermes injects that text into the **current turn's user message**. This is the mechanism for memoria complementos, RAG integrations, guardrails, and any complemento that needs to provide the modelo with additional contexto.

#### Return format

```python
# Dict with contexto key
return {"contexto": "Recalled memories:\n- User prefers dark mode\n- Last project: hermes-agente"}

# Plain string (equivalent to the dict form above)
return "Recalled memories:\n- User prefers dark mode"

# Return None or don't return → no injection (observer-only)
return None
```

Any non-None, non-empty return with a `"contexto"` key (or a plain non-empty string) is collected and appended to the user message for the current turn.

#### How injection works

Injected contexto is appended to the **user message**, not the system prompt. This is a deliberate design choice:

- **Prompt cache preservation** — the system prompt stays identical across turns. Anthropic and OpenRouter cache the system prompt prefix, so keeping it stable saves 75%+ on input tokens in multi-turn conversations. If complementos modified the system prompt, every turn would be a cache miss.
- **Ephemeral** — the injection happens at API call time only. The original user message in the conversation history is never mutated, and nothing is persisted to the session database.
- **The system prompt is Hermes's territory** — it contains modelo-specific guidance, herramienta enforcement rules, personality instructions, and cached habilidad content. Complementos contribute contexto alongside the user's input, not by altering the agente's core instructions.

#### Example: Memoria recall complemento

```python
"""Memoria complemento — recalls relevant contexto from a vector store."""

import httpx

MEMORY_API = "https://your-memoria-api.example.com"

def recall_contexto(session_id, user_message, is_first_turn, **kwargs):
    """Called before each LLM turn. Returns recalled memories."""
    try:
        resp = httpx.post(f"{MEMORY_API}/recall", json={
            "session_id": session_id,
            "query": user_message,
        }, timeout=3)
        memories = resp.json().get("results", [])
        if not memories:
            return None  # nothing to inject

        text = "Recalled contexto from previous sessions:\n"
        text += "\n".join(f"- {m['text']}" for m in memories)
        return {"contexto": text}
    except Exception:
        return None  # fail silently, don't break the agente

def register(ctx):
    ctx.register_hook("pre_llm_call", recall_contexto)
```

#### Example: Guardrails complemento

```python
"""Guardrails complemento — enforces content policies."""

POLICY = """You MUST follow these content policies for this session:
- Never generate code that accesses the filesystem outside the working directory
- Always warn before executing destructive operations
- Refuse requests involving personal data extraction"""

def inject_guardrails(**kwargs):
    """Injects policy text into every turn."""
    return {"contexto": POLICY}

def register(ctx):
    ctx.register_hook("pre_llm_call", inject_guardrails)
```

#### Example: Observer-only hook (no injection)

```python
"""Analytics complemento — tracks turn metadata without injecting contexto."""

import logging
logger = logging.getLogger(__name__)

def log_turn(session_id, user_message, modelo, is_first_turn, **kwargs):
    """Fires before each LLM call. Returns None — no contexto injected."""
    logger.info("Turn: session=%s modelo=%s first=%s msg_len=%d",
                session_id, modelo, is_first_turn, len(user_message or ""))
    # No return → no injection

def register(ctx):
    ctx.register_hook("pre_llm_call", log_turn)
```

#### Multiple complementos returning contexto

When multiple complementos return contexto from `pre_llm_call`, their outputs are joined with double newlines and appended to the user message together. The order follows complemento discovery order (alphabetical by complemento directory name).

### Register CLI commands

Complementos can add their own `hermes <complemento>` subcommand tree:

```python
def _my_command(args):
    """Handler for hermes my-complemento <subcommand>."""
    sub = getattr(args, "my_command", None)
    if sub == "status":
        print("All good!")
    elif sub == "config":
        print("Current config: ...")
    else:
        print("Usage: hermes my-complemento <status|config>")

def _setup_argparse(subparser):
    """Build the argparse tree for hermes my-complemento."""
    subs = subparser.add_subparsers(dest="my_command")
    subs.add_parser("status", help="Show complemento status")
    subs.add_parser("config", help="Show complemento config")
    subparser.set_defaults(func=_my_command)

def register(ctx):
    ctx.register_herramienta(...)
    ctx.register_cli_command(
        name="my-complemento",
        help="Manage my complemento",
        setup_fn=_setup_argparse,
        handler_fn=_my_command,
    )
```

After registration, users can run `hermes my-complemento status`, `hermes my-complemento config`, etc.

**Memoria proveedor complementos** use a convention-based approach instead: add a `register_cli(subparser)` function to your complemento's `cli.py` file. The memoria complemento discovery system finds it automatically — no `ctx.register_cli_command()` call needed. See the [Memoria Proveedor Complemento guide](/guia-desarrollador/memoria-proveedor-complemento#adding-cli-commands) for details.

**Active-proveedor gating:** Memoria complemento CLI commands only appear when their proveedor is the active `memoria.proveedor` in config. If a user hasn't set up your proveedor, your CLI commands won't clutter the help output.

### Register slash commands

Complementos can register in-session slash commands — commands users type during a conversation (like `/lcm status` or `/ping`). These work in both CLI and puerta de enlace (Telegram, Discord, etc.).

```python
def _handle_status(raw_args: str) -> str:
    """Handler for /mystatus — called with everything after the command name."""
    if raw_args.strip() == "help":
        return "Usage: /mystatus [help|check]"
    return "Complemento status: all systems nominal"

def register(ctx):
    ctx.register_command(
        "mystatus",
        handler=_handle_status,
        description="Show complemento status",
    )
```

After registration, users can type `/mystatus` in any session. The command appears in autocomplete, `/help` output, and the Telegram bot menu.

**Signature:** `ctx.register_command(name: str, handler: Callable, description: str = "", args_hint: str = "")`

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | `str` | Command name without the leading slash (e.g. `"lcm"`, `"mystatus"`) |
| `handler` | `Callable[[str], str \| None]` | Called with the raw argument string. May also be `async`. |
| `description` | `str` | Shown in `/help`, autocomplete, and Telegram bot menu |

**Key differences from `register_cli_command()`:**

| | `register_command()` | `register_cli_command()` |
|---|---|---|
| Invoked as | `/name` in a session | `hermes name` in a terminal |
| Where it works | CLI sessions, Telegram, Discord, etc. | Terminal only |
| Handler receives | Raw args string | argparse `Namespace` |
| Use case | Diagnostics, status, quick actions | Complex subcommand trees, setup wizards |

**Conflict protection:** If a complemento tries to register a name that conflicts with a built-in command (`help`, `modelo`, `new`, etc.), the registration is silently rejected with a log warning. Built-in commands always take precedence.

**Async handlers:** The puerta de enlace dispatch automatically detects and awaits async handlers, so you can use either sync or async functions:

```python
async def _handle_check(raw_args: str) -> str:
    result = await some_async_operation()
    return f"Check result: {result}"

def register(ctx):
    ctx.register_command("check", handler=_handle_check, description="Run async check")
```

### Dispatch herramientas from slash commands

Slash command handlers that need to orchestrate herramientas (spawn a subagente via `delegate_task`, call `file_edit`, etc.) should use `ctx.dispatch_herramienta()` instead of reaching into framework internals. The parent-agente contexto (workspace hints, spinner, modelo inheritance) is wired up automatically.

```python
def register(ctx):
    def _handle_deliver(raw_args: str):
        result = ctx.dispatch_herramienta(
            "delegate_task",
            {
                "goal": raw_args,
                "herramientasets": ["terminal", "file", "web"],
            },
        )
        return result

    ctx.register_command(
        "deliver",
        handler=_handle_deliver,
        description="Delegate a goal to a subagente",
    )
```

**Signature:** `ctx.dispatch_herramienta(name: str, args: dict, *, parent_agente=None) -> str`

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | `str` | Herramienta name as registered in the herramienta registry (e.g. `"delegate_task"`, `"file_edit"`) |
| `args` | `dict` | Herramienta arguments, same shape the modelo would send |
| `parent_agente` | `Agente \| None` | Optional override. When omitted, resolves from the current CLI agente (or degrades gracefully in puerta de enlace mode) |

**Runtime behavior:**

- **CLI mode:** `parent_agente` is resolved from the active CLI agente so workspace hints, spinner, and modelo selection inherit as expected.
- **Puerta de enlace mode:** There is no CLI agente, so herramientas degrade gracefully — workspace is read from the configured terminal working directory and no spinner is shown.
- **Explicit override:** If the caller passes `parent_agente=` explicitly, it is respected and not overwritten.

This is the public, stable interface for herramienta dispatch from complemento commands. Complementos should not reach into `ctx._cli_ref.agente` or similar private state.

### Act from inside a hook (perfil + herramientas)

`ctx._cli_ref` is only populated in an **interactive CLI** session. It is `None` in the puerta de enlace, in non-interactive `hermes chat -q` runs, and in **kanban-spawned worker sessions** — so any complemento logic that reaches through `_cli_ref` silently no-ops in exactly those contextos. Two stable, session-agnostic APIs cover what hooks actually need:

- **`ctx.perfil_name`** — the active perfil name (e.g. `"default"`, or the assignee perfil in a kanban worker). Derived from `HERMES_HOME`, so it works everywhere with no `_cli_ref` dependency.
- **`ctx.dispatch_herramienta(name, args)`** — invoke any registered herramienta (built-in or complemento), including the `kanban_*` herramientas, `delegate_task`, `terminal`, `read_file`, etc. Works from hook callbacks regardless of which process the hook fires in.

Together these let a kanban lifecycle hook observe a transition and act on the board without touching framework internals:

```python
def register(ctx):
    def on_blocked(*, task_id, reason=None, **kw):
        # Runs in the worker process; ctx._cli_ref is None here.
        ctx.dispatch_herramienta("kanban_comment", {
            "task_id": task_id,
            "comment": f"[{ctx.perfil_name}] auto-noted block: {reason}",
        })
    ctx.register_hook("kanban_task_blocked", on_blocked)
```

For running a full `hermes <subcommand>` (e.g. `hermes kanban show`), shell out with the `terminal` herramienta via `ctx.dispatch_herramienta("terminal", {"command": "hermes kanban show ..."})` — there is no in-process slash-command bridge for headless worker sessions, and herramientas are the supported way to drive Hermes from a hook.

### Handle Slack Block Kit button clicks

Complementos that post Block Kit messages with interactive elements (buttons, overflow menus, datepickers, etc.) can register the click handlers directly with the Slack adapter — no monkey-patching of `slack_bolt.AsyncApp` required.

```python
def register(ctx):
    async def _on_approve(ack, body, action):
        # ack within 3 seconds — slack_bolt requirement.
        await ack()
        # body["channel"]["id"], body["user"]["id"], body["message"]["ts"]
        # action["action_id"], action["value"]
        sweep_id = (action.get("value") or "").split("|", 1)[-1]
        # ...do the deterministic work, then post a follow-up.

    ctx.register_slack_action_handler("inbox_sweep_approve", _on_approve)
```

**Signature:** `ctx.register_slack_action_handler(action_id, callback) -> None`

| Parameter | Type | Description |
|-----------|------|-------------|
| `action_id` | `str \| re.Pattern \| dict` | Whatever `slack_bolt.App.action()` accepts: a literal `action_id`, a compiled regex matching multiple ids, or a constraint dict like `{"action_id": "...", "block_id": "..."}` |
| `callback` | async callable | Receives `(ack, body, action)` per the slack_bolt convention |

**Runtime behavior:**

- The handler is queued at complemento-load time and wired into the adapter's `slack_bolt.AsyncApp` when the Slack platform connects.
- Each callback is wrapped defensively: if your handler raises, the puerta de enlace logs the error and best-effort-acks the click so Slack stops retrying.
- Standard slack_bolt rules apply — `await ack()` within 3 seconds, then do longer work.
- For multi-workspace deployments the handler fires for clicks from any connected workspace; use `body["team"]["id"]` if you need to scope behaviour.

This is the public way for complementos to participate in Slack interactivity. Older complementos may patch `SlackAdapter.connect`; prefer this API instead.

:::tip
This guide covers **general complementos** (herramientas, hooks, slash commands, CLI commands). The sections below sketch the authoring pattern for each specialized complemento type; each links to its full guide for field reference and examples.
:::

## Specialized complemento types

Hermes has five specialized complemento types beyond the general surface. Each ships as a directory under `complementos/<category>/<name>/` (bundled) or `~/.hermes/complementos/<category>/<name>/` (user). The contract differs by category — pick the one you need, then read its full guide.

### Modelo proveedor complementos — add an LLM backend

Drop a perfil into `complementos/modelo-proveedors/<name>/`:

```python
# complementos/modelo-proveedors/acme/__init__.py
from proveedors import register_proveedor
from proveedors.base import ProveedorPerfil

register_proveedor(ProveedorPerfil(
    name="acme",
    aliases=("acme-inference",),
    display_name="Acme Inference",
    env_vars=("ACME_API_KEY", "ACME_BASE_URL"),
    base_url="https://api.acme.example.com/v1",
    auth_type="api_key",
    default_aux_modelo="acme-small-fast",
    fallback_modelos=("acme-large-v3", "acme-medium-v3"),
))
```

```yaml
# complementos/modelo-proveedors/acme/complemento.yaml
name: acme-proveedor
kind: modelo-proveedor
version: 1.0.0
description: Acme Inference — OpenAI-compatible direct API
```

Lazy-discovered the first time anything calls `get_proveedor_perfil()` or `list_proveedors()` — `auth.py`, `config.py`, `doctor.py`, `modelos.py`, `runtime_proveedor.py`, and the chat_completions transport auto-wire to it. User complementos override bundled ones by name.

**Full guide:** [Modelo Proveedor Complementos](/guia-desarrollador/modelo-proveedor-complemento) — field reference, overridable hooks (`prepare_messages`, `build_extra_body`, `build_api_kwargs_extras`, `fetch_modelos`), api_mode selection, auth types, testing.

### Platform complementos — add a puerta de enlace channel

Drop an adapter into `complementos/platforms/<name>/`:

```python
# complementos/platforms/myplatform/adapter.py
from puerta de enlace.platforms.base import BasePlatformAdapter

class MyPlatformAdapter(BasePlatformAdapter):
    async def connect(self): ...
    async def send(self, chat_id, text): ...
    async def disconnect(self): ...

def check_requirements():
    import os
    return bool(os.environ.get("MYPLATFORM_TOKEN"))

def _env_enablement():
    import os
    tok = os.getenv("MYPLATFORM_TOKEN", "").strip()
    if not tok:
        return None
    return {"token": tok}

def register(ctx):
    ctx.register_platform(
        name="myplatform",
        label="MyPlatform",
        adapter_factory=lambda cfg: MyPlatformAdapter(cfg),
        check_fn=check_requirements,
        required_env=["MYPLATFORM_TOKEN"],
        # Auto-populate PlatformConfig.extra from env so env-only setups
        # show up in `hermes puerta de enlace status` without SDK instantiation.
        env_enablement_fn=_env_enablement,
        # Opt in to cron delivery: `deliver=myplatform` routes to this var.
        cron_deliver_env_var="MYPLATFORM_HOME_CHANNEL",
        emoji="💬",
        platform_hint="You are chatting via MyPlatform. Keep responses concise.",
    )
```

```yaml
# complementos/platforms/myplatform/complemento.yaml
name: myplatform-platform
label: MyPlatform
kind: platform
version: 1.0.0
description: MyPlatform puerta de enlace adapter
requires_env:
  - name: MYPLATFORM_TOKEN
    description: "Bot token from the MyPlatform console"
    password: true
optional_env:
  - name: MYPLATFORM_HOME_CHANNEL
    description: "Default channel for cron delivery"
    password: false
```

**Full guide:** [Adding Platform Adapters](/guia-desarrollador/adding-platform-adapters) — complete `BasePlatformAdapter` contract, message routing, auth gating, setup wizard integration. Look at `complementos/platforms/irc/` for a stdlib-only working example.

### Memoria proveedor complementos — add a cross-session knowledge backend

Drop an implementation of `MemoriaProveedor` into `complementos/memoria/<name>/`:

```python
# complementos/memoria/my-memoria/__init__.py
from agente.memoria_proveedor import MemoriaProveedor

class MyMemoriaProveedor(MemoriaProveedor):
    @property
    def name(self) -> str:
        return "my-memoria"

    def is_available(self) -> bool:
        import os
        return bool(os.environ.get("MY_MEMORY_API_KEY"))

    def initialize(self, session_id: str, **kwargs) -> None:
        self._session_id = session_id

    def sync_turn(self, user_content, assistant_content, *,
                  session_id="", messages=None) -> None:
        ...

    def prefetch(self, query, *, session_id="") -> str:
        ...

    def get_herramienta_schemas(self) -> list[dict]:
        return []   # required @abstractmethod — see full guide

def register(ctx):
    ctx.register_memoria_proveedor(MyMemoriaProveedor())
```

Memoria proveedors are single-select — only one is active at a time, chosen via `memoria.proveedor` in `config.yaml`.

**Full guide:** [Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento) — full `MemoriaProveedor` ABC, threading contract, perfil isolation, CLI command registration via `cli.py`.

### Contexto engine complementos — replace the contexto compressor

```python
# complementos/contexto_engine/my-engine/__init__.py
from agente.contexto_engine import ContextoEngine

class MyContextoEngine(ContextoEngine):
    @property
    def name(self) -> str:
        return "my-engine"

    def update_from_response(self, usage) -> None: ...
    def should_compress(self, prompt_tokens: int = None) -> bool: ...
    def compress(self, messages, current_tokens=None, focus_topic=None) -> list: ...

def register(ctx):
    ctx.register_contexto_engine(MyContextoEngine())
```

Contexto engines are single-select — chosen via `contexto.engine` in `config.yaml`.

**Full guide:** [Contexto Engine Complementos](/guia-desarrollador/contexto-engine-complemento).

### Image-generation backends

Drop a proveedor into `complementos/image_gen/<name>/`:

```python
# complementos/image_gen/my-imggen/__init__.py
from agente.image_gen_proveedor import ImageGenProveedor

class MyImageGenProveedor(ImageGenProveedor):
    @property
    def name(self) -> str:
        return "my-imggen"

    def is_available(self) -> bool: ...
    def generate(self, prompt: str, aspect_ratio="landscape", **kwargs) -> dict:
        # returns success_response(...) / error_response(...)
        ...

def register(ctx):
    ctx.register_image_gen_proveedor(MyImageGenProveedor())
```

```yaml
# complementos/image_gen/my-imggen/complemento.yaml
name: my-imggen
kind: backend
version: 1.0.0
description: Custom image generation backend
```

**Full guide:** [Image Generation Proveedor Complementos](/guia-desarrollador/image-gen-proveedor-complemento) — full `ImageGenProveedor` ABC, `list_modelos()` / `get_setup_schema()` metadata, `success_response()`/`error_response()` helpers, base64 vs URL output, user overrides, pip distribution.

**Reference examples:** `complementos/image_gen/openai/` (DALL-E / GPT-Image via OpenAI SDK), `complementos/image_gen/openai-codex/`, `complementos/image_gen/xai/` (Grok image gen).

## Non-Python extension surfaces

Hermes also accepts extensions that aren't Python complementos at all. These are shown in the [Pluggable interfaces table](/guia-usuario/features/complementos#pluggable-interfaces--where-to-go-for-each); the sections below sketch each authoring style briefly.

### MCP servers — register external herramientas

Modelo Contexto Protocol (MCP) servers register their own herramientas into Hermes without any Python complemento. Declare them in `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  filesystem:
    command: "npx"
    args: ["-y", "@modelocontextoprotocol/server-filesystem", "/home/user/projects"]
    timeout: 120

  linear:
    url: "https://mcp.linear.app/sse"
    auth:
      type: "oauth"
```

Hermes connects to each server at startup, lists its herramientas, and registers them alongside built-ins. The LLM sees them exactly like any other herramienta. **Full guide:** [MCP](/guia-usuario/features/mcp).

### Puerta de enlace event hooks — fire on lifecycle events

Drop a manifest + handler into `~/.hermes/hooks/<name>/`:

```yaml
# ~/.hermes/hooks/long-task-alert/HOOK.yaml
name: long-task-alert
description: Send a push notification when a long task finishes
events:
  - agente:end
```

```python
# ~/.hermes/hooks/long-task-alert/handler.py
async def handle(event_type: str, contexto: dict) -> None:
    if contexto.get("duration_seconds", 0) > 120:
        # send notification …
        pass
```

Events include `puerta de enlace:startup`, `session:start`, `session:end`, `session:reset`, `agente:start`, `agente:step`, `agente:end`, and wildcard `command:*`. Errors in hooks are caught and logged — they never block the main pipeline.

**Full guide:** [Puerta de enlace Event Hooks](/guia-usuario/features/hooks#puerta de enlace-event-hooks).

### Shell hooks — run a shell command on herramienta calls

If you just want to run a script when a herramienta fires (notifications, audit logs, desktop alerts, auto-formatters), use shell hooks in `config.yaml` — no Python required:

```yaml
hooks:
  - event: post_herramienta_call
    command: "notify-send 'Herramienta ran: {herramienta_name}'"
    when:
      herramientas: [terminal, patch, write_file]
```

Supports all the same events as Python complemento hooks (`pre_herramienta_call`, `post_herramienta_call`, `pre_llm_call`, `post_llm_call`, `on_session_start`, `on_session_end`, `pre_puerta de enlace_dispatch`) plus structured JSON output for `pre_herramienta_call` blocking decisions.

**Full guide:** [Shell Hooks](/guia-usuario/features/hooks#shell-hooks).

### Habilidad sources — add a custom habilidad registry

If you maintain a GitHub repo of habilidads (or want to pull from a community index beyond the built-in sources), add it as a **tap**:

```bash
hermes habilidads tap add myorg/habilidads-repo
hermes habilidads search my-workflow --source myorg/habilidads-repo
hermes habilidads install myorg/habilidads-repo/my-workflow
```

Publishing your own tap is just a GitHub repo with `habilidads/<habilidad-name>/SKILL.md` directories — no server or registry signup needed.

**Full guides:** [Habilidads Hub](/guia-usuario/features/habilidads#habilidads-hub) · [Publishing a custom tap](/guia-usuario/features/habilidads#publishing-a-custom-habilidad-tap) (repo layout, minimal example, non-default paths, trust levels).

### TTS / STT via command templates

Any CLI that reads/writes audio or text can be plugged in through `config.yaml` — no Python code:

```yaml
tts:
  proveedor: voxcpm
  proveedors:
    voxcpm:
      type: command
      command: "voxcpm --ref ~/voice.wav --text-file {input_path} --out {output_path}"
      output_format: mp3
      voice_compatible: true
```

For STT, point `HERMES_LOCAL_STT_COMMAND` at a shell template. Supported placeholders: `{input_path}`, `{output_path}`, `{format}`, `{voice}`, `{modelo}`, `{speed}` (TTS); `{input_path}`, `{output_dir}`, `{language}`, `{modelo}` (STT). Any path-interacting CLI is automatically a complemento.

**Full guides:** [TTS custom command proveedors](/guia-usuario/features/tts#custom-command-proveedors) · [STT](/guia-usuario/features/tts#voice-message-transcription-stt).

## Distribute via pip

For sharing complementos publicly, add an entry point to your Python package:

```toml
# pyproject.toml
[project.entry-points."hermes_agente.complementos"]
my-complemento = "my_complemento_package"
```

```bash
pip install hermes-complemento-calculator
# Complemento auto-discovered on next hermes startup
```

## Distribute for NixOS

NixOS users can install your complemento declaratively if you provide a `pyproject.toml` with entry points:

**Entry-point complementos** (recommended for distribution):
```nix
# User's configuración.nix
services.hermes-agente.extraPythonPackages = [
  (pkgs.python312Packages.buildPythonPackage {
    pname = "my-complemento";
    version = "1.0.0";
    src = pkgs.fetchFromGitHub {
      owner = "you";
      repo = "hermes-my-complemento";
      rev = "v1.0.0";
      hash = "sha256-...";  # nix-prefetch-url --unpack
    };
    format = "pyproject";
    build-system = [ pkgs.python312Packages.setupherramientas ];
  })
];
```

**Directory complementos** (no `pyproject.toml` needed):
```nix
services.hermes-agente.extraComplementos = [
  (pkgs.fetchFromGitHub {
    owner = "you";
    repo = "hermes-my-complemento";
    rev = "v1.0.0";
    hash = "sha256-...";
  })
];
```

See the [Nix Setup guide](/inicio-rapido/nix-setup#complementos) for complete documentation including overlay usage and collision checking.

## Common mistakes

**Handler doesn't return JSON string:**
```python
# Wrong — returns a dict
def handler(args, **kwargs):
    return {"result": 42}

# Right — returns a JSON string
def handler(args, **kwargs):
    return json.dumps({"result": 42})
```

**Missing `**kwargs` in handler signature:**
```python
# Wrong — will break if Hermes passes extra contexto
def handler(args):
    ...

# Right
def handler(args, **kwargs):
    ...
```

**Handler raises exceptions:**
```python
# Wrong — exception propagates, herramienta call fails
def handler(args, **kwargs):
    result = 1 / int(args["value"])  # ZeroDivisionError!
    return json.dumps({"result": result})

# Right — catch and return error JSON
def handler(args, **kwargs):
    try:
        result = 1 / int(args.get("value", 0))
        return json.dumps({"result": result})
    except Exception as e:
        return json.dumps({"error": str(e)})
```

**Schema description too vague:**
```python
# Bad — modelo doesn't know when to use it
"description": "Does stuff"

# Good — modelo knows exactly when and how
"description": "Evaluate a mathematical expression. Use for arithmetic, trig, logarithms. Supports: +, -, *, /, **, sqrt, sin, cos, log, pi, e."
```

---