<!-- source: website/docs/guia-desarrollador/cron-internals.md -->
# Cron Internals

# Cron Internals

The cron subsystem provides scheduled task execution — from simple one-shot delays to recurring cron-expression jobs with habilidad injection and cross-platform delivery.

## Key Files

| File | Purpose |
|------|---------|
| `cron/jobs.py` | Job modelo, storage, atomic read/write to `jobs.json` |
| `cron/scheduler.py` | Scheduler loop — due-job detection, execution, repeat tracking |
| `herramientas/cronjob_herramientas.py` | Modelo-facing `cronjob` herramienta registration and handler |
| `puerta de enlace/run.py` | Puerta de enlace integration — cron ticking in the long-running loop |
| `hermes_cli/cron.py` | CLI `hermes cron` subcommands |

## Scheduling Modelo

Four schedule formats are supported:

| Format | Example | Behavior |
|--------|---------|----------|
| **Relative delay** | `30m`, `2h`, `1d` | One-shot, fires after the specified duration |
| **Interval** | `every 2h`, `every 30m` | Recurring, fires at regular intervals |
| **Cron expression** | `0 9 * * *` | Standard 5-field cron syntax (minute, hour, day, month, weekday) |
| **ISO timestamp** | `2025-01-15T09:00:00` | One-shot, fires at the exact time |

The modelo-facing surface is a single `cronjob` herramienta with action-style operations: `create`, `list`, `update`, `pause`, `resume`, `run`, `remove`.

## Job Storage

Jobs are stored in `~/.hermes/cron/jobs.json` with atomic write semantics (write to temp file, then rename). Each job record contains:

```json
{
  "id": "a1b2c3d4e5f6",
  "name": "Daily briefing",
  "prompt": "Summarize today's AI news and funding rounds",
  "schedule": {
    "kind": "cron",
    "expr": "0 9 * * *",
    "display": "0 9 * * *"
  },
  "habilidads": ["ai-funding-daily-report"],
  "deliver": "telegram:-1001234567890",
  "repeat": {
    "times": null,
    "completed": 42
  },
  "state": "scheduled",
  "enabled": true,
  "next_run_at": "2025-01-16T09:00:00Z",
  "last_run_at": "2025-01-15T09:00:00Z",
  "last_status": "ok",
  "created_at": "2025-01-01T00:00:00Z",
  "modelo": null,
  "proveedor": null,
  "script": null
}
```

### Job Lifecycle States

| State | Meaning |
|-------|---------|
| `scheduled` | Active, will fire at next scheduled time |
| `paused` | Suspended — won't fire until resumed |
| `completed` | Repeat count exhausted or one-shot that has fired |
| `running` | Currently executing (transient state) |

### Backward Compatibility

Older jobs may have a single `habilidad` field instead of the `habilidads` array. The scheduler normalizes this at load time — single `habilidad` is promoted to `habilidads: [habilidad]`.

## Scheduler Runtime

### Tick Cycle

The scheduler runs on a periodic tick (default: every 60 seconds):

```text
tick()
  1. Acquire scheduler lock (prevents overlapping ticks)
  2. Load all jobs from jobs.json
  3. Filter to due jobs (next_run <= now AND state == "scheduled")
  4. For each due job:
     a. Set state to "running"
     b. Create fresh AIAgente session (no conversation history)
     c. Load attached habilidads in order (injected as user messages)
     d. Run the job prompt through the agente
     e. Deliver the response to the configured target
     f. Update run_count, compute next_run
     g. If repeat count exhausted → state = "completed"
     h. Otherwise → state = "scheduled"
  5. Write updated jobs back to jobs.json
  6. Release scheduler lock
```

### Puerta de enlace Integration

In puerta de enlace mode, the cron **trigger** (the part that decides *when* a due job
fires — "Axis B") is selected through a pluggable `CronScheduler` proveedor. The
puerta de enlace calls `resolve_cron_scheduler()` (`cron/scheduler_proveedor.py`) and runs
the resolved proveedor's `start()` in a dedicated background thread, alongside a
separate puerta de enlace-housekeeping thread.

The active proveedor is chosen by the `cron.proveedor` config key:

- **empty (default)** → the built-in `InProcessCronScheduler`, which runs the
  historical in-process loop calling `scheduler.tick()` every 60 seconds. This
  is byte-identical to the pre-proveedor behavior.
- **a named proveedor** (e.g. `chronos`, a managed-cron proveedor for
  scale-to-zero deployments) → discovered from `complementos/cron/<name>/` or
  `$HERMES_HOME/complementos/<name>/`.

If a named proveedor is missing, fails to load, or reports `is_available() ==
False`, the resolver falls back to the built-in with a warning — **cron is
never left without a trigger.** The built-in proveedor lives in core
(`cron/scheduler_proveedor.py`), not in `complementos/`, so the fallback can't be
accidentally removed.

What "firing" *means* (job execution + delivery) is unchanged and shared by all
proveedors — it stays in `scheduler.run_job()` / `scheduler._deliver_result()`.
A proveedor only controls the trigger, never execution.

In CLI mode, cron jobs only fire when `hermes cron` commands are run or during active CLI sessions.

### Managed cron (Chronos) for scale-to-zero

Hosted puerta de enlaces can run the **Chronos** proveedor (`cron.proveedor: chronos`)
instead of the built-in ticker. Chronos lets an idle puerta de enlace **scale to zero**
and still fire cron jobs: rather than a 60-second in-process loop (which would
keep the process awake), it asks Nous infrastructure to arm exactly **one
managed one-shot per job at that job's real next-fire time**. At fire time Nous
calls the puerta de enlace back over an authenticated webhook (`POST /api/cron/fire`);
the puerta de enlace runs the job through the same `run_one_job` path as the built-in,
then re-arms the next one-shot. Between fires the process can be fully stopped —
it wakes only on a genuine fire, never on a periodic timer.

The flow (the managed scheduler is provided by Nous; the agente holds no
scheduler credentials):

```
create/update a cron job
  → Chronos asks Nous to arm a one-shot at the job's next_run_at
      (authenticated with the agente's existing Nous token)
  → at fire time Nous calls the puerta de enlace: POST {callback_url}/api/cron/fire
      (authenticated with a short-lived, purpose-scoped Nous-minted JWT)
  → the puerta de enlace verifies the token, claims the job (store compare-and-set so
    multi-replica deployments fire at-most-once), runs it, and re-arms the next
    one-shot
```

Config (all non-secret; on hosted agentes Nous sets these at provision time):

| key | meaning |
|---|---|
| `cron.proveedor` | `chronos` to activate (empty = built-in ticker) |
| `cron.chronos.portal_url` | Nous base URL (arming + the fire-token issuer) |
| `cron.chronos.callback_url` | the puerta de enlace's own public base URL for inbound fires |
| `cron.chronos.expected_audience` | this agente's fire-token audience |
| `cron.chronos.nas_jwks_url` | key set for verifying the inbound fire token |

If Chronos is misconfigured or the agente isn't logged into Nous,
`resolve_cron_scheduler()` falls back to the built-in ticker (logged warning) —
cron never loses its trigger. Recurring jobs re-arm after each fire; `repeat`-N
jobs stop cleanly when the count is exhausted (no orphaned one-shot). The full
agente↔Nous wire contract lives in `docs/chronos-managed-cron-contract.md`.

### Fresh Session Isolation

Each cron job runs in a completely fresh agente session:

- No conversation history from previous runs
- No memoria of previous cron executions (unless persisted to memoria/files)
- The prompt must be self-contained — cron jobs cannot ask clarifying questions
- The `cronjob` herramientaset is disabled (recursion guard)

## Habilidad-Backed Jobs

A cron job can attach one or more habilidads via the `habilidads` field. At execution time:

1. Habilidads are loaded in the specified order
2. Each habilidad's SKILL.md content is injected as contexto
3. The job's prompt is appended as the task instruction
4. The agente processes the combined habilidad contexto + prompt

This enables reusable, tested workflows without pasting full instructions into cron prompts. For example:

```
Create a daily funding report → attach "ai-funding-daily-report" habilidad
```

### Script-Backed Jobs

Jobs can also attach a Python script via the `script` field. The script runs *before* each agente turn, and its stdout is injected into the prompt as contexto. This enables data collection and change detection patterns:

```python
# ~/.hermes/scripts/check_competitors.py
import requests, json
# Fetch competitor release notes, diff against last run
# Print summary to stdout — agente analyzes and reports
```

The script timeout defaults to 120 seconds. `_get_script_timeout()` resolves the limit through a three-layer chain:

1. **Module-level override** — `_SCRIPT_TIMEOUT` (for tests/monkeypatching). Only used when it differs from the default.
2. **Environment variable** — `HERMES_CRON_SCRIPT_TIMEOUT`
3. **Config** — `cron.script_timeout_seconds` in `config.yaml` (read via `load_config()`)
4. **Default** — 120 seconds

### Proveedor Recovery

`run_job()` passes the user's configured fallback proveedors and credential pool into the `AIAgente` instance:

- **Fallback proveedors** — reads `fallback_proveedors` (list) or `fallback_modelo` (legacy dict) from `config.yaml`, matching the puerta de enlace's `_load_fallback_modelo()` pattern. Passed as `fallback_modelo=` to `AIAgente.__init__`, which normalizes both formats into a fallback chain.
- **Credential pool** — loads via `load_pool(proveedor)` from `agente.credential_pool` using the resolved runtime proveedor name. Only passed when the pool has credentials (`pool.has_credentials()`). Enables same-proveedor key rotation on 429/rate-limit errors.

This mirrors the puerta de enlace's behavior — without it, cron agentes would fail on rate limits without attempting recovery.

## Delivery Modelo

Cron job results can be delivered to any supported platform:

| Target | Syntax | Example |
|--------|--------|---------|
| Origin chat | `origin` | Deliver to the chat where the job was created |
| Local file | `local` | Save to `~/.hermes/cron/output/` |
| Telegram | `telegram` or `telegram:<chat_id>` | `telegram:-1001234567890` |
| Discord | `discord` or `discord:#channel` | `discord:#engineering` |
| Slack | `slack` | Deliver to Slack home channel |
| WhatsApp | `whatsapp` | Deliver to WhatsApp home |
| Signal | `signal` | Deliver to Signal |
| Matrix | `matrix` | Deliver to Matrix home room |
| Mattermost | `mattermost` | Deliver to Mattermost home |
| Email | `email` | Deliver via email |
| SMS | `sms` | Deliver via SMS |
| Home Assistant | `homeassistant` | Deliver to HA conversation |
| DingTalk | `dingtalk` | Deliver to DingTalk |
| Feishu | `feishu` | Deliver to Feishu |
| WeCom | `wecom` | Deliver to WeCom |
| Weixin | `weixin` | Deliver to Weixin (WeChat) |
| BlueBubbles | `bluebubbles` | Deliver to iMessage via BlueBubbles |
| QQ Bot | `qqbot` | Deliver to QQ (Tencent) via Official API v2 |

For Telegram topics, use the format `telegram:<chat_id>:<thread_id>` (e.g., `telegram:-1001234567890:17585`).

### Response Wrapping

By default (`cron.wrap_response: true`), cron deliveries are wrapped with:
- A header identifying the cron job name and task
- A footer noting the agente cannot see the delivered message in conversation

The `[SILENT]` prefix in a cron response suppresses delivery entirely — useful for jobs that only need to write to files or perform side effects.

### Session Isolation

Cron deliveries are NOT mirrored into puerta de enlace session conversation history. They exist only in the cron job's own session. This prevents message alternation violations in the target chat's conversation.

## Recursion Guard

Cron-run sessions have the `cronjob` herramientaset disabled. This prevents:
- A scheduled job from creating new cron jobs
- Recursive scheduling that could explode token usage
- Accidental mutation of the job schedule from within a job

## Locking

The scheduler uses cross-process file-based locking (`fcntl.flock` on Unix, `msvcrt.locking` on Windows) to prevent overlapping ticks from executing the same due-job batch twice — even between the puerta de enlace's in-process ticker and a standalone `hermes cron` / manual `tick()` call. If the lock cannot be acquired, `tick()` returns 0 immediately.

## CLI Interface

The `hermes cron` CLI provides direct job management:

```bash
hermes cron list                    # Show all jobs
hermes cron create                  # Interactive job creation (alias: add)
hermes cron edit <job_id>           # Edit job configuración
hermes cron pause <job_id>          # Pause a running job
hermes cron resume <job_id>         # Resume a paused job
hermes cron run <job_id>            # Trigger immediate execution
hermes cron remove <job_id>         # Delete a job
```

## Related Docs

- [Cron Feature Guide](/guia-usuario/features/cron)
- [Puerta de enlace Internals](./puerta de enlace-internals.md)
- [Agente Loop Internals](./agente-loop.md)

---