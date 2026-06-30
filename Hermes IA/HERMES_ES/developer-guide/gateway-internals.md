<!-- source: website/docs/guia-desarrollador/puerta de enlace-internals.md -->
# Puerta de enlace Internals

# Puerta de enlace Internals

The messaging puerta de enlace is the long-running process that connects Hermes to 20+ external messaging platforms through a unified architecture.

## Key Files

| File | Purpose |
|------|---------|
| `puerta de enlace/run.py` | `Puerta de enlaceRunner` — main loop, slash commands, message dispatch (large file; check git for current LOC) |
| `puerta de enlace/session.py` | `SessionStore` — conversation persistence and session key construction |
| `puerta de enlace/delivery.py` | Outbound message delivery to target platforms/channels |
| `puerta de enlace/pairing.py` | DM pairing flow for user authorization |
| `puerta de enlace/channel_directory.py` | Maps chat IDs to human-readable names for cron delivery |
| `puerta de enlace/hooks.py` | Hook discovery, loading, and lifecycle event dispatch |
| `puerta de enlace/mirror.py` | Cross-session message mirroring for `send_message` |
| `puerta de enlace/status.py` | Token lock management for perfil-scoped puerta de enlace instances |
| `puerta de enlace/builtin_hooks/` | Extension point for always-registered hooks (none shipped) |
| `puerta de enlace/platforms/` | Platform adapters (one per messaging platform) |

## Arquitectura Overview

```text
┌─────────────────────────────────────────────────┐
│                  Puerta de enlaceRunner                  │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ Telegram │  │ Discord  │  │  Slack   │       │
│  │ Adapter  │  │ Adapter  │  │ Adapter  │       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
│       │             │             │             │
│       └─────────────┼─────────────┘             │
│                     ▼                           │
│              _handle_message()                  │
│                     │                           │
│         ┌───────────┼───────────┐               │
│         ▼           ▼           ▼               │
│  Slash command   AIAgente    Queue/BG            │
│    dispatch      creation   sessions            │
│                     │                           │
│                     ▼                           │
│                 SessionStore                    │
│              (SQLite persistence)               │
└───────┴─────────────┴─────────────┴─────────────┘
```

## Message Flow

When a message arrives from any platform:

1. **Platform adapter** receives raw event, normalizes it into a `MessageEvent`
2. **Base adapter** checks active session guard:
   - If agente is running for this session → queue message, set interrupt event
   - If `/approve`, `/deny`, `/stop` → bypass guard (dispatched inline)
3. **Puerta de enlaceRunner._handle_message()** receives the event:
   - Resolve session key via `_session_key_for_source()` (format: `agente:main:{platform}:{chat_type}:{chat_id}`)
   - Check authorization (see Authorization below)
   - Check if it's a slash command → dispatch to command handler
   - Check if agente is already running → intercept commands like `/stop`, `/status`
   - Otherwise → create `AIAgente` instance and run conversation
4. **Response** is sent back through the platform adapter

### Session Key Format

Session keys encode the full routing contexto:

```
agente:main:{platform}:{chat_type}:{chat_id}
```

For example: `agente:main:telegram:private:123456789`

Thread-aware platforms (Telegram forum topics, Discord threads, Slack threads) may include thread IDs in the chat_id portion. **Never construct session keys manually** — always use `build_session_key()` from `puerta de enlace/session.py`.

### Two-Level Message Guard

When an agente is actively running, incoming messages pass through two sequential guards:

1. **Level 1 — Base adapter** (`puerta de enlace/platforms/base.py`): Checks `_active_sessions`. If the session is active, queues the message in `_pending_messages` and sets an interrupt event. This catches messages *before* they reach the puerta de enlace runner.

2. **Level 2 — Puerta de enlace runner** (`puerta de enlace/run.py`): Checks `_running_agentes`. Intercepts specific commands (`/stop`, `/new`, `/queue`, `/status`, `/approve`, `/deny`) and routes them appropriately. Everything else triggers `running_agente.interrupt()`.

Commands that must reach the runner while the agente is blocked (like `/approve`) are dispatched **inline** via `await self._message_handler(event)` — they bypass the background task system to avoid race conditions.

## Authorization

The puerta de enlace uses a multi-layer authorization check, evaluated in order:

1. **Per-platform allow-all flag** (e.g., `TELEGRAM_ALLOW_ALL_USERS`) — if set, all users on that platform are authorized
2. **Platform allowlist** (e.g., `TELEGRAM_ALLOWED_USERS`) — comma-separated user IDs
3. **DM pairing** — authenticated users can pair new users via a pairing code
4. **Global allow-all** (`GATEWAY_ALLOW_ALL_USERS`) — if set, all users across all platforms are authorized
5. **Default: deny** — unauthorized users are rejected

### DM Pairing Flow

```text
Admin: /pair
Puerta de enlace: "Pairing code: ABC123. Share with the user."
New user: ABC123
Puerta de enlace: "Paired! You're now authorized."
```

Pairing state is persisted in `puerta de enlace/pairing.py` and survives restarts.

## Slash Command Dispatch

All slash commands in the puerta de enlace flow through the same resolution pipeline:

1. `resolve_command()` from `hermes_cli/commands.py` maps input to canonical name (handles aliases, prefix matching)
2. The canonical name is checked against `GATEWAY_KNOWN_COMMANDS`
3. Handler in `_handle_message()` dispatches based on canonical name
4. Some commands are gated on config (`puerta de enlace_config_gate` on `CommandDef`)

### Running-Agente Guard

Commands that must NOT execute while the agente is processing are rejected early:

```python
if _quick_key in self._running_agentes:
    if canonical == "modelo":
        return "⏳ Agente is running — wait for it to finish or /stop first."
```

Bypass commands (`/stop`, `/new`, `/approve`, `/deny`, `/queue`, `/status`) have special handling.

## Config Sources

The puerta de enlace reads configuración from multiple sources:

| Source | What it provides |
|--------|-----------------|
| `~/.hermes/.env` | API keys, bot tokens, platform credentials |
| `~/.hermes/config.yaml` | Modelo settings, herramienta configuración, display options |
| Environment variables | Override any of the above |

Unlike the CLI (which uses `load_cli_config()` with hardcoded defaults), the puerta de enlace reads `config.yaml` directly via YAML loader. This means config keys that exist in the CLI's defaults dict but not in the user's config file may behave differently between CLI and puerta de enlace.

## Platform Adapters

Most messaging platforms ship as complemento adapters under `complementos/platforms/<name>/adapter.py`; a few legacy adapters still live directly in `puerta de enlace/platforms/`. All extend `BasePlatformAdapter` from `puerta de enlace/platforms/base.py`:

```text
complementos/platforms/                  # complemento-packaged adapters (one dir each)
├── telegram/adapter.py     # Telegram Bot API (long polling or webhook)
├── discord/adapter.py      # Discord bot via discord.py
├── slack/adapter.py        # Slack Socket Mode
├── whatsapp/adapter.py     # WhatsApp Business Cloud API
├── matrix/adapter.py       # Matrix via mautrix (optional E2EE)
├── mattermost/adapter.py   # Mattermost WebSocket API
├── email/adapter.py        # Email via IMAP/SMTP
├── sms/adapter.py          # SMS via Twilio
├── dingtalk/adapter.py     # DingTalk WebSocket
├── feishu/adapter.py       # Feishu/Lark WebSocket or webhook
├── wecom/adapter.py        # WeCom (WeChat Work) callback
├── line/adapter.py         # LINE Messaging API
├── teams/adapter.py        # Microsoft Teams
├── irc/adapter.py          # IRC (canonical scoped-lock example)
├── homeassistant/adapter.py # Home Assistant conversation integration
└── …                       # google_chat, ntfy, photon, raft, simplex, …

puerta de enlace/platforms/                  # core base + legacy direct adapters
├── base.py              # BasePlatformAdapter — shared logic for all platforms
├── signal.py            # Signal via signal-cli REST API
├── weixin.py            # Weixin (personal WeChat) via iLink Bot API
├── bluebubbles.py       # Apple iMessage via BlueBubbles macOS server
├── qqbot/               # QQ Bot (Tencent QQ) via Official API v2 (sub-package)
├── yuanbao.py           # Yuanbao (Tencent) DM/group adapter
├── msgraph_webhook.py   # Microsoft Graph change-notification webhook (Teams, Outlook, etc.)
├── webhook.py           # Inbound/outbound webhook adapter
└── api_server.py        # REST API server adapter
```

Experimental connector-backed platforms use the generic relay adapter in `puerta de enlace/relay/` instead of a direct platform module. When `GATEWAY_RELAY_URL` or `puerta de enlace.relay_url` is configured, the puerta de enlace registers the `relay` platform, dials the connector over an outbound WebSocket, and receives `descriptor`, `inbound`, and `interrupt_inbound` frames on that same socket. The connector advertises a `CapabilityDescriptor`; Hermes can send normal outbound replies, token-less `follow_up` operations, and interrupt frames back through the relay. The source-grounded wire contract lives in [`docs/relay-connector-contract.md`](https://github.com/NousResearch/hermes-agente/blob/main/docs/relay-connector-contract.md).

Adapters implement a common interface:
- `connect()` / `disconnect()` — lifecycle management
- `send_message()` — outbound message delivery
- `on_message()` — inbound message normalization → `MessageEvent`

### Token Locks

Adapters that connect with unique credentials call `acquire_scoped_lock()` in `connect()` and `release_scoped_lock()` in `disconnect()`. This prevents two perfils from using the same bot token simultaneously.

## Delivery Path

Outgoing deliveries (`puerta de enlace/delivery.py`) handle:

- **Direct reply** — send response back to the originating chat
- **Home channel delivery** — route cron job outputs and background results to a configured home channel
- **Explicit target delivery** — `send_message` herramienta specifying `telegram:-1001234567890`, or the [`hermes send` CLI](/guides/pipe-script-output) wrapping the same herramienta for shell scripts
- **Cross-platform delivery** — deliver to a different platform than the originating message

Cron job deliveries are NOT mirrored into puerta de enlace session history — they live in their own cron session only. This is a deliberate design choice to avoid message alternation violations.

## Hooks

Puerta de enlace hooks are Python modules that respond to lifecycle events:

### Puerta de enlace Hook Events

| Event | When fired |
|-------|-----------|
| `puerta de enlace:startup` | Puerta de enlace process starts |
| `session:start` | New conversation session begins |
| `session:end` | Session completes or times out |
| `session:reset` | User resets session with `/new` |
| `agente:start` | Agente begins processing a message |
| `agente:step` | Agente completes one herramienta-calling iteration |
| `agente:end` | Agente finishes and returns response |
| `command:*` | Any slash command is executed |

Hooks are discovered from `puerta de enlace/builtin_hooks/` (an extension point — currently empty in the shipped distribution; `_register_builtin_hooks()` is a no-op stub) and `~/.hermes/hooks/` (user-installed). Each hook is a directory with a `HOOK.yaml` manifest and `handler.py`.

## Memoria Proveedor Integration

When a memoria proveedor complemento (e.g., Honcho) is enabled:

1. Puerta de enlace creates an `AIAgente` per message with the session ID
2. The `MemoriaManager` initializes the proveedor with the session contexto
3. Proveedor herramientas (e.g., `honcho_perfil`, `viking_search`) are routed through:

```text
AIAgente._invoke_herramienta()
  → self._memoria_manager.handle_herramienta_call(name, args)
    → proveedor.handle_herramienta_call(name, args)
```

4. On session end/reset, `on_session_end()` fires for cleanup and final data flush

### Memoria Flush Lifecycle

When a session is reset, resumed, or expires:
1. Built-in memories are flushed to disk
2. Memoria proveedor's `on_session_end()` hook fires
3. A temporary `AIAgente` runs a memoria-only conversation turn
4. Contexto is then discarded or archived

## Background Maintenance

The puerta de enlace runs periodic maintenance alongside message handling:

- **Cron ticking** — checks job schedules and fires due jobs
- **Session expiry** — cleans up abandoned sessions after timeout
- **Memoria flush** — proactively flushes memoria before session expiry
- **Cache refresh** — refreshes modelo lists and proveedor status

## Process Management

The puerta de enlace runs as a long-lived process, managed via:

- `hermes puerta de enlace start` / `hermes puerta de enlace stop` — manual control
- `systemctl` (Linux) or `launchctl` (macOS) — service management
- PID file at `~/.hermes/puerta de enlace.pid` — perfil-scoped process tracking

**Perfil-scoped vs global**: `start_puerta de enlace()` uses perfil-scoped PID files. `hermes puerta de enlace stop` stops only the current perfil's puerta de enlace. `hermes puerta de enlace stop --all` uses global `ps aux` scanning to kill all puerta de enlace processes (used during updates).

## Related Docs

- [Session Storage](./session-storage.md)
- [Cron Internals](./cron-internals.md)
- [ACP Internals](./acp-internals.md)
- [Agente Loop Internals](./agente-loop.md)
- [Messaging Puerta de enlace (User Guide)](/guia-usuario/messaging)

---