<!-- source: website/docs/guia-desarrollador/adding-herramientas.md -->
# Adding Herramientas

# Adding Herramientas

Before writing a herramienta, ask yourself: **should this be a [habilidad](creating-habilidads.md) instead?**

:::warning Built-in Core Herramientas Only
This page is for adding a **built-in Hermes herramienta** to the repository itself.
If you want a personal, project-local, or otherwise custom herramienta without
modifying Hermes core, use the complemento route instead:

- [Complementos](/guia-usuario/features/complementos)
- [Build a Hermes Complemento](/guides/build-a-hermes-complemento)

Default to complementos for most custom herramienta creation. Only follow this page when
you explicitly want to ship a new built-in herramienta in `herramientas/` and `herramientasets.py`.
:::

Make it a **Habilidad** when the capability can be expressed as instructions + shell commands + existing herramientas (arXiv search, git workflows, Docker management, PDF processing).

Make it a **Herramienta** when it requires end-to-end integration with API keys, custom processing logic, binary data handling, or streaming (browser automation, TTS, vision analysis).

## Descripción General

Adding a herramienta touches **2 files**:

1. **`herramientas/your_herramienta.py`** — handler, schema, check function, `registry.register()` call
2. **`herramientasets.py`** — add herramienta name to `_HERMES_CORE_TOOLS` (or a specific herramientaset)

Any `herramientas/*.py` file with a top-level `registry.register()` call is auto-discovered at startup — no manual import list required.

## Step 1: Create the Built-in Herramienta File

Every herramienta file follows the same structure:

```python
# herramientas/weather_herramienta.py
"""Weather Herramienta -- look up current weather for a location."""

import json
import os
import logging

logger = logging.getLogger(__name__)


# --- Availability check ---

def check_weather_requirements() -> bool:
    """Return True if the herramienta's dependencies are available."""
    return bool(os.getenv("WEATHER_API_KEY"))


# --- Handler ---

def weather_herramienta(location: str, units: str = "metric") -> str:
    """Fetch weather for a location. Returns JSON string."""
    api_key = os.getenv("WEATHER_API_KEY")
    if not api_key:
        return json.dumps({"error": "WEATHER_API_KEY not configured"})
    try:
        # ... call weather API ...
        return json.dumps({"location": location, "temp": 22, "units": units})
    except Exception as e:
        return json.dumps({"error": str(e)})


# --- Schema ---

WEATHER_SCHEMA = {
    "name": "weather",
    "description": "Get current weather for a location.",
    "parameters": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "City name or coordinates (e.g. 'London' or '51.5,-0.1')"
            },
            "units": {
                "type": "string",
                "enum": ["metric", "imperial"],
                "description": "Temperature units (default: metric)",
                "default": "metric"
            }
        },
        "required": ["location"]
    }
}


# --- Registration ---

from herramientas.registry import registry

registry.register(
    name="weather",
    herramientaset="weather",
    schema=WEATHER_SCHEMA,
    handler=lambda args, **kw: weather_herramienta(
        location=args.get("location", ""),
        units=args.get("units", "metric")),
    check_fn=check_weather_requirements,
    requires_env=["WEATHER_API_KEY"],
)
```

### Key Rules

:::danger Important
- Handlers **MUST** return a JSON string (via `json.dumps()`), never raw dicts
- Errors **MUST** be returned as `{"error": "message"}`, never raised as exceptions
- The `check_fn` is called when building herramienta definitions — if it returns `False`, the herramienta is silently excluded
- The `handler` receives `(args: dict, **kwargs)` where `args` is the LLM's herramienta call arguments
:::

## Step 2: Add the Built-in Herramienta to a Herramientaset

In `herramientasets.py`, add the herramienta name:

```python
# If it should be available on all platforms (CLI + messaging):
_HERMES_CORE_TOOLS = [
    ...
    "weather",  # <-- add here
]

# Or create a new standalone herramientaset:
"weather": {
    "description": "Weather lookup herramientas",
    "herramientas": ["weather"],
    "includes": []
},
```

## ~~Step 3: Add Discovery Import~~ (No longer needed)

Herramienta modules with a top-level `registry.register()` call are auto-discovered by `discover_builtin_herramientas()` in `herramientas/registry.py`. No manual import list to maintain — just create your file in `herramientas/` and it's picked up at startup.

## Async Handlers

If your handler needs async code, mark it with `is_async=True`:

```python
async def weather_herramienta_async(location: str) -> str:
    async with aiohttp.ClientSession() as session:
        ...
    return json.dumps(result)

registry.register(
    name="weather",
    herramientaset="weather",
    schema=WEATHER_SCHEMA,
    handler=lambda args, **kw: weather_herramienta_async(args.get("location", "")),
    check_fn=check_weather_requirements,
    is_async=True,  # registry calls _run_async() automatically
)
```

The registry handles async bridging transparently — you never call `asyncio.run()` yourself.

## Handlers That Need task_id

Herramientas that manage per-session state receive `task_id` via `**kwargs`:

```python
def _handle_weather(args, **kw):
    task_id = kw.get("task_id")
    return weather_herramienta(args.get("location", ""), task_id=task_id)

registry.register(
    name="weather",
    ...
    handler=_handle_weather,
)
```

## Agente-Loop Intercepted Herramientas

Some herramientas (`todo`, `memoria`, `session_search`, `delegate_task`) need access to per-session agente state. These are intercepted by `run_agente.py` before reaching the registry. The registry still holds their schemas, but `dispatch()` returns a fallback error if the intercept is bypassed.

## Optional: Setup Wizard Integration

If your herramienta requires an API key, add it to `hermes_cli/config.py`:

```python
OPTIONAL_ENV_VARS = {
    ...
    "WEATHER_API_KEY": {
        "description": "Weather API key for weather lookup",
        "prompt": "Weather API key",
        "url": "https://weatherapi.com/",
        "herramientas": ["weather"],
        "password": True,
    },
}
```

## Checklist

- [ ] Herramienta file created with handler, schema, check function, and registration
- [ ] Added to appropriate herramientaset in `herramientasets.py`
- [ ] Confirmed this really should be a built-in/core herramienta and not a complemento
- [ ] Handler returns JSON strings, errors returned as `{"error": "..."}`
- [ ] Optional: API key added to `OPTIONAL_ENV_VARS` in `hermes_cli/config.py`
- [ ] Optional: Added to `herramientaset_distributions.py` for batch processing
- [ ] Tested with `hermes chat -q "Use the weather herramienta for London"`

---