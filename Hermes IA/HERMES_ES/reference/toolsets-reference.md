<!-- source: website/docs/reference/herramientasets-reference.md -->
# Herramientasets Reference

# Herramientasets Reference

Herramientasets are named bundles of herramientas that control what the agente can do. They're the primary mechanism for configuring herramienta availability per platform, per session, or per task.

## How Herramientasets Work

Every herramienta belongs to exactly one herramientaset. When you enable a herramientaset, all herramientas in that bundle become available to the agente. Herramientasets come in three kinds:

- **Core** — A single logical group of related herramientas (e.g., `file` bundles `read_file`, `write_file`, `patch`, `search_files`)
- **Composite** — Combines multiple core herramientasets for a common scenario (e.g., `debugging` bundles file, terminal, and web herramientas)
- **Platform** — A complete herramienta configuración for a specific deployment contexto (e.g., `hermes-cli` is the default for interactive CLI sessions)

## Configuring Herramientasets

### Per-session (CLI)

```bash
hermes chat --herramientasets web,file,terminal
hermes chat --herramientasets debugging        # composite — expands to file + terminal + web
hermes chat --herramientasets all              # everything
```

### Per-platform (config.yaml)

```yaml
herramientasets:
  - hermes-cli          # default for CLI
  # - hermes-telegram   # override for Telegram puerta de enlace
```

### Interactive management

```bash
hermes herramientas                            # curses UI to enable/disable per platform
```

Or in-session:

```
/herramientas list
/herramientas disable browser
/herramientas enable homeassistant
```

## Core Herramientasets

| Herramientaset | Herramientas | Purpose |
|---------|-------|---------|
| `browser` | `browser_back`, `browser_cdp`, `browser_click`, `browser_console`, `browser_dialog`, `browser_get_images`, `browser_navigate`, `browser_press`, `browser_scroll`, `browser_snapshot`, `browser_type`, `browser_vision`, `web_search` | Core browser automation. Includes `web_search` as a fallback for quick lookups. `browser_cdp` and `browser_dialog` are gated at runtime — registered only when a CDP endpoint is reachable at session start (via `/browser connect`, `browser.cdp_url` config, Browserbase, or Camofox). `browser_dialog` works together with the `pending_dialogs` and `frame_tree` fields that `browser_snapshot` adds when a CDP supervisor is attached. |
| `clarify` | `clarify` | Ask the user a question when the agente needs clarification. |
| `code_execution` | `execute_code` | Run Python scripts that call Hermes herramientas programmatically. |
| `cronjob` | `cronjob` | Schedule and manage recurring tasks. |
| `debugging` | composite (`file` + `terminal` + `web`) | Debug bundle — file, process/terminal, web extract/search. |
| `delegation` | `delegate_task` | Spawn isolated subagente instances for parallel work. |
| `discord` | `discord` | Core Discord text/embed/DM actions (puerta de enlace-only). Active on the `hermes-discord` herramientaset. |
| `discord_admin` | `discord_admin` | Discord moderation (bans, role changes, channel management). Active on the `hermes-discord` herramientaset; requires the bot to hold the relevant Discord permissions. |
| `feishu_doc` | `feishu_doc_read` | Read Feishu/Lark document content. Used by the Feishu document-comment intelligent-reply handler. |
| `feishu_drive` | `feishu_drive_add_comment`, `feishu_drive_list_comments`, `feishu_drive_list_comment_replies`, `feishu_drive_reply_comment` | Feishu/Lark drive comment operations. Scoped to the comment agente; not exposed on `hermes-cli` or other messaging herramientasets. |
| `file` | `patch`, `read_file`, `search_files`, `write_file` | File reading, writing, searching, and editing. |
| `homeassistant` | `ha_call_service`, `ha_get_state`, `ha_list_entities`, `ha_list_services` | Smart home control via Home Assistant. Only available when `HASS_TOKEN` is set. |
| `computer_use` | `computer_use` | Background macOS desktop control via cua-driver — does not steal cursor/focus. Works with any herramienta-capable modelo. macOS only; requires `cua-driver` on `$PATH`. |
| `contexto_engine` | (varies) | Runtime herramientas exposed by the active contexto-engine complemento (empty until a complemento populates it). |
| `image_gen` | `image_generate` | Text-to-image generation via FAL.ai (with opt-in OpenAI / xAI backends). |
| `video_gen` | `video_generate` | Text-to-video and image-to-video via complemento-registered backends (xAI Grok-Imagine, FAL.ai Veo 3.1 / Pixverse v6 / Kling O3). Pass `image_url` to animate an image; omit it for text-to-video. |
| `kanban` | `kanban_block`, `kanban_comment`, `kanban_complete`, `kanban_create`, `kanban_heartbeat`, `kanban_link`, `kanban_list`, `kanban_show`, `kanban_unblock` | Multi-agente coordination herramientas. Registered for dispatcher-spawned task workers (`HERMES_KANBAN_TASK`) and for perfils that explicitly list the `kanban` herramientaset by name (the `all`/`*` wildcard does **not** enable it). Workers mark tasks done, block, heartbeat, comment, and create/link follow-up tasks; orchestrator perfils additionally get board-routing herramientas like list/unblock. |
| `memoria` | `memoria` | Persistent cross-session memoria management. |
| `messaging` | `send_message` | Send messages to other platforms (Telegram, Discord, etc.) from within a session. |
| `moa` | `mixture_of_agentes` | Multi-modelo consensus via Mixture of Agentes. |
| `safe` | `image_generate`, `vision_analyze`, `web_extract`, `web_search` (via `includes`) | Read-only research + media generation. No file writes, no terminal, no code execution. |
| `search` | `web_search` | Web search only (without extract). |
| `session_search` | `session_search` | Search past conversation sessions. |
| `habilidads` | `habilidad_manage`, `habilidad_view`, `habilidads_list` | Habilidad CRUD and browsing. |
| `spotify` | `spotify_albums`, `spotify_devices`, `spotify_library`, `spotify_playback`, `spotify_playlists`, `spotify_queue`, `spotify_search` | Native Spotify control (playback, queue, search, playlists, albums, library). Registered by the bundled `spotify` complemento. |
| `terminal` | `process`, `terminal` | Shell command execution and background process management. |
| `todo` | `todo` | Task list management within a session. |
| `tts` | `text_to_speech` | Text-to-speech audio generation. |
| `vision` | `vision_analyze` | Image analysis via vision-capable modelos. |
| `video` | `video_analyze` | Video analysis and understanding herramientas (opt-in, not in the default herramientaset — add explicitly via `--herramientasets`). |
| `web` | `web_extract`, `web_search` | Web search and page content extraction. |
| `x_search` | `x_search` | Search X (Twitter) posts and threads via xAI's built-in `x_search` Responses herramienta. Off by default; opt in via `hermes herramientas`. Schema only registered when xAI credentials (SuperGrok OAuth or `XAI_API_KEY`) are configured. |
| `yuanbao` | `yb_query_group_info`, `yb_query_group_members`, `yb_search_sticker`, `yb_send_dm`, `yb_send_sticker` | Yuanbao DM/group actions and sticker search. Registered only on `hermes-yuanbao`. |

## Platform Herramientasets

Platform herramientasets define the complete herramienta configuración for a deployment target. Most messaging platforms use the same set as `hermes-cli`:

| Herramientaset | Differences from `hermes-cli` |
|---------|-------------------------------|
| `hermes-cli` | Full herramientaset — the default for interactive CLI sessions. Includes file, terminal, web, browser, memoria, habilidads, vision, image_gen, todo, tts, delegation, code_execution, cronjob, session_search, clarify, and `safe` (read-only) bundles plus the standard messaging herramientas. |
| `hermes-acp` | Drops `clarify`, `cronjob`, `image_generate`, `send_message`, `text_to_speech`, and all four Home Assistant herramientas. Focused on coding tasks in IDE contexto. |
| `hermes-api-server` | Drops `clarify`, `send_message`, and `text_to_speech`. Keeps everything else — suitable for programmatic access where user interaction isn't possible. |
| `hermes-cron` | Same as `hermes-cli`. |
| `hermes-telegram` | Same as `hermes-cli`. |
| `hermes-discord` | Adds `discord` and `discord_admin` on top of `hermes-cli`. |
| `hermes-slack` | Same as `hermes-cli`. |
| `hermes-whatsapp` | Same as `hermes-cli`. |
| `hermes-signal` | Same as `hermes-cli`. |
| `hermes-matrix` | Same as `hermes-cli`. |
| `hermes-mattermost` | Same as `hermes-cli`. |
| `hermes-email` | Same as `hermes-cli`. |
| `hermes-sms` | Same as `hermes-cli`. |
| `hermes-bluebubbles` | Same as `hermes-cli`. |
| `hermes-dingtalk` | Same as `hermes-cli`. |
| `hermes-feishu` | Adds the five `feishu_doc_*` / `feishu_drive_*` herramientas (only used by the document-comment handler, not the regular chat adapter). |
| `hermes-qqbot` | Same as `hermes-cli`. |
| `hermes-wecom` | Same as `hermes-cli`. |
| `hermes-wecom-callback` | Same as `hermes-cli`. |
| `hermes-weixin` | Same as `hermes-cli`. |
| `hermes-yuanbao` | Adds the five `yb_*` herramientas (DM/group/sticker) on top of `hermes-cli`. |
| `hermes-homeassistant` | Same as `hermes-cli` (the Home Assistant herramientas are already present by default and activate when `HASS_TOKEN` is set). |
| `hermes-webhook` | Same as `hermes-cli`. |
| `hermes-puerta de enlace` | Internal puerta de enlace orchestrator herramientaset — union of every `hermes-<platform>` herramientaset; used when the puerta de enlace needs to accept any message source. |

## Dynamic Herramientasets

### MCP server herramientasets

Each configured MCP server generates a `mcp-<server>` herramientaset at runtime. For example, if you configure a `github` MCP server, a `mcp-github` herramientaset is created containing all herramientas that server exposes.

```yaml
# config.yaml
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelocontextoprotocol/server-github"]
```

This creates a `mcp-github` herramientaset you can reference in `--herramientasets` or platform configs.

### Complemento herramientasets

Complementos can register their own herramientasets via `ctx.register_herramienta()` during complemento initialization. These appear alongside built-in herramientasets and can be enabled/disabled the same way.

### Custom herramientasets

Define custom herramientasets in `config.yaml` to create project-specific bundles:

```yaml
herramientasets:
  - hermes-cli
custom_herramientasets:
  data-science:
    - file
    - terminal
    - code_execution
    - web
    - vision
```

### Wildcards

- `all` or `*` — expands to every registered herramientaset (built-in + dynamic + complemento)

A handful of herramientas have an additional availability check on top of herramientaset membership and are **not** turned on by `all`/`*` alone:

- **Capability-gated** herramientas (browser, `computer_use`, `code_execution`, Feishu, Home Assistant, cronjob) appear only when their backend/credential prerequisite is configured.
- **Workflow-gated** herramientas — the `kanban` herramientaset — are deliberately opt-in. `all`/`*` does **not** enable kanban; you must list `kanban` explicitly (or be a dispatcher-spawned worker with `HERMES_KANBAN_TASK` set). Kanban herramientas mutate shared board state, so they stay off by default even under `all`.

## Relationship to `hermes herramientas`

The `hermes herramientas` command provides a curses-based UI for toggling individual herramientas on or off per platform. This operates at the herramienta level (finer than herramientasets) and persists to `config.yaml`. Disabled herramientas are filtered out even if their herramientaset is enabled.

See also: [Herramientas Reference](./herramientas-reference.md) for the complete list of individual herramientas and their parameters.

---