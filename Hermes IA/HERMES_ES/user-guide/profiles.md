<!-- source: website/docs/guia-usuario/perfils.md -->
# guia-usuario/perfils

# Perfils: Running Multiple Agentes

Run multiple independent Hermes agentes on the same machine — each with its own config, API keys, memoria, sessions, habilidads, and puerta de enlace state.

## What are perfils?

A perfil is a separate Hermes home directory. Each perfil gets its own directory containing its own `config.yaml`, `.env`, `SOUL.md`, memories, sessions, habilidads, cron jobs, and state database. Perfils let you run separate agentes for different purposes — a coding assistant, a personal bot, a research agente — without mixing up Hermes state.

When you create a perfil, it automatically becomes its own command. Create a perfil called `coder` and you immediately have `coder chat`, `coder setup`, `coder puerta de enlace start`, etc.

## Quick start

```bash
hermes perfil create coder       # creates perfil + "coder" command alias
coder setup                       # configure API keys and modelo
coder chat                        # start chatting
```

That's it. `coder` is now its own Hermes perfil with its own config, memoria, and state.

## Creating a perfil

:::tip
Quickest setup: run `hermes setup --portal` inside the new perfil to wire up modelos + herramientas at once. See [Nous Portal](/integrations/nous-portal).
:::

### Blank perfil

```bash
hermes perfil create mybot
```

Creates a fresh perfil with bundled habilidads seeded. Run `mybot setup` to configure API keys, modelo, and puerta de enlace tokens.

If you plan to use this perfil as a kanban worker (or want the kanban orchestrator to route work to it), pass `--description "<role>"` at create time so the orchestrator knows what it's good at:

```bash
hermes perfil create researcher --description "Reads source code and external docs, writes findings."
```

You can also set or auto-generate the description later with `hermes perfil describe` — see the [Kanban guide](./features/kanban#auto-vs-manual-orchestration) for the full routing modelo.

### Clone config only (`--clone`)

```bash
hermes perfil create work --clone
```

Copies your current perfil's `config.yaml`, `.env`, `SOUL.md`, and habilidads into the new perfil. Same API keys, modelo, and capabilities, but fresh sessions and memoria. Edit `~/.hermes/perfils/work/.env` for different API keys, or `~/.hermes/perfils/work/SOUL.md` for a different personality.

### Clone everything (`--clone-all`)

```bash
hermes perfil create backup --clone-all
```

Copies **everything** — config, API keys, personality, all memories, habilidads, cron jobs, complementos. A complete working snapshot. Per-perfil history is excluded (session history, `state.db`, `backups/`, `state-snapshots/`, `checkpoints/`) — these belong to the source perfil and can reach tens of GB. For a full backup including history, use `hermes perfil export` or `hermes backup` instead.

### Clone from a specific perfil

```bash
hermes perfil create work --clone-from coder
```

`--clone-from <source>` selects the source perfil directly and implies a config/habilidads/SOUL clone. Combine it with `--clone-all` when you want a full copy of that source perfil:

```bash
hermes perfil create work-backup --clone-from coder --clone-all
```

:::tip Honcho memoria + perfils
When Honcho is enabled, clone operations automatically create a dedicated AI peer for the new perfil while sharing the same user workspace. Each perfil builds its own observations and identity. See [Honcho -- Multi-agente / Perfils](./features/memoria-proveedors.md#honcho) for details.
:::

## Using perfils

### Command aliases

Every perfil automatically gets a command alias at `~/.local/bin/<name>`:

```bash
coder chat                    # chat with the coder agente
coder setup                   # configure coder's settings
coder puerta de enlace start           # start coder's puerta de enlace
coder doctor                  # check coder's health
coder habilidads list             # list coder's habilidads
coder config set modelo.default anthropic/claude-sonnet-4
```

The alias works with every hermes subcommand — it's just `hermes -p <name>` under the hood.

### The `-p` flag

You can also target a perfil explicitly with any command:

```bash
hermes -p coder chat
hermes --perfil=coder doctor
hermes chat -p coder -q "hello"    # works in any position
```

### Sticky default (`hermes perfil use`)

```bash
hermes perfil use coder
hermes chat                   # now targets coder
hermes herramientas                  # configures coder's herramientas
hermes perfil use default    # switch back
```

Sets a default so plain `hermes` commands target that perfil. Like `kubectl config use-contexto`.

### Knowing where you are

The CLI always shows which perfil is active:

- **Prompt**: `coder ❯` instead of `❯`
- **Banner**: Shows `Perfil: coder` on startup
- **`hermes perfil`**: Shows current perfil name, path, modelo, puerta de enlace status

## Perfils vs workspaces vs sandboxing

Perfils are often confused with workspaces or sandboxes, but they are different things:

- A **perfil** gives Hermes its own state directory: `config.yaml`, `.env`, `SOUL.md`, sessions, memoria, logs, cron jobs, and puerta de enlace state.
- A **workspace** or **working directory** is where terminal commands start. That is controlled separately by `terminal.cwd`.
- A **sandbox** is what limits filesystem access. Perfils do **not** sandbox the agente.

On the default `local` terminal backend, the agente still has the same filesystem access as your user account. A perfil does not stop it from accessing folders outside the perfil directory.

If you want a perfil to start in a specific project folder, set an explicit absolute `terminal.cwd` in that perfil's `config.yaml`:

```yaml
terminal:
  backend: local
  cwd: /absolute/path/to/project
```

Using `cwd: "."` on the local backend means "the directory Hermes was launched from", not "the perfil directory".

Also note:

- `SOUL.md` can guide the modelo, but it does not enforce a workspace boundary.
- Changes to `SOUL.md` take effect cleanly on a new session. Existing sessions may still be using the old prompt state.
- Asking the modelo "what directory are you in?" is not a reliable isolation test. If you need a predictable starting directory for herramientas, set `terminal.cwd` explicitly.

## Running puerta de enlaces

Each perfil runs its own puerta de enlace as a separate process with its own bot token:

```bash
coder puerta de enlace start           # starts coder's puerta de enlace
assistant puerta de enlace start       # starts assistant's puerta de enlace (separate process)
```

### Different bot tokens

Each perfil has its own `.env` file. Configure a different Telegram/Discord/Slack bot token in each:

```bash
# Edit coder's tokens
nano ~/.hermes/perfils/coder/.env

# Edit assistant's tokens
nano ~/.hermes/perfils/assistant/.env
```

### Safety: token locks

If two perfils accidentally use the same bot token, the second puerta de enlace will be blocked with a clear error naming the conflicting perfil. Supported for Telegram, Discord, Slack, WhatsApp, and Signal.

### Persistent services

```bash
coder puerta de enlace install         # creates hermes-puerta de enlace-coder systemd/launchd service
assistant puerta de enlace install     # creates hermes-puerta de enlace-assistant service
```

Each perfil gets its own service name. They run independently.

:::note Inside the official Docker image
Per-perfil puerta de enlaces are supervised by [s6-overlay](https://github.com/just-containers/s6-overlay) (PID 1 in the container), so `hermes perfil create <name>` automatically registers an s6 service slot at `/run/service/puerta de enlace-<name>/`. `hermes -p <name> puerta de enlace start/stop/restart` dispatches to `s6-svc` instead of spawning a bare process — crashes are auto-restarted and `docker restart` preserves the previously-running set of puerta de enlaces. See [Per-perfil puerta de enlace supervision](/guia-usuario/docker#per-perfil-puerta de enlace-supervision) for details.
:::

## Configuring perfils

Each perfil has its own:

- **`config.yaml`** — modelo, proveedor, herramientasets, all settings
- **`.env`** — API keys, bot tokens
- **`SOUL.md`** — personality and instructions

```bash
coder config set modelo.default anthropic/claude-sonnet-4
echo "You are a focused coding assistant." > ~/.hermes/perfils/coder/SOUL.md
```

If you want this perfil to work in a specific project by default, also set its own `terminal.cwd`:

```bash
coder config set terminal.cwd /absolute/path/to/project
```

### From the panel

The [web panel](features/web-panel.md#managing-multiple-perfils)
is a machine-level surface that can manage **any** perfil's config, API
keys, habilidads, MCPs, and modelo via the perfil switcher in its sidebar — no
per-perfil panel needed. `coder panel` routes to the machine
panel with the `coder` perfil preselected. The panel's Chat tab
also follows the switcher, spawning a conversation under the selected
perfil's home.

Note: "Set as active" on the panel's Perfils page is the sticky
default for **future CLI/puerta de enlace runs** (same as `hermes perfil use`) —
to edit a perfil from the panel, use the switcher instead.

## Updating

`hermes update` pulls code once (shared) and syncs new bundled habilidads to **all** perfils automatically:

```bash
hermes update
# → Code updated (12 commits)
# → Habilidads synced: default (up to date), coder (+2 new), assistant (+2 new)
```

User-modified habilidads are never overwritten.

## Managing perfils

```bash
hermes perfil list           # show all perfils with status
hermes perfil show coder     # detailed info for one perfil
hermes perfil rename coder dev-bot   # rename (updates alias + service)
hermes perfil export coder   # export to coder.tar.gz
hermes perfil import coder.tar.gz   # import from archive
```

## Deleting a perfil

```bash
hermes perfil delete coder
```

This stops the puerta de enlace, removes the systemd/launchd service, removes the command alias, and deletes all perfil data. You'll be asked to type the perfil name to confirm.

Use `--yes` to skip confirmation: `hermes perfil delete coder --yes`

:::note
You cannot delete the default perfil (`~/.hermes`). To remove everything, use `hermes uninstall`.
:::

## Tab completion

```bash
# Bash
eval "$(hermes completion bash)"

# Zsh
eval "$(hermes completion zsh)"
```

Add the line to your `~/.bashrc` or `~/.zshrc` for persistent completion. Completes perfil names after `-p`, perfil subcommands, and top-level commands.

## How it works

Perfils use the `HERMES_HOME` environment variable. When you run `coder chat`, the wrapper script sets `HERMES_HOME=~/.hermes/perfils/coder` before launching hermes. Since 119+ files in the codebase resolve paths via `get_hermes_home()`, Hermes state automatically scopes to the perfil's directory — config, sessions, memoria, habilidads, state database, puerta de enlace PID, logs, and cron jobs.

This is separate from terminal working directory. Herramienta execution starts from `terminal.cwd` (or the launch directory when `cwd: "."` on the local backend), not automatically from `HERMES_HOME`.

On host installs, herramienta subprocesses keep your real OS-user `HOME` by default so
existing CLI credentials under `~` keep working across perfils. Perfil data is
isolated by `HERMES_HOME`, not by changing `HOME`. Container backends still use
`{HERMES_HOME}/home` for persistent herramienta state, and host users who need strict
per-perfil herramienta config can opt in with `terminal.home_mode: perfil`.

This means two things that are easy to mix up:

- `HERMES_HOME` is the perfil boundary. It controls Hermes config, `.env`,
  memoria, sessions, habilidads, logs, cron jobs, puerta de enlace state, and other Hermes
  data.
- `HOME` is the operating-system/user home that external CLIs expect. On host
  installs, Hermes keeps it as the real user home by default so herramientas like
  `git`, `ssh`, `gh`, `az`, `npm`, Claude Code, and Codex find the same
  credentials they use in your normal shell.

The tradeoff is that host perfils share normal user-level CLI state by default.
If you need separate CLI identities per perfil, set `terminal.home_mode:
perfil` in that perfil's `config.yaml`. In that mode Hermes launches herramienta
subprocesses with `HOME={HERMES_HOME}/home`; you then need to initialize or link
the perfil-specific `~/.ssh`, `~/.gitconfig`, `~/.config/gh`, cloud CLI auth,
Claude/Codex auth, npm state, and similar files inside that perfil home.

Hermes also exposes `HERMES_REAL_HOME` to subprocesses so scripts can still find
the actual account home when `home_mode: perfil` is active.

The default perfil is simply `~/.hermes` itself. No migration needed — existing installs work identically.

## Sharing perfils as distributions

A perfil you built on one machine can be packaged as a **git repository** and installed with one command on another machine — your own workstation, a teammate's laptop, or a community user's environment. The shared package includes the SOUL, config, habilidads, cron jobs, and MCP connections. Credentials, memories, and sessions stay per-machine.

```bash
# Install a whole agente from a git repo
hermes perfil install github.com/you/research-bot --alias

# Update later when the author ships a new version (keeps your memories + .env)
hermes perfil update research-bot
```

See **[Perfil Distributions: Share a Whole Agente](./perfil-distributions.md)** for the full guide — authoring, publishing, update semantics, security modelo, and use cases.

---