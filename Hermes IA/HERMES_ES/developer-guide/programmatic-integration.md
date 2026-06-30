<!-- source: website/docs/guia-desarrollador/programmatic-integration.md -->
# Programmatic Integration

# Programmatic Integration

Hermes ships three protocols for driving the agente from external programs — IDE complementos, custom UIs, CI pipelines, embedded sub-agentes. Pick the one that matches your transport and consumer.

| Protocol | Transport | Best for | Defined by |
|----------|-----------|----------|------------|
| **ACP** | JSON-RPC over stdio | IDE clients (VS Code, Zed, JetBrains) that already speak the [Agente Client Protocol](https://github.com/zed-industries/agente-client-protocol) | `acp_adapter/` |
| **TUI puerta de enlace** | JSON-RPC over stdio (or WebSocket) | Custom hosts that want fine-grained control of sessions, slash commands, approvals, and streaming events | `tui_puerta de enlace/server.py` |
| **API server** | HTTP + Server-Sent Events | OpenAI-compatible frontends (Open WebUI, LobeChat, LibreChat…) and language-agnostic web clients | `puerta de enlace/platforms/api_server.py` |

All three drive the same `AIAgente` core. They differ only in wire format and which set of features they expose.

---

## ACP (Agente Client Protocol)

`hermes acp` starts a stdio JSON-RPC server speaking ACP. Used in production by VS Code (Zed Industries' ACP extension), Zed, and any JetBrains IDE with an ACP complemento.

Capabilities exposed: session creation, prompt submission, streaming agente message chunks, herramienta-call events, permission requests, session fork, cancel, and autenticación. Herramienta output is rendered into ACP `Diff`/`HerramientaCall` content blocks the IDE understands.

Full lifecycle, event bridge, and approval flow: [ACP Internals](./acp-internals).

```bash
hermes acp                  # serve ACP on stdio
hermes acp --bootstrap      # print install snippet for an ACP-capable IDE
```

---

## TUI Puerta de enlace JSON-RPC

`tui_puerta de enlace/server.py` is the protocol the Ink TUI (`hermes --tui`) and the embedded panel PTY bridge talk to. Any external host can speak the same protocol over stdio (or WebSocket via `tui_puerta de enlace/ws.py`).

### Method catalog (selected)

```
prompt.submit           prompt.background       session.steer
session.create          session.list            session.active_list
session.activate        session.close           session.interrupt
session.history         session.compress        session.branch
session.title           session.usage           session.status
clarify.respond         sudo.respond            secret.respond
approval.respond        config.set / config.get commands.catalog
command.resolve         command.dispatch        cli.exec
reload.mcp              reload.env              process.stop
delegation.status       subagente.interrupt      spawn_tree.save / list / load
terminal.resize         clipboard.paste         image.attach
```

`session.active_list`, `session.activate`, and `session.close` are the process-local live-session controls used by the TUI session switcher. Use `session.list` / `/resume` for saved transcript discovery; use the active-session methods only for sessions that are currently open in the TUI puerta de enlace process.

### Events streamed back

`message.delta`, `message.complete`, `herramienta.start`, `herramienta.progress`, `herramienta.complete`, `approval.request`, `clarify.request`, `sudo.request`, `secret.request`, `puerta de enlace.ready`, plus session lifecycle and error events.

### Pi-style RPC mapping

Every command in the Pi-mono RPC spec ([issue #360](https://github.com/NousResearch/hermes-agente/issues/360)) has a TUI-puerta de enlace equivalent:

| Pi command | Hermes equivalent |
|------------|-------------------|
| `prompt` | `prompt.submit` (or ACP `session/prompt`) |
| `steer` | `session.steer` |
| `follow_up` | `prompt.submit` queued after current turn |
| `abort` | `session.interrupt` |
| `set_modelo` | `command.dispatch` for `/modelo <proveedor:modelo>` (mid-session, persistent) |
| `compact` | `session.compress` |
| `get_state` | `session.status` |
| `get_messages` | `session.history` |
| `switch_session` | `session.resume` |
| `fork` | `session.branch` |
| `ui_request` / `ui_response` | `clarify.respond` / `sudo.respond` / `secret.respond` / `approval.respond` |

---

## OpenAI-Compatible API Server

`puerta de enlace/platforms/api_server.py` exposes hermes over HTTP for any client that already speaks the OpenAI format. Useful when you want a web frontend, a curl-driven CI runner, or a non-Python consumer.

Endpoints:

```
POST /v1/chat/completions        OpenAI Chat Completions (streaming via SSE)
POST /v1/responses               OpenAI Responses API (stateful)
POST /v1/runs                    Start a run, returns run_id (202)
GET  /v1/runs/{id}               Run status
GET  /v1/runs/{id}/events        SSE stream of lifecycle events
POST /v1/runs/{id}/approval      Resolve a pending approval
POST /v1/runs/{id}/stop          Interrupt the run
GET  /v1/capabilities            Machine-readable feature flags
GET  /v1/modelos                  Lists hermes-agente
GET  /health, /health/detailed
```

Setup, headers (`X-Hermes-Session-Id`, `X-Hermes-Session-Key`), and frontend wiring: [API Server](../guia-usuario/features/api-server).

---

## Which one should I use?

- **You're writing an IDE complemento and the IDE already speaks ACP** → ACP. Zero protocol work on the IDE side.
- **You're writing a custom desktop / web / TUI host and want every Hermes feature** (slash commands, approvals, clarify, multi-agente, session branching) → TUI puerta de enlace JSON-RPC.
- **You want any OpenAI-compatible frontend, a language-agnostic HTTP client, or curl-driven automation** → API server.
- **You want a Python in-process embed without a subprocess** → import `run_agente.AIAgente` directly. See [Agente Loop](./agente-loop).

---

## Modelo hot-swapping

Mid-session modelo switching works on every surface — it's the `/modelo` slash command under the hood.

- **CLI / TUI:** `/modelo claude-sonnet-4` or `/modelo openrouter:anthropic/claude-sonnet-4.6`
- **TUI puerta de enlace RPC:** `command.dispatch` with `{"command": "/modelo claude-sonnet-4"}`
- **ACP:** the IDE sends the slash command as a prompt; the agente dispatches it
- **API server:** include a `modelo` field in the request body or set `X-Hermes-Modelo`

Proveedor-aware resolution (the same modelo name picks the right format for whatever proveedor you're on) is built in. See `hermes_cli/modelo_switch.py`.

---

## A note on `--mode rpc`

Hermes does not have a `--mode rpc` flag. The three protocols above already cover the use cases — ACP for IDE-protocol clients, the TUI puerta de enlace for stdio JSON-RPC hosts, and the API server for HTTP. If you find a real gap that none of them fill, open an issue with the concrete consumer you're building.

---