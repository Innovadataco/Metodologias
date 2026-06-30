<!-- source: website/docs/guia-desarrollador/herramientas-runtime.md -->
# Herramientas Runtime

# Herramientas Runtime

Hermes herramientas are self-registering functions grouped into herramientasets and executed through a central registry/dispatch system.

Primary files:

- `herramientas/registry.py`
- `modelo_herramientas.py`
- `herramientasets.py`
- `herramientas/terminal_herramienta.py`
- `herramientas/environments/*`

## Herramienta registration modelo

Each herramienta module calls `registry.register(...)` at import time.

`modelo_herramientas.py` is responsible for importing/discovering herramienta modules and building the schema list used by the modelo.

### How `registry.register()` works

Every herramienta file in `herramientas/` calls `registry.register()` at module level to declare itself. The function signature is:

```python
registry.register(
    name="terminal",               # Unique herramienta name (used in API schemas)
    herramientaset="terminal",            # Herramientaset this herramienta belongs to
    schema={...},                  # OpenAI function-calling schema (description, parameters)
    handler=handle_terminal,       # The function that executes when the herramienta is called
    check_fn=check_terminal,       # Optional: returns True/False for availability
    requires_env=["SOME_VAR"],     # Optional: env vars needed (for UI display)
    is_async=False,                # Whether the handler is an async coroutine
    description="Run commands",    # Human-readable description
    emoji="💻",                    # Emoji for spinner/progress display
)
```

Each call creates a `HerramientaEntry` stored in the singleton `HerramientaRegistry._herramientas` dict keyed by herramienta name. If a name collision occurs across herramientasets, a warning is logged and the later registration wins.

### Discovery: `discover_builtin_herramientas()`

When `modelo_herramientas.py` is imported, it calls `discover_builtin_herramientas()` from `herramientas/registry.py`. This function scans every `herramientas/*.py` file using AST parsing to find modules that contain top-level `registry.register()` calls, then imports them:

```python
# herramientas/registry.py (simplified)
def discover_builtin_herramientas(herramientas_dir=None):
    herramientas_path = Path(herramientas_dir) if herramientas_dir else Path(__file__).parent
    for path in sorted(herramientas_path.glob("*.py")):
        if path.name in {"__init__.py", "registry.py", "mcp_herramienta.py"}:
            continue
        if _module_registers_herramientas(path):  # AST check for top-level registry.register()
            importlib.import_module(f"herramientas.{path.stem}")
```

This auto-discovery means new herramienta files are picked up automatically — no manual list to maintain. The AST check only matches top-level `registry.register()` calls (not calls inside functions), so helper modules in `herramientas/` are not imported.

Each import triggers the module's `registry.register()` calls. Errors in optional herramientas (e.g., missing `fal_client` for image generation) are caught and logged — they don't prevent other herramientas from loading.

After core herramienta discovery, MCP herramientas and complemento herramientas are also discovered:

1. **MCP herramientas** — `herramientas.mcp_herramienta.discover_mcp_herramientas()` reads MCP server config and registers herramientas from external servers.
2. **Complemento herramientas** — `hermes_cli.complementos.discover_complementos()` loads user/project/pip complementos that may register additional herramientas.

## Herramienta availability checking (`check_fn`)

Each herramienta can optionally provide a `check_fn` — a callable that returns `True` when the herramienta is available and `False` otherwise. Typical checks include:

- **API key present** — e.g., `lambda: bool(os.environ.get("SERP_API_KEY"))` for web search
- **Service running** — e.g., checking if the Honcho server is configured
- **Binary installed** — e.g., verifying `playwright` is available for browser herramientas

When `registry.get_definitions()` builds the schema list for the modelo, it runs each herramienta's `check_fn()`:

```python
# Simplified from registry.py
if entry.check_fn:
    try:
        available = bool(entry.check_fn())
    except Exception:
        available = False   # Exceptions = unavailable
    if not available:
        continue            # Skip this herramienta entirely
```

Key behaviors:
- Check results are **cached per-call** — if multiple herramientas share the same `check_fn`, it only runs once.
- Exceptions in `check_fn()` are treated as "unavailable" (fail-safe).
- The `is_herramientaset_available()` method checks whether a herramientaset's `check_fn` passes, used for UI display and herramientaset resolution.

## Herramientaset resolution

Herramientasets are named bundles of herramientas. Hermes resolves them through:

- explicit enabled/disabled herramientaset lists
- platform presets (`hermes-cli`, `hermes-telegram`, etc.)
- dynamic MCP herramientasets
- curated special-purpose sets like `hermes-acp`

### How `get_herramienta_definitions()` filters herramientas

The main entry point is `modelo_herramientas.get_herramienta_definitions(enabled_herramientasets, disabled_herramientasets, quiet_mode)`:

1. **If `enabled_herramientasets` is provided** — only herramientas from those herramientasets are included. Each herramientaset name is resolved via `resolve_herramientaset()` which expands composite herramientasets into individual herramienta names.

2. **If `disabled_herramientasets` is provided** — start with ALL herramientasets, then subtract the disabled ones.

3. **If neither** — include all known herramientasets.

4. **Registry filtering** — the resolved herramienta name set is passed to `registry.get_definitions()`, which applies `check_fn` filtering and returns OpenAI-format schemas.

5. **Dynamic schema patching** — after filtering, `execute_code` and `browser_navigate` schemas are dynamically adjusted to only reference herramientas that actually passed filtering (prevents modelo hallucination of unavailable herramientas).

### Legacy herramientaset names

Old herramientaset names with `_herramientas` suffixes (e.g., `web_herramientas`, `terminal_herramientas`) are mapped to their modern herramienta names via `_LEGACY_TOOLSET_MAP` for backward compatibility.

## Dispatch

At runtime, herramientas are dispatched through the central registry, with agente-loop exceptions for some agente-level herramientas such as memoria/todo/session-search handling.

### Dispatch flow: modelo herramienta_call → handler execution

When the modelo returns a `herramienta_call`, the flow is:

```
Modelo response with herramienta_call
    ↓
run_agente.py agente loop
    ↓
modelo_herramientas.handle_function_call(name, args, task_id, user_task)
    ↓
[Agente-loop herramientas?] → handled directly by agente loop (todo, memoria, session_search, delegate_task)
    ↓
[Complemento pre-hook] → invoke_hook("pre_herramienta_call", ...)
    ↓
registry.dispatch(name, args, **kwargs)
    ↓
Look up HerramientaEntry by name
    ↓
[Async handler?] → bridge via _run_async()
[Sync handler?]  → call directly
    ↓
Return result string (or JSON error)
    ↓
[Complemento post-hook] → invoke_hook("post_herramienta_call", ...)
```

### Error wrapping

All herramienta execution is wrapped in error handling at two levels:

1. **`registry.dispatch()`** — catches any exception from the handler and returns `{"error": "Herramienta execution failed: ExceptionType: message"}` as JSON.

2. **`handle_function_call()`** — wraps the entire dispatch in a secondary try/except that returns `{"error": "Error executing herramienta_name: message"}`.

This ensures the modelo always receives a well-formed JSON string, never an unhandled exception.

### Agente-loop herramientas

Four herramientas are intercepted before registry dispatch because they need agente-level state (TodoStore, MemoriaStore, etc.):

- `todo` — planning/task tracking
- `memoria` — persistent memoria writes
- `session_search` — cross-session recall
- `delegate_task` — spawns subagente sessions

These herramientas' schemas are still registered in the registry (for `get_herramienta_definitions`), but their handlers return a stub error if dispatch somehow reaches them directly.

### Async bridging

When a herramienta handler is async, `_run_async()` bridges it to the sync dispatch path:

- **CLI path (no running loop)** — uses a persistent event loop to keep cached async clients alive
- **Puerta de enlace path (running loop)** — spins up a disposable thread with `asyncio.run()`
- **Worker threads (parallel herramientas)** — uses per-thread persistent loops stored in thread-local storage

## The DANGEROUS_PATTERNS approval flow

The terminal herramienta integrates a dangerous-command approval system defined in `herramientas/approval.py`:

1. **Pattern detection** — `DANGEROUS_PATTERNS` is a list of `(regex, description)` tuples covering destructive operations:
   - Recursive deletes (`rm -rf`)
   - Filesystem formatting (`mkfs`, `dd`)
   - SQL destructive operations (`DROP TABLE`, `DELETE FROM` without `WHERE`)
   - System config overwrites (`> /etc/`)
   - Service manipulation (`systemctl stop`)
   - Remote code execution (`curl | sh`)
   - Fork bombs, process kills, etc.

2. **Detection** — before executing any terminal command, `detect_dangerous_command(command)` checks against all patterns.

3. **Approval prompt** — if a match is found:
   - **CLI mode** — an interactive prompt asks the user to approve, deny, or allow permanently
   - **Puerta de enlace mode** — an async approval callback sends the request to the messaging platform
   - **Smart approval** — optionally, an auxiliary LLM can auto-approve low-risk commands that match patterns (e.g., `rm -rf node_modules/` is safe but matches "recursive delete")

4. **Session state** — approvals are tracked per-session. Once you approve "recursive delete" for a session, subsequent `rm -rf` commands don't re-prompt.

5. **Permanent allowlist** — the "allow permanently" option writes the pattern to `config.yaml`'s `command_allowlist`, persisting across sessions.

## Terminal/runtime environments

The terminal system supports multiple backends:

- local
- docker
- ssh
- singularity
- modal
- daytona

It also supports:

- per-task cwd overrides
- background process management
- PTY mode
- approval callbacks for dangerous commands

## Concurrency

Herramienta calls may execute sequentially or concurrently depending on the herramienta mix and interaction requirements.

## Related docs

- [Herramientasets Reference](../reference/herramientasets-reference.md)
- [Built-in Herramientas Reference](../reference/herramientas-reference.md)
- [Agente Loop Internals](./agente-loop.md)
- [ACP Internals](./acp-internals.md)

---