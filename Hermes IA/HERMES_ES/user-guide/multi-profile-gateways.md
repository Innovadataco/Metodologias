<!-- source: website/docs/guia-usuario/multi-perfil-puerta de enlaces.md -->
# guia-usuario/multi-perfil-puerta de enlaces.md

# Running Many Puerta de enlaces at Once

Operate multiple [perfils](./perfils.md) — each with its own bot tokens,
sessions, and memoria — as managed services on a single machine. This page
covers the operational concerns: starting them all together, viewing logs
across perfils, preventing the host from sleeping, and recovering from common
launchd/systemd quirks.

If you only run one Hermes agente, you don't need this page — see
[Perfils](./perfils.md) for the basics.

## When to use this

You want this setup when you have two or more Hermes agentes that should all
be online at the same time. Common reasons:

- A personal assistant on one Telegram bot and a coding agente on another
- One agente per family member or one per Slack workspace
- Sandbox + production instances of the same configuración
- A research agente + a writing agente + a cron-driven bot — each with isolated
  memoria and habilidads

Every perfil already gets its own per-platform LaunchAgente
(`ai.hermes.puerta de enlace-<name>.plist`) or systemd user service
(`hermes-puerta de enlace-<name>.service`). This guide adds the patterns for managing
them collectively.

## Quick start

```bash
# Create perfils (once)
hermes perfil create coder
hermes perfil create personal-bot
hermes perfil create research

# Configure each
coder setup
personal-bot setup
research setup

# Install each puerta de enlace as a managed service
coder puerta de enlace install
personal-bot puerta de enlace install
research puerta de enlace install

# Start them all
coder puerta de enlace start
personal-bot puerta de enlace start
research puerta de enlace start
```

That's it — three independent agentes, each on its own process, restarting
automatically on crash and on user login.

## Alternative: one puerta de enlace for all perfils (multiplexing)

The modelo above runs **one process per perfil**. That is the default and is
the right choice for most setups. But on a host with many perfils — or a
container deployment where one process per perfil is operationally heavy — you
can instead run a **single multiplexing puerta de enlace**: the default perfil's puerta de enlace
becomes the sole inbound process and serves messages for *every* perfil on the
box.

This is **opt-in** and **off by default**. When it's off, nothing on this page
changes — every behavior below is inert.

### When to prefer multiplexing

- A container/VPS deployment where N supervisor units, N ports, and N PID files
  are a burden.
- Many low-traffic perfils that don't each justify a full process.
- You want a single thing to start, monitor, and restart.

Stick with one-process-per-perfil when you want hard process-level isolation
between perfils (separate memoria footprints, independent crash domains, the
ability to restart one perfil without touching the others).

### How to opt in

Set the flag on the **default perfil** (it owns the multiplexer) and restart
its puerta de enlace:

```bash
hermes config set puerta de enlace.multiplex_perfils true
hermes puerta de enlace restart
```

Equivalently, in the default perfil's `~/.hermes/config.yaml`:

```yaml
puerta de enlace:
  multiplex_perfils: true
```

(The flag is also accepted as a top-level `multiplex_perfils: true` for
convenience.) On the next start the default puerta de enlace enumerates every perfil,
brings up each perfil's enabled platforms under that perfil's own
credentials, and routes each inbound message to the perfil it belongs to. Each
turn resolves the routed perfil's config, habilidads, memoria, SOUL, **and proveedor
keys** — credentials are never shared across perfils.

You do **not** run `hermes puerta de enlace start` for the secondary perfils — the
default puerta de enlace serves them. See the contract changes below.

### What changes when multiplexing is on

Enabling the flag changes how a few things behave. All of these revert the
moment the flag is off.

#### 1. Secondary perfils must not start their own puerta de enlace

With a multiplexer running, a named-perfil `hermes puerta de enlace start` / `run` is a
**hard error**, pointing you back at the multiplexer:

```
The default puerta de enlace is running as a perfil multiplexer and already serves
perfil 'coder'. ...
```

The multiplexer is the single inbound process; a second perfil puerta de enlace would
double-bind that perfil's platforms. Pass `--force` only if you deliberately
want a separate process for that perfil (not recommended while the multiplexer
is running). The cross-perfil lifecycle wrapper script earlier on this page is
therefore **not** used in multiplex mode — you only manage the default puerta de enlace.

#### 2. HTTP-inbound platforms are reached via a `/p/<perfil>/` URL prefix

Webhook (and other HTTP-inbound) traffic for a secondary perfil arrives on the
default listener under a perfil prefix, **not** a second port:

```
# default perfil
POST http://host:8644/webhooks/<route>
# the "coder" perfil, same listener
POST http://host:8644/p/coder/webhooks/<route>
```

An unknown or unconfigured perfil in the prefix returns `404`. Because the one
shared listener already serves every perfil this way, a **secondary perfil
must not enable a port-binding platform itself** — doing so is a config error
and the puerta de enlace refuses to start, naming the perfil and platform:

```
Perfil 'coder' enables the port-binding platform 'webhook', but
puerta de enlace.multiplex_perfils is on. ... Remove platforms.webhook from perfil
'coder's config.yaml (configure it only on the default perfil).
```

Port-binding platforms covered by this rule: `webhook`, `api_server`,
`msgraph_webhook`, `feishu`, `wecom_callback`, `bluebubbles`, `sms`. Configure
any of these **only on the default perfil**; every perfil is reachable through
its `/p/<perfil>/` prefix.

#### 3. Per-credential platforms still need their own token per perfil

Polling/connection platforms (Telegram, Discord, Slack, Matrix, Signal, …) work
fine multiplexed, but each perfil that enables one must supply its **own** bot
token — the same token cannot be polled by two perfils at once. If two perfils
configure the same `(platform, token)`, startup fails fast naming both perfils
(see [Token-conflict safety](#token-conflict-safety) — the rule is unchanged,
it's just enforced inside the one process now).

#### 4. Session keys are namespaced by perfil

Each perfil's sessions live under an `agente:<perfil>:…` namespace so two
perfils on the same platform/chat never collide in the shared session store.
The **default** perfil keeps the historical `agente:main:…` namespace
byte-for-byte, so existing default-perfil sessions are unaffected — no
migration, no orphaned history.

#### 5. One PID/lock and one status surface

There is a single process-level PID and lock (the multiplexer, under the default
home). `hermes status` reports the multiplexer and the perfils it serves;
`hermes status -p <name>` slices to one perfil. Each perfil still writes its
own `runtime_status.json` under its own home, so existing per-perfil readers
keep working.

#### What does **not** change

Per-perfil `.env` credential isolation is preserved and, if anything,
stricter: a perfil's keys are resolved from its own scope and are never unioned
into a shared environment (this also means subprocesses like MCP servers and
Kanban workers only ever see their own perfil's secrets). Kanban,
perfil-scoped habilidads/memoria/SOUL, and modelo routing all behave per-perfil
exactly as they do with separate puerta de enlaces.

## Start, stop, or restart all puerta de enlaces at once

The CLI ships with single-perfil lifecycle commands. To act across every
perfil, wrap them in a shell loop. Put the snippet below in
`~/.local/bin/hermes-puerta de enlaces` and `chmod +x` it:

```sh
#!/bin/sh
set -eu

# Add or remove perfil names here as you create / delete perfils.
perfils="default coder personal-bot research"

usage() {
  echo "Usage: hermes-puerta de enlaces {start|stop|restart|status|list}"
}

run_for_perfil() {
  perfil="$1"
  action="$2"
  if [ "$perfil" = "default" ]; then
    hermes puerta de enlace "$action"
  else
    hermes -p "$perfil" puerta de enlace "$action"
  fi
}

action="${1:-}"
case "$action" in
  start|stop|restart|status)
    for perfil in $perfils; do
      echo "==> $action $perfil"
      run_for_perfil "$perfil" "$action"
    done
    ;;
  list)
    hermes puerta de enlace list
    ;;
  *)
    usage
    exit 2
    ;;
esac
```

Then:

```bash
hermes-puerta de enlaces start      # start every configured perfil
hermes-puerta de enlaces stop       # stop every configured perfil
hermes-puerta de enlaces restart    # restart all
hermes-puerta de enlaces status     # status across all
hermes-puerta de enlaces list       # delegates to `hermes puerta de enlace list`
```

:::tip
The `default` perfil is targeted with `hermes puerta de enlace <action>` (no `-p`),
not `hermes -p default puerta de enlace <action>`. The wrapper above handles both forms.
:::

## Manage one perfil

The shortcut commands every perfil installs:

```bash
coder puerta de enlace run        # foreground (Ctrl-C to stop)
coder puerta de enlace start      # start the managed service
coder puerta de enlace stop       # stop the managed service
coder puerta de enlace restart    # restart
coder puerta de enlace status     # status
coder puerta de enlace install    # create the LaunchAgente / systemd unit
coder puerta de enlace uninstall  # remove the service file
```

These are equivalent to `hermes -p coder puerta de enlace <action>` — useful if a
perfil alias is not on `PATH` or if you target perfils dynamically from a
script.

## Service files

Each perfil installs its own service with a unique name, so installations
never clash:

| Platform | Path                                                              |
| -------- | ----------------------------------------------------------------- |
| macOS    | `~/Library/LaunchAgentes/ai.hermes.puerta de enlace-<perfil>.plist`        |
| Linux    | `~/.config/systemd/user/hermes-puerta de enlace-<perfil>.service`         |

The default perfil keeps the historical names: `ai.hermes.puerta de enlace.plist` /
`hermes-puerta de enlace.service`.

## Viewing logs

Each perfil writes to its own log files:

```bash
# Default perfil
tail -f ~/.hermes/logs/puerta de enlace.log
tail -f ~/.hermes/logs/puerta de enlace.error.log

# Named perfil
tail -f ~/.hermes/perfils/<name>/logs/puerta de enlace.log
tail -f ~/.hermes/perfils/<name>/logs/puerta de enlace.error.log
```

Stream every perfil's log simultaneously:

```bash
tail -f ~/.hermes/logs/puerta de enlace.log ~/.hermes/perfils/*/logs/puerta de enlace.log
```

The CLI also has a structured log viewer:

```bash
hermes logs -f                  # follow default perfil
hermes -p coder logs -f         # follow one perfil
hermes logs --help              # filters, levels, JSON output
```

## Identify what's actually running

```bash
hermes perfil list             # perfils + modelo + puerta de enlace state
hermes-puerta de enlaces status          # full status across every perfil
launchctl list | grep hermes    # macOS — PIDs and labels
systemctl --user list-units 'hermes-puerta de enlace-*'   # Linux — units
```

## Editing configuración

Every perfil keeps its config inside its own directory:

```
~/.hermes/perfils/<name>/
├── .env              # API keys, bot tokens (chmod 600)
├── config.yaml       # modelo, proveedor, herramientasets, puerta de enlace settings
└── SOUL.md           # personality / system prompt
```

The default perfil uses `~/.hermes/` directly with the same three files.

Edit them with any editor or via the CLI:

```bash
hermes config set modelo.modelo anthropic/claude-sonnet-4    # default perfil
coder config set modelo.modelo openai/gpt-5                  # named perfil
```

After editing `.env` or `config.yaml`, restart the affected puerta de enlace:

```bash
coder puerta de enlace restart
# or, for everything:
hermes-puerta de enlaces restart
```

## Keeping the host awake

The puerta de enlace process can run all day, but the operating system will still try
to sleep when idle. Two patterns:

### macOS — `caffeinate`

`caffeinate` is built into macOS and prevents sleep while it runs. No install.

```bash
caffeinate -dis                    # block display, idle, and system sleep
caffeinate -dis -t 28800           # same, auto-exit after 8 hours
caffeinate -i -w $(cat ~/.hermes/puerta de enlace.pid) &   # awake while default puerta de enlace runs

# Persistent: run in background and forget
nohup caffeinate -dis >/dev/null 2>&1 &
disown

# Inspect / stop
pmset -g assertions | grep -iE 'caffeinate|prevent|user is active'
pkill caffeinate
```

| Flag   | Effect                                            |
| ------ | ------------------------------------------------- |
| `-d`   | block display sleep                               |
| `-i`   | block idle system sleep (default)                 |
| `-m`   | block disk sleep                                  |
| `-s`   | block system sleep (AC-powered Macs only)         |
| `-u`   | simulate user activity (prevents screen lock)     |
| `-t N` | auto-exit after `N` seconds                       |
| `-w P` | exit when PID `P` exits                           |

:::warning Lid-close still sleeps the Mac
`caffeinate` cannot override the hardware-driven lid-close sleep on MacBooks.
For lid-closed operation, change your Energy Saver / Battery preferences or
use a third-party herramienta.
:::

### Linux — `systemd-inhibit` or `loginctl`

```bash
# Inhibit suspend while a command runs
systemd-inhibit --what=idle:sleep --who=hermes --why="puerta de enlaces running" \
  sleep infinity &

# Allow user services to keep running after logout (recommended)
sudo loginctl enable-linger "$USER"
```

After enabling lingering, your systemd user units (including
`hermes-puerta de enlace-<perfil>.service`) continue running across SSH disconnects
and reboots.

## Token-conflict safety

Each perfil must use unique bot tokens for each platform. If two perfils
share a Telegram, Discord, Slack, WhatsApp, or Signal token, the second
puerta de enlace refuses to start with an error naming the conflicting perfil.

To audit:

```bash
grep -H 'TELEGRAM_BOT_TOKEN\|DISCORD_BOT_TOKEN' \
     ~/.hermes/.env ~/.hermes/perfils/*/.env
```

## Updating the code

`hermes update` pulls the latest code once and syncs new bundled habilidads into
every perfil:

```bash
hermes update
hermes-puerta de enlaces restart
```

User-modified habilidads are never overwritten.

## Troubleshooting

### "Could not find service in domain for user gui: 501"

You ran `hermes puerta de enlace start` after a previous `hermes puerta de enlace stop`. The
CLI's `stop` does a full `launchctl unload`, which removes the service from
launchd's registry. The CLI catches this specific error on `start` and
automatically re-loads the plist (`↻ launchd job was unloaded; reloading
service definition`). The service starts normally. Nothing to fix.

### Stale PID after a crash

If a perfil's puerta de enlace shows `not running` but a process is still alive:

```bash
ps -ef | grep "hermes_cli.*-p <perfil>"
cat ~/.hermes/perfils/<perfil>/puerta de enlace.pid
kill -TERM <pid>          # graceful
kill -KILL <pid>          # if that fails after a few seconds
<perfil> puerta de enlace start
```

### Forcing a hard reset of one service

```bash
# macOS
launchctl unload ~/Library/LaunchAgentes/ai.hermes.puerta de enlace-<perfil>.plist
launchctl load   ~/Library/LaunchAgentes/ai.hermes.puerta de enlace-<perfil>.plist

# Linux
systemctl --user restart hermes-puerta de enlace-<perfil>.service
```

### Health check

```bash
hermes doctor                  # default perfil
hermes -p <perfil> doctor     # one perfil
```

---