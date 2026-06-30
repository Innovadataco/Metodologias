<!-- source: website/docs/guides/cron-troubleshooting.md -->
# Cron Troubleshooting

# Cron Troubleshooting

When a cron job isn't behaving as expected, work through these checks in order. Most issues fall into one of four categories: timing, delivery, permissions, or habilidad loading.

---

## Jobs Not Firing

### Check 1: Verify the job exists and is active

```bash
hermes cron list
```

Look for the job and confirm its state is `[active]` (not `[paused]` or `[completed]`). If it shows `[completed]`, the repeat count may be exhausted — edit the job to reset it.

### Check 2: Confirm the schedule is correct

A misformatted schedule silently defaults to one-shot or is rejected entirely. Test your expression:

| Your expression | Should evaluate to |
|----------------|-------------------|
| `0 9 * * *` | 9:00 AM every day |
| `0 9 * * 1` | 9:00 AM every Monday |
| `every 2h` | Every 2 hours from now |
| `30m` | 30 minutes from now |
| `2025-06-01T09:00:00` | June 1, 2025 at 9:00 AM UTC |

If the job fires once and then disappears from the list, it's a one-shot schedule (`30m`, `1d`, or an ISO timestamp) — expected behavior.

### Check 3: Is the puerta de enlace running?

Cron jobs are fired by the puerta de enlace's background ticker thread, which ticks every 60 seconds. A regular CLI chat session does **not** automatically fire cron jobs.

If you're expecting jobs to fire automatically, you need a running puerta de enlace (`hermes puerta de enlace` for foreground, or `hermes puerta de enlace start` for the installed service). For one-off debugging, you can manually trigger a tick with `hermes cron tick`.

### Check 4: Check the system clock and timezone

Jobs use the local timezone. If your machine's clock is wrong or in a different timezone than expected, jobs will fire at the wrong times. Verify:

```bash
date
hermes cron list   # Compare next_run times with local time
```

---

## Delivery Failures

### Check 1: Verify the deliver target is correct

Delivery targets are case-sensitive and require the correct platform to be configured. A misconfigured target silently drops the response.

| Target | Requires |
|--------|----------|
| `telegram` | `TELEGRAM_BOT_TOKEN` in `~/.hermes/.env` |
| `discord` | `DISCORD_BOT_TOKEN` in `~/.hermes/.env` |
| `slack` | `SLACK_BOT_TOKEN` in `~/.hermes/.env` |
| `whatsapp` | WhatsApp puerta de enlace configured |
| `signal` | Signal puerta de enlace configured |
| `matrix` | Matrix homeserver configured |
| `email` | SMTP configured in `config.yaml` |
| `sms` | SMS proveedor configured |
| `local` | Write access to `~/.hermes/cron/output/` |
| `origin` | Delivers to the chat where the job was created |

Other supported platforms include `mattermost`, `homeassistant`, `dingtalk`, `feishu`, `wecom`, `weixin`, `bluebubbles`, `qqbot`, and `webhook`. You can also target a specific chat with `platform:chat_id` syntax (e.g., `telegram:-1001234567890`).

If delivery fails, the job still runs — it just won't send anywhere. Check `hermes cron list` for updated `last_error` field (if available).

### Check 2: Check `[SILENT]` usage

If your cron job produces no output, delivery is suppressed. If the agente response includes the cron quiet marker `[SILENT]`, delivery is also suppressed. This is intentional for monitoring jobs — but make sure your prompt is not accidentally suppressing everything.

Use prompts like "respond with only [SILENT] if nothing changed." Avoid asking the agente to include `[SILENT]` inside a longer explanation, because cron treats that marker as a suppression signal.

### Check 3: Platform token permissions

Each messaging platform bot needs specific permissions to receive messages. If delivery silently fails:

- **Telegram**: Bot must be an admin in the target group/channel
- **Discord**: Bot must have permission to send in the target channel
- **Slack**: Bot must be added to the workspace and have `chat:write` scope

### Check 4: Response wrapping

By default, cron responses are wrapped with a header and footer (`cron.wrap_response: true` in `config.yaml`). Some platforms or integrations may not handle this well. To disable:

```yaml
cron:
  wrap_response: false
```

---

## Habilidad Loading Failures

### Check 1: Verify habilidads are installed

```bash
hermes habilidads list
```

Habilidads must be installed before they can be attached to cron jobs. If a habilidad is missing, install it first with `hermes habilidads install <habilidad-name>` or via `/habilidads` in the CLI.

### Check 2: Check habilidad name vs. habilidad folder name

Habilidad names are case-sensitive and must match the installed habilidad's folder name. If your job specifies `ai-funding-daily-report` but the habilidad folder is `ai-funding-daily-report`, confirm the exact name from `hermes habilidads list`.

### Check 3: Habilidads that require interactive herramientas

Cron jobs run with the `cronjob`, `messaging`, and `clarify` herramientasets disabled. This prevents recursive cron creation, direct message sending (delivery is handled by the scheduler), and interactive prompts. If a habilidad relies on these herramientasets, it won't work in a cron contexto.

Check the habilidad's documentation to confirm it works in non-interactive (headless) mode.

### Check 4: Multi-habilidad ordering

When using multiple habilidads, they load in order. If Habilidad A depends on contexto from Habilidad B, make sure B loads first:

```bash
/cron add "0 9 * * *" "..." --habilidad contexto-habilidad --habilidad target-habilidad
```

In this example, `contexto-habilidad` loads before `target-habilidad`.

---

## Job Errors and Failures

### Check 1: Review recent job output

If a job ran and failed, you may see error contexto in:

1. The chat where the job delivers (if delivery succeeded)
2. `~/.hermes/logs/agente.log` for scheduler messages (or `errors.log` for warnings)
3. The job's `last_run` metadata via `hermes cron list`

### Check 2: Common error patterns

**"No such file or directory" for scripts**
The `script` path must be an absolute path (or relative to the Hermes config directory). Verify:
```bash
ls ~/.hermes/scripts/your-script.py   # Must exist
hermes cron edit <job_id> --script ~/.hermes/scripts/your-script.py
```

**"Habilidad not found" at job execution**
The habilidad must be installed on the machine running the scheduler. If you move between machines, habilidads don't automatically sync — reinstall them with `hermes habilidads install <habilidad-name>`.

**Job runs but delivers nothing**
Likely a delivery target issue (see Delivery Failures above), no output, or a response containing the cron quiet marker `[SILENT]`.

**Job hangs or times out**
The scheduler uses an inactivity-based timeout (default 600s, configurable via `HERMES_CRON_TIMEOUT` env var, `0` for unlimited). The agente can run as long as it's actively calling herramientas — the timer only fires after sustained inactivity. Long-running jobs should use scripts to handle data collection and deliver only the result.

### Check 3: Lock contention

The scheduler uses file-based locking to prevent overlapping ticks. If two puerta de enlace instances are running (or a CLI session conflicts with a puerta de enlace), jobs may be delayed or skipped.

Kill duplicate puerta de enlace processes:
```bash
ps aux | grep hermes
# Kill duplicate processes, keep only one
```

### Check 4: Permissions on jobs.json

Jobs are stored in `~/.hermes/cron/jobs.json`. If this file is not readable/writable by your user, the scheduler will fail silently:

```bash
ls -la ~/.hermes/cron/jobs.json
chmod 600 ~/.hermes/cron/jobs.json   # Your user should own it
```

---

## Performance Issues

### Slow job startup

Each cron job creates a fresh AIAgente session, which may involve proveedor autenticación and modelo loading. For time-sensitive schedules, add buffer time (e.g., `0 8 * * *` instead of `0 9 * * *`).

### Too many overlapping jobs

The scheduler executes jobs sequentially within each tick. If multiple jobs are due at the same time, they run one after another. Consider staggering schedules (e.g., `0 9 * * *` and `5 9 * * *` instead of both at `0 9 * * *`) to avoid delays.

### Large script output

Scripts that dump megabytes of output will slow down the agente and may hit token limits. Filter/summarize at the script level — emit only what the agente needs to reason about.

---

## Diagnostic Commands

```bash
hermes cron list                    # Show all jobs, states, next_run times
hermes cron run <job_id>            # Schedule for next tick (for testing)
hermes cron edit <job_id>           # Fix configuración issues
hermes logs                         # View recent Hermes logs
hermes habilidads list                  # Verify installed habilidads
```

---

## Getting More Help

If you've worked through this guide and the issue persists:

1. Run the job with `hermes cron run <job_id>` (fires on next puerta de enlace tick) and watch for errors in the chat output
2. Check `~/.hermes/logs/agente.log` for scheduler messages and `~/.hermes/logs/errors.log` for warnings
3. Open an issue at [github.com/NousResearch/hermes-agente](https://github.com/NousResearch/hermes-agente) with:
   - The job ID and schedule
   - The delivery target
   - What you expected vs. what happened
   - Relevant error messages from the logs

---

*For the complete cron reference, see [Automate Anything with Cron](/guides/automate-with-cron) and [Scheduled Tasks (Cron)](/guia-usuario/features/cron).*

---