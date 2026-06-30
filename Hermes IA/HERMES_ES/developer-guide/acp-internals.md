<!-- source: website/docs/guia-desarrollador/acp-internals.md -->
# ACP Internals

# ACP Internals

The ACP adapter wraps Hermes' synchronous `AIAgente` in an async JSON-RPC stdio server.

Key implementation files:

- `acp_adapter/entry.py`
- `acp_adapter/server.py`
- `acp_adapter/session.py`
- `acp_adapter/events.py`
- `acp_adapter/permissions.py`
- `acp_adapter/herramientas.py`
- `acp_adapter/auth.py`
- `acp_registry/agente.json`

## Boot flow

```text
hermes acp / hermes-acp / python -m acp_adapter
  -> acp_adapter.entry.main()
  -> parse --version / --check / --setup before server startup
  -> load ~/.hermes/.env
  -> configure stderr logging
  -> construct HermesACPAgente
  -> acp.run_agente(agente, use_unstable_protocol=True)
```

The Zed ACP Registry path launches the same adapter through `uvx --from 'hermes-agente[acp]==<version>' hermes-acp`, pointed at the `hermes-agente` PyPI release.

Stdout is reserved for ACP JSON-RPC transport. Human-readable logs go to stderr.

## Major components

### `HermesACPAgente`

`acp_adapter/server.py` implements the ACP agente protocol.

Responsibilities:

- initialize / authenticate
- new/load/resume/fork/list/cancel session methods
- prompt execution
- session modelo switching
- wiring sync AIAgente callbacks into ACP async notifications

### `SessionManager`

`acp_adapter/session.py` tracks live ACP sessions.

Each session stores:

- `session_id`
- `agente`
- `cwd`
- `modelo`
- `history`
- `cancel_event`

The manager is thread-safe and supports:

- create
- get
- remove
- fork
- list
- cleanup
- cwd updates

### Event bridge

`acp_adapter/events.py` converts AIAgente callbacks into ACP `session_update` events.

Bridged callbacks:

- `herramienta_progress_callback`
- `thinking_callback` (currently set to `None` in the ACP bridge — reasoning is forwarded through `step_callback` instead)
- `step_callback`

Because `AIAgente` runs in a worker thread while ACP I/O lives on the main event loop, the bridge uses:

```python
asyncio.run_coroutine_threadsafe(...)
```

### Permission bridge

`acp_adapter/permissions.py` adapts dangerous terminal approval prompts into ACP permission requests.

Mapping:

- `allow_once` -> Hermes `once`
- `allow_always` -> Hermes `always`
- reject options -> Hermes `deny`

Timeouts and bridge failures deny by default.

### Herramienta rendering helpers

`acp_adapter/herramientas.py` maps Hermes herramientas to ACP herramienta kinds and builds editor-facing content.

Examples:

- `patch` / `write_file` -> file diffs
- `terminal` -> shell command text
- `read_file` / `search_files` -> text previews
- large results -> truncated text blocks for UI safety

## Session lifecycle

```text
new_session(cwd)
  -> create SessionState
  -> create AIAgente(platform="acp", enabled_herramientasets=["hermes-acp"])
  -> bind task_id/session_id to cwd override

prompt(..., session_id)
  -> extract text from ACP content blocks
  -> reset cancel event
  -> install callbacks + approval bridge
  -> run AIAgente in ThreadPoolExecutor
  -> update session history
  -> emit final agente message chunk
```

### Cancelation

`cancel(session_id)`:

- sets the session cancel event
- calls `agente.interrupt()` when available
- causes the prompt response to return `stop_reason="cancelled"`

### Forking

`fork_session()` deep-copies message history into a new live session, preserving conversation state while giving the fork its own session ID and cwd.

## Proveedor/auth behavior

ACP does not implement its own auth store.

Instead it reuses Hermes' runtime resolver:

- `acp_adapter/auth.py`
- `hermes_cli/runtime_proveedor.py`

So ACP advertises and uses the currently configured Hermes proveedor/credentials. It also always advertises a terminal setup auth method (`hermes-setup`, args `--setup`) so first-run registry clients can open Hermes' interactive modelo/proveedor configuración before starting a normal ACP session.

## Working directory binding

ACP sessions carry an editor cwd.

The session manager binds that cwd to the ACP session ID via task-scoped terminal/file overrides, so file and terminal herramientas operate relative to the editor workspace.

## Duplicate same-name herramienta calls

The event bridge tracks herramienta IDs FIFO per herramienta name, not just one ID per name. This is important for:

- parallel same-name calls
- repeated same-name calls in one step

Without FIFO queues, completion events would attach to the wrong herramienta invocation.

## Approval callback restoration

ACP temporarily installs an approval callback on the terminal herramienta during prompt execution, then restores the previous callback afterward. This avoids leaving ACP session-specific approval handlers installed globally forever.

## Current limitations

- ACP sessions are persisted to the shared `~/.hermes/state.db` (SessionDB) and transparently restored across process restarts; they appear in `session_search`
- non-text prompt blocks are currently ignored for request text extraction
- editor-specific UX varies by ACP client implementation

## Related files

- `tests/acp/` — ACP test suite
- `herramientasets.py` — `hermes-acp` herramientaset definition
- `hermes_cli/main.py` — `hermes acp` CLI subcommand
- `pyproject.toml` — `[acp]` optional dependency + `hermes-acp` script

---