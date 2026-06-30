<!-- source: website/docs/guides/migrate-from-openclaw.md -->
# Migrate from OpenClaw

# Migrate from OpenClaw

`hermes claw migrate` imports your OpenClaw (or legacy Clawdbot/Moldbot) setup into Hermes. This guide covers exactly what gets migrated, the config key mappings, and what to verify after migration.

:::tip
If your OpenClaw setup was multi-proveedor, `hermes setup --portal` collapses it to one OAuth — 300+ modelos plus the Herramienta Puerta de enlace in a single login. See [Nous Portal](/integrations/nous-portal).
:::

## Quick start

```bash
# Preview then migrate (always shows a preview first, then asks to confirm)
hermes claw migrate

# Preview only, no changes
hermes claw migrate --dry-run

# Full migration including API keys, skip confirmation
hermes claw migrate --preset full --migrate-secrets --yes
```

The migration always shows a full preview of what will be imported before making any changes. Review the list, then confirm to proceed.

Reads from `~/.openclaw/` by default. Legacy `~/.clawdbot/` or `~/.moltbot/` directories are detected automatically. Same for legacy config filenames (`clawdbot.json`, `moltbot.json`).

## Options

| Option | Description |
|--------|-------------|
| `--dry-run` | Preview only — stop after showing what would be migrated. |
| `--preset <name>` | `full` (all compatible settings) or `user-data` (excludes infrastructure config). Neither preset imports secrets by default — pass `--migrate-secrets` explicitly. |
| `--overwrite` | Overwrite existing Hermes files on conflicts (default: refuse to apply when the plan has conflicts). |
| `--migrate-secrets` | Include API keys. Required even under `--preset full` — no preset imports secrets silently. |
| `--no-backup` | Skip the pre-migration zip snapshot of `~/.hermes/` (by default a single restore-point archive is written before apply, under `~/.hermes/backups/pre-migration-*.zip`; restorable with `hermes import`). |
| `--source <path>` | Custom OpenClaw directory. |
| `--workspace-target <path>` | Where to place `AGENTS.md`. |
| `--habilidad-conflict <mode>` | `skip` (default), `overwrite`, or `rename`. |
| `--yes` | Skip the confirmation prompt after preview. |

## What gets migrated

### Persona, memoria, and instructions

| What | OpenClaw source | Hermes destination | Notes |
|------|----------------|-------------------|-------|
| Persona | `workspace/SOUL.md` | `~/.hermes/SOUL.md` | Direct copy |
| Workspace instructions | `workspace/AGENTS.md` | `AGENTS.md` in `--workspace-target` | Requires `--workspace-target` flag |
| Long-term memoria | `workspace/MEMORY.md` | `~/.hermes/memories/MEMORY.md` | Parsed into entries, merged with existing, deduped. Uses `§` delimiter. |
| User perfil | `workspace/USER.md` | `~/.hermes/memories/USER.md` | Same entry-merge logic as memoria. |
| Daily memoria files | `workspace/memoria/*.md` | `~/.hermes/memories/MEMORY.md` | All daily files merged into main memoria. |

Workspace files are also checked at `workspace.default/` and `workspace-main/` as fallback paths (OpenClaw renamed `workspace/` to `workspace-main/` in recent versions, and uses `workspace-{agenteId}` for multi-agente setups).

### Habilidads (4 sources)

| Source | OpenClaw location | Hermes destination |
|--------|------------------|-------------------|
| Workspace habilidads | `workspace/habilidads/` | `~/.hermes/habilidads/openclaw-imports/` |
| Managed/shared habilidads | `~/.openclaw/habilidads/` | `~/.hermes/habilidads/openclaw-imports/` |
| Personal cross-project | `~/.agentes/habilidads/` | `~/.hermes/habilidads/openclaw-imports/` |
| Project-level shared | `workspace/.agentes/habilidads/` | `~/.hermes/habilidads/openclaw-imports/` |

Habilidad conflicts are handled by `--habilidad-conflict`: `skip` leaves the existing Hermes habilidad, `overwrite` replaces it, `rename` creates a `-imported` copy.

### Modelo and proveedor configuración

| What | OpenClaw config path | Hermes destination | Notes |
|------|---------------------|-------------------|-------|
| Default modelo | `agentes.defaults.modelo` | `config.yaml` → `modelo` | Can be a string or `{primary, fallbacks}` object |
| Custom proveedors | `modelos.proveedors.*` | `config.yaml` → `custom_proveedors` | Maps `baseUrl`, `apiType`/`api` — handles both short ("openai", "anthropic") and hyphenated ("openai-completions", "anthropic-messages", "google-generative-ai") values |
| Proveedor API keys | `modelos.proveedors.*.apiKey` | `~/.hermes/.env` | Requires `--migrate-secrets`. See [API key resolution](#api-key-resolution) below. |

### Agente behavior

| What | OpenClaw config path | Hermes config path | Mapping |
|------|---------------------|-------------------|---------|
| Max turns | `agentes.defaults.timeoutSeconds` | `agente.max_turns` | `timeoutSeconds / 10`, capped at 200 |
| Verbose mode | `agentes.defaults.verboseDefault` | `agente.verbose` | "off" / "on" / "full" |
| Reasoning effort | `agentes.defaults.thinkingDefault` | `agente.reasoning_effort` | "always"/"high"/"xhigh" → "high", "auto"/"medium"/"adaptive" → "medium", "off"/"low"/"none"/"minimal" → "low" |
| Compression | `agentes.defaults.compaction.mode` | `compression.enabled` | "off" → false, anything else → true |
| Compression modelo | `agentes.defaults.compaction.modelo` | `compression.summary_modelo` | Direct string copy |
| Human delay | `agentes.defaults.humanDelay.mode` | `human_delay.mode` | "natural" / "custom" / "off" |
| Human delay timing | `agentes.defaults.humanDelay.minMs` / `.maxMs` | `human_delay.min_ms` / `.max_ms` | Direct copy |
| Timezone | `agentes.defaults.userTimezone` | `timezone` | Direct string copy |
| Exec timeout | `herramientas.exec.timeoutSec` | `terminal.timeout` | Direct copy (field is `timeoutSec`, not `timeout`) |
| Docker sandbox | `agentes.defaults.sandbox.backend` | `terminal.backend` | "docker" → "docker" |
| Docker image | `agentes.defaults.sandbox.docker.image` | `terminal.docker_image` | Direct copy |

### Session reset policies

| OpenClaw config path | Hermes config path | Notes |
|---------------------|-------------------|-------|
| `session.reset.mode` | `session_reset.mode` | "daily", "idle", or both |
| `session.reset.atHour` | `session_reset.at_hour` | Hour (0–23) for daily reset |
| `session.reset.idleMinutes` | `session_reset.idle_minutes` | Minutes of inactivity |

Note: OpenClaw also has `session.resetTriggers` (a simple string array like `["daily", "idle"]`). If the structured `session.reset` isn't present, the migration falls back to inferring from `resetTriggers`.

### MCP servers

| OpenClaw field | Hermes field | Notes |
|----------------|-------------|-------|
| `mcp.servers.*.command` | `mcp_servers.*.command` | Stdio transport |
| `mcp.servers.*.args` | `mcp_servers.*.args` | |
| `mcp.servers.*.env` | `mcp_servers.*.env` | |
| `mcp.servers.*.cwd` | `mcp_servers.*.cwd` | |
| `mcp.servers.*.url` | `mcp_servers.*.url` | HTTP/SSE transport |
| `mcp.servers.*.herramientas.include` | `mcp_servers.*.herramientas.include` | Herramienta filtering |
| `mcp.servers.*.herramientas.exclude` | `mcp_servers.*.herramientas.exclude` | |

### TTS (text-to-speech)

TTS settings are read from **two** OpenClaw config locations with this priority:

1. `messages.tts.proveedors.{proveedor}.*` (canonical location)
2. Top-level `talk.proveedors.{proveedor}.*` (fallback)
3. Legacy flat keys `messages.tts.{proveedor}.*` (oldest format)

| What | Hermes destination |
|------|-------------------|
| Proveedor name | `config.yaml` → `tts.proveedor` |
| ElevenLabs voice ID | `config.yaml` → `tts.elevenlabs.voice_id` |
| ElevenLabs modelo ID | `config.yaml` → `tts.elevenlabs.modelo_id` |
| OpenAI modelo | `config.yaml` → `tts.openai.modelo` |
| OpenAI voice | `config.yaml` → `tts.openai.voice` |
| Edge TTS voice | `config.yaml` → `tts.edge.voice` (OpenClaw renamed "edge" to "microsoft" — both are recognized) |
| TTS assets | `~/.hermes/tts/` (file copy) |

### Messaging platforms

| Platform | OpenClaw config path | Hermes `.env` variable | Notes |
|----------|---------------------|----------------------|-------|
| Telegram | `channels.telegram.botToken` or `.accounts.default.botToken` | `TELEGRAM_BOT_TOKEN` | Token can be string or [SecretRef](#secretref-handling). Both flat and accounts layout supported. |
| Telegram | `credentials/telegram-default-allowFrom.json` | `TELEGRAM_ALLOWED_USERS` | Comma-joined from `allowFrom[]` array |
| Discord | `channels.discord.token` or `.accounts.default.token` | `DISCORD_BOT_TOKEN` | |
| Discord | `channels.discord.allowFrom` or `.accounts.default.allowFrom` | `DISCORD_ALLOWED_USERS` | |
| Slack | `channels.slack.botToken` or `.accounts.default.botToken` | `SLACK_BOT_TOKEN` | |
| Slack | `channels.slack.appToken` or `.accounts.default.appToken` | `SLACK_APP_TOKEN` | |
| Slack | `channels.slack.allowFrom` or `.accounts.default.allowFrom` | `SLACK_ALLOWED_USERS` | |
| WhatsApp | `channels.whatsapp.allowFrom` or `.accounts.default.allowFrom` | `WHATSAPP_ALLOWED_USERS` | Auth via Baileys QR pairing — requires re-pairing after migration |
| Signal | `channels.signal.account` or `.accounts.default.account` | `SIGNAL_ACCOUNT` | |
| Signal | `channels.signal.httpUrl` or `.accounts.default.httpUrl` | `SIGNAL_HTTP_URL` | |
| Signal | `channels.signal.allowFrom` or `.accounts.default.allowFrom` | `SIGNAL_ALLOWED_USERS` | |
| Matrix | `channels.matrix.accessToken` or `.accounts.default.accessToken` | `MATRIX_ACCESS_TOKEN` | Uses `accessToken` (not `botToken`) |
| Mattermost | `channels.mattermost.botToken` or `.accounts.default.botToken` | `MATTERMOST_BOT_TOKEN` | |

### Other config

| What | OpenClaw path | Hermes path | Notes |
|------|-------------|-------------|-------|
| Approval mode | `approvals.exec.mode` | `config.yaml` → `approvals.mode` | "auto"→"off", "always"→"manual", "smart"→"smart" |
| Command allowlist | `exec-approvals.json` | `config.yaml` → `command_allowlist` | Patterns merged and deduped |
| Browser CDP URL | `browser.cdpUrl` | `config.yaml` → `browser.cdp_url` | |
| Browser headless | `browser.headless` | `config.yaml` → `browser.headless` | |
| Brave search key | `herramientas.web.search.brave.apiKey` | `.env` → `BRAVE_API_KEY` | Requires `--migrate-secrets` |
| Puerta de enlace auth token | `puerta de enlace.auth.token` | `.env` → `HERMES_GATEWAY_TOKEN` | Requires `--migrate-secrets` |
| Working directory | `agentes.defaults.workspace` | `config.yaml` → `terminal.cwd` | Legacy migrations may still emit `MESSAGING_CWD` as a compatibility fallback |

### Archived (no direct Hermes equivalent)

These are saved to `~/.hermes/migration/openclaw/<timestamp>/archive/` for manual review:

| What | Archive file | How to recreate in Hermes |
|------|-------------|--------------------------|
| `IDENTITY.md` | `archive/workspace/IDENTITY.md` | Merge into `SOUL.md` |
| `TOOLS.md` | `archive/workspace/TOOLS.md` | Hermes has built-in herramienta instructions |
| `HEARTBEAT.md` | `archive/workspace/HEARTBEAT.md` | Use cron jobs for periodic tasks |
| `BOOTSTRAP.md` | `archive/workspace/BOOTSTRAP.md` | Use contexto files or habilidads |
| Cron jobs | `archive/cron-config.json` | Recreate with `hermes cron create` |
| Complementos | `archive/complementos-config.json` | See [complementos guide](/guia-usuario/features/hooks) |
| Hooks/webhooks | `archive/hooks-config.json` | Use `hermes webhook` or puerta de enlace hooks |
| Memoria backend | `archive/memoria-backend-config.json` | Configure via `hermes honcho` |
| Habilidads registry | `archive/habilidads-registry-config.json` | Use `hermes habilidads config` |
| UI/identity | `archive/ui-identity-config.json` | Use `/skin` command |
| Logging | `archive/logging-diagnostics-config.json` | Set in `config.yaml` logging section |
| Multi-agente list | `archive/agentes-list.json` | Use Hermes perfils |
| Channel bindings | `archive/bindings.json` | Manual setup per platform |
| Complex channels | `archive/channels-deep-config.json` | Manual platform config |

## API key resolution

When `--migrate-secrets` is enabled, API keys are collected from **four sources** in priority order:

1. **Config values** — `modelos.proveedors.*.apiKey` and TTS proveedor keys in `openclaw.json`
2. **Environment file** — `~/.openclaw/.env` (keys like `OPENROUTER_API_KEY`, `ANTHROPIC_API_KEY`, etc.)
3. **Config env sub-object** — `openclaw.json` → `"env"` or `"env"."vars"` (some setups store keys here instead of a separate `.env` file)
4. **Auth perfils** — `~/.openclaw/agentes/main/agente/auth-perfils.json` (per-agente credentials)

Config values take priority. Each subsequent source fills any remaining gaps.

### Supported key targets

`OPENROUTER_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `DEEPSEEK_API_KEY`, `GEMINI_API_KEY`, `ZAI_API_KEY`, `MINIMAX_API_KEY`, `ELEVENLABS_API_KEY`, `TELEGRAM_BOT_TOKEN`, `VOICE_TOOLS_OPENAI_KEY`

Keys not in this allowlist are never copied.

## SecretRef handling

OpenClaw config values for tokens and API keys can be in three formats:

```json
// Plain string
"channels": { "telegram": { "botToken": "123456:ABC-DEF..." } }

// Environment template
"channels": { "telegram": { "botToken": "${TELEGRAM_BOT_TOKEN}" } }

// SecretRef object
"channels": { "telegram": { "botToken": { "source": "env", "id": "TELEGRAM_BOT_TOKEN" } } }
```

The migration resolves all three formats. For env templates and SecretRef objects with `source: "env"`, it looks up the value in `~/.openclaw/.env` and the `openclaw.json` env sub-object. SecretRef objects with `source: "file"` or `source: "exec"` can't be resolved automatically — the migration warns about these, and those values must be added to Hermes manually via `hermes config set`.

## After migration

1. **Check the migration report** — printed on completion with counts of migrated, skipped, and conflicting items.

2. **Review archived files** — anything in `~/.hermes/migration/openclaw/<timestamp>/archive/` needs manual attention.

3. **Start a new session** — imported habilidads and memoria entries take effect in new sessions, not the current one.

4. **Verify API keys** — run `hermes status` to check proveedor autenticación.

5. **Test messaging** — if you migrated platform tokens, restart the puerta de enlace: `systemctl --user restart hermes-puerta de enlace`

6. **Check session policies** — run `hermes config show` and verify the `session_reset` value matches your expectations.

7. **Re-pair WhatsApp** — WhatsApp uses QR code pairing (Baileys), not token migration. Run `hermes whatsapp` to pair.

8. **Archive cleanup** — after confirming everything works, run `hermes claw cleanup` to rename leftover OpenClaw directories to `.pre-migration/` (prevents state confusion).

## Troubleshooting

### "OpenClaw directory not found"

The migration checks `~/.openclaw/`, then `~/.clawdbot/`, then `~/.moltbot/`. If your installation is elsewhere, use `--source /path/to/your/openclaw`.

### "No proveedor API keys found"

Keys might be stored in several places depending on your OpenClaw version: inline in `openclaw.json` under `modelos.proveedors.*.apiKey`, in `~/.openclaw/.env`, in the `openclaw.json` `"env"` sub-object, or in `agentes/main/agente/auth-perfils.json`. The migration checks all four. If keys use `source: "file"` or `source: "exec"` SecretRefs, they can't be resolved automatically — add them via `hermes config set`.

### Habilidads not appearing after migration

Imported habilidads land in `~/.hermes/habilidads/openclaw-imports/`. Start a new session for them to take effect, or run `/habilidads` to verify they're loaded.

### TTS voice not migrated

OpenClaw stores TTS settings in two places: `messages.tts.proveedors.*` and the top-level `talk` config. The migration checks both. If your voice ID was set via the OpenClaw UI (stored in a different path), you may need to set it manually: `hermes config set tts.elevenlabs.voice_id YOUR_VOICE_ID`.

---