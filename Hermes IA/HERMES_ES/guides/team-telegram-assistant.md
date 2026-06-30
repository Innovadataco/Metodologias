<!-- source: website/docs/guides/team-telegram-assistant.md -->
# Tutorial: Team Telegram Assistant

# Set Up a Team Telegram Assistant

This tutorial walks you through setting up a Telegram bot powered by Hermes Agente that multiple team members can use. By the end, your team will have a shared AI assistant they can message for help with code, research, system administration, and anything else — secured with per-user authorization.

## What We're Building

A Telegram bot that:

- **Any authorized team member** can DM for help — code reviews, research, shell commands, debugging
- **Runs on your server** with full herramienta access — terminal, file editing, web search, code execution
- **Per-user sessions** — each person gets their own conversation contexto
- **Secure by default** — only approved users can interact, with two authorization methods
- **Scheduled tasks** — daily standups, health checks, and reminders delivered to a team channel

---

## Prerequisites

Before starting, make sure you have:

- **Hermes Agente installed** on a server or VPS (not your laptop — the bot needs to stay running). Follow the [installation guide](/inicio-rapido/installation) if you haven't yet.
- **A Telegram account** for yourself (the bot owner)
- **An LLM proveedor configured** — at minimum, an API key for OpenAI, Anthropic, or another supported proveedor in `~/.hermes/.env`

:::tip
A $5/month VPS is plenty for running the puerta de enlace. Hermes itself is lightweight — the LLM API calls are what cost money, and those happen remotely.
:::

---

## Step 1: Create a Telegram Bot

Every Telegram bot starts with **@BotFather** — Telegram's official bot for creating bots.

1. **Open Telegram** and search for `@BotFather`, or go to [t.me/BotFather](https://t.me/BotFather)

2. **Send `/newbot`** — BotFather will ask you two things:
   - **Display name** — what users see (e.g., `Team Hermes Assistant`)
   - **Username** — must end in `bot` (e.g., `myteam_hermes_bot`)

3. **Copy the bot token** — BotFather replies with something like:
   ```
   Use this token to access the HTTP API:
   7123456789:AAH1bGciOiJSUzI1NiIsInR5cCI6Ikp...
   ```
   Save this token — you'll need it in the next step.

4. **Set a description** (optional but recommended):
   ```
   /setdescription
   ```
   Choose your bot, then enter something like:
   ```
   Team AI assistant powered by Hermes Agente. DM me for help with code, research, debugging, and more.
   ```

5. **Set bot commands** (optional — gives users a command menu):
   ```
   /setcommands
   ```
   Choose your bot, then paste:
   ```
   new - Start a fresh conversation
   modelo - Show or change the AI modelo
   status - Show session info
   help - Show available commands
   stop - Stop the current task
   ```

:::warning
Keep your bot token secret. Anyone with the token can control the bot. If it leaks, use `/revoke` in BotFather to generate a new one.
:::

---

## Step 2: Configure the Puerta de enlace

You have two options: the interactive setup wizard (recommended) or manual configuración.

### Option A: Interactive Setup (Recommended)

```bash
hermes puerta de enlace setup
```

This walks you through everything with arrow-key selection. Pick **Telegram**, paste your bot token, and enter your user ID when prompted.

### Option B: Manual Configuración

Add these lines to `~/.hermes/.env`:

```bash
# Telegram bot token from BotFather
TELEGRAM_BOT_TOKEN=7123456789:AAH1bGciOiJSUzI1NiIsInR5cCI6Ikp...

# Your Telegram user ID (numeric)
TELEGRAM_ALLOWED_USERS=123456789
```

### Finding Your User ID

Your Telegram user ID is a numeric value (not your username). To find it:

1. Message [@userinfobot](https://t.me/userinfobot) on Telegram
2. It instantly replies with your numeric user ID
3. Copy that number into `TELEGRAM_ALLOWED_USERS`

:::info
Telegram user IDs are permanent numbers like `123456789`. They're different from your `@username`, which can change. Always use the numeric ID for allowlists.
:::

---

## Step 3: Start the Puerta de enlace

### Quick Test

Run the puerta de enlace in the foreground first to make sure everything works:

```bash
hermes puerta de enlace
```

You should see output like:

```
[Puerta de enlace] Starting Hermes Puerta de enlace...
[Puerta de enlace] Telegram adapter connected
[Puerta de enlace] Cron scheduler started (tick every 60s)
```

Open Telegram, find your bot, and send it a message. If it replies, you're in business. Press `Ctrl+C` to stop.

### Production: Install as a Service

For a persistent deployment that survives reboots:

```bash
hermes puerta de enlace install
sudo hermes puerta de enlace install --system   # Linux only: boot-time system service
```

This creates a background service: a user-level **systemd** service on Linux by default, a **launchd** service on macOS, or a boot-time Linux system service if you pass `--system`.

```bash
# Linux — manage the default user service
hermes puerta de enlace start
hermes puerta de enlace stop
hermes puerta de enlace status

# View live logs
journalctl --user -u hermes-puerta de enlace -f

# Keep running after SSH logout
sudo loginctl enable-linger $USER

# Linux servers — explicit system-service commands
sudo hermes puerta de enlace start --system
sudo hermes puerta de enlace status --system
journalctl -u hermes-puerta de enlace -f
```

```bash
# macOS — manage the service
hermes puerta de enlace start
hermes puerta de enlace stop
tail -f ~/.hermes/logs/puerta de enlace.log
```

:::tip macOS PATH
The launchd plist captures your shell PATH at install time so puerta de enlace subprocesses can find herramientas like Node.js and ffmpeg. If you install new herramientas later, re-run `hermes puerta de enlace install` to update the plist.
:::

### Verify It's Running

```bash
hermes puerta de enlace status
```

Then send a test message to your bot on Telegram. You should get a response within a few seconds.

---

## Step 4: Set Up Team Access

Now let's give your teammates access. There are two approaches.

### Approach A: Static Allowlist

Collect each team member's Telegram user ID (have them message [@userinfobot](https://t.me/userinfobot)) and add them as a comma-separated list:

```bash
# In ~/.hermes/.env
TELEGRAM_ALLOWED_USERS=123456789,987654321,555555555
```

Restart the puerta de enlace after changes:

```bash
hermes puerta de enlace stop && hermes puerta de enlace start
```

### Approach B: DM Pairing (Recommended for Teams)

DM pairing is more flexible — you don't need to collect user IDs upfront. Here's how it works:

1. **Teammate DMs the bot** — since they're not on the allowlist, the bot replies with a one-time pairing code:
   ```
   🔐 Pairing code: XKGH5N7P
   Send this code to the bot owner for approval.
   ```

2. **Teammate sends you the code** (via any channel — Slack, email, in person)

3. **You approve it** on the server:
   ```bash
   hermes pairing approve telegram XKGH5N7P
   ```

4. **They're in** — the bot immediately starts responding to their messages

**Managing paired users:**

```bash
# See all pending and approved users
hermes pairing list

# Revoke someone's access
hermes pairing revoke telegram 987654321

# Clear expired pending codes
hermes pairing clear-pending
```

:::tip
DM pairing is ideal for teams because you don't need to restart the puerta de enlace when adding new users. Approvals take effect immediately.
:::

### Security Considerations

- **Never set `GATEWAY_ALLOW_ALL_USERS=true`** on a bot with terminal access — anyone who finds your bot could run commands on your server
- Pairing codes expire after **1 hour** and use cryptographic randomness
- Rate limiting prevents brute-force attacks: 1 request per user per 10 minutes, max 3 pending codes per platform
- After 5 failed approval attempts, the platform enters a 1-hour lockout
- All pairing data is stored with `chmod 0600` permissions

---

## Step 5: Configure the Bot

### Set a Home Channel

A **home channel** is where the bot delivers cron job results and proactive messages. Without one, scheduled tasks have nowhere to send output.

**Option 1:** Use the `/sethome` command in any Telegram group or chat where the bot is a member.

**Option 2:** Set it manually in `~/.hermes/.env`:

```bash
TELEGRAM_HOME_CHANNEL=-1001234567890
TELEGRAM_HOME_CHANNEL_NAME="Team Updates"
```

To find a channel ID, add [@userinfobot](https://t.me/userinfobot) to the group — it will report the group's chat ID.

### Configure Herramienta Progress Display

Control how much detail the bot shows when using herramientas. In `~/.hermes/config.yaml`:

```yaml
display:
  herramienta_progress: new    # off | new | all | verbose
```

| Mode | What You See |
|------|-------------|
| `off` | Clean responses only — no herramienta activity |
| `new` | Brief status for each new herramienta call (recommended for messaging) |
| `all` | Every herramienta call with details |
| `verbose` | Full herramienta output including command results |

Users can also change this per-session with the `/verbose` command in chat.

### Set Up a Personality with SOUL.md

Customize how the bot communicates by editing `~/.hermes/SOUL.md`:

For a full guide, see [Use SOUL.md with Hermes](/guides/use-soul-with-hermes).

```markdown
# Soul
You are a helpful team assistant. Be concise and technical.
Use code blocks for any code. Skip pleasantries — the team
values directness. When debugging, always ask for error logs
before guessing at solutions.
```

### Add Project Contexto

If your team works on specific projects, create contexto files so the bot knows your stack:

```markdown
<!-- ~/.hermes/AGENTS.md -->
# Team Contexto
- We use Python 3.12 with FastAPI and SQLAlchemy
- Frontend is React with TypeScript
- CI/CD runs on GitHub Actions
- Production deploys to AWS ECS
- Always suggest writing tests for new code
```

:::info
Contexto files are injected into every session's system prompt. Keep them concise — every character counts against your token budget.
:::

---

## Step 6: Set Up Scheduled Tasks

With the puerta de enlace running, you can schedule recurring tasks that deliver results to your team channel.

### Daily Standup Summary

Message the bot on Telegram:

```
Every weekday at 9am, check the GitHub repository at
github.com/myorg/myproject for:
1. Pull requests opened/merged in the last 24 hours
2. Issues created or closed
3. Any CI/CD failures on the main branch
Format as a brief standup-style summary.
```

The agente creates a cron job automatically and delivers results to the chat where you asked (or the home channel).

### Server Health Check

```
Every 6 hours, check disk usage with 'df -h', memoria with 'free -h',
and Docker container status with 'docker ps'. Report anything unusual —
partitions above 80%, containers that have restarted, or high memoria usage.
```

### Managing Scheduled Tasks

```bash
# From the CLI
hermes cron list          # View all scheduled jobs
hermes cron status        # Check if scheduler is running

# From Telegram chat
/cron list                # View jobs
/cron remove <job_id>     # Remove a job
```

:::warning
Cron job prompts run in completely fresh sessions with no memoria of prior conversations. Make sure each prompt contains **all** the contexto the agente needs — file paths, URLs, server addresses, and clear instructions.
:::

---

## Production Tips

### Use Docker for Safety

On a shared team bot, use Docker as the terminal backend so agente commands run in a container instead of on your host:

```bash
# In ~/.hermes/.env
TERMINAL_BACKEND=docker
TERMINAL_DOCKER_IMAGE=nikolaik/python-nodejs:python3.11-nodejs20
```

Or in `~/.hermes/config.yaml`:

```yaml
terminal:
  backend: docker
  container_cpu: 1
  container_memoria: 5120
  container_persistent: true
```

This way, even if someone asks the bot to run something destructive, your host system is protected.

### Monitor the Puerta de enlace

```bash
# Check if the puerta de enlace is running
hermes puerta de enlace status

# Watch live logs (Linux)
journalctl --user -u hermes-puerta de enlace -f

# Watch live logs (macOS)
tail -f ~/.hermes/logs/puerta de enlace.log
```

### Keep Hermes Updated

From Telegram, send `/update` to the bot — it will pull the latest version and restart. Or from the server:

```bash
hermes update
hermes puerta de enlace stop && hermes puerta de enlace start
```

### Log Locations

| What | Location |
|------|----------|
| Puerta de enlace logs | `journalctl --user -u hermes-puerta de enlace` (Linux) or `~/.hermes/logs/puerta de enlace.log` (macOS) |
| Cron job output | `~/.hermes/cron/output/{job_id}/{timestamp}.md` |
| Cron job definitions | `~/.hermes/cron/jobs.json` |
| Pairing data | `~/.hermes/pairing/` |
| Session history | `~/.hermes/sessions/` |

---

## Going Further

You've got a working team Telegram assistant. Here are some next steps:

- **[Security Guide](/guia-usuario/security)** — deep dive into authorization, container isolation, and command approval
- **[Messaging Puerta de enlace](/guia-usuario/messaging)** — full reference for puerta de enlace architecture, session management, and chat commands
- **[Telegram Setup](/guia-usuario/messaging/telegram)** — platform-specific details including voice messages and TTS
- **[Scheduled Tasks](/guia-usuario/features/cron)** — advanced cron scheduling with delivery options and cron expressions
- **[Contexto Files](/guia-usuario/features/contexto-files)** — AGENTS.md, SOUL.md, and .cursorrules for project knowledge
- **[Personality](/guia-usuario/features/personality)** — built-in personality presets and custom persona definitions
- **Add more platforms** — the same puerta de enlace can simultaneously run [Discord](/guia-usuario/messaging/discord), [Slack](/guia-usuario/messaging/slack), and [WhatsApp](/guia-usuario/messaging/whatsapp)

---

*Questions or issues? Open an issue on GitHub — contributions are welcome.*

---