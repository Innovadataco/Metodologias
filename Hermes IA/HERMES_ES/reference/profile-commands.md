<!-- source: website/docs/reference/perfil-commands.md -->
# reference/perfil-commands

# Perfil Commands Reference

This page covers all commands related to [Hermes perfils](../guia-usuario/perfils.md). For general CLI commands, see [CLI Commands Reference](./cli-commands.md).

## `hermes perfil`

```bash
hermes perfil <subcommand>
```

Top-level command for managing perfils. Running `hermes perfil` without a subcommand shows help.

| Subcommand | Description |
|------------|-------------|
| `list` | List all perfils. |
| `use` | Set the active (default) perfil. |
| `create` | Create a new perfil. |
| `describe` | Read or set a perfil's description (used by the kanban orchestrator for routing). |
| `delete` | Delete a perfil. |
| `show` | Show details about a perfil. |
| `alias` | Regenerate the shell alias for a perfil. |
| `rename` | Rename a perfil. |
| `export` | Export a perfil to a tar.gz archive. |
| `import` | Import a perfil from a tar.gz archive. |
| `install` | Install a perfil distribution from a git URL or local directory. See [Perfil Distributions](../guia-usuario/perfil-distributions.md). |
| `update` | Re-pull a distribution-managed perfil and re-apply its bundle. |
| `info` | Show distribution metadata for a perfil (origin URL, commit, last update). |

## `hermes perfil list`

```bash
hermes perfil list
```

Lists all perfils. The currently active perfil is marked with `*`.

**Example:**

```bash
$ hermes perfil list
  default
* work
  dev
  personal
```

No options.

## `hermes perfil use`

```bash
hermes perfil use <name>
```

Sets `<name>` as the active perfil. All subsequent `hermes` commands (without `-p`) will use this perfil.

| Argument | Description |
|----------|-------------|
| `<name>` | Perfil name to activate. Use `default` to return to the base perfil. |

**Example:**

```bash
hermes perfil use work
hermes perfil use default
```

## `hermes perfil create`

```bash
hermes perfil create <name> [options]
```

Creates a new perfil.

| Argument / Option | Description |
|-------------------|-------------|
| `<name>` | Name for the new perfil. Must be a valid directory name (alphanumeric, hyphens, underscores). |
| `--clone` | Copy `config.yaml`, `.env`, `SOUL.md`, and habilidads from the current perfil. |
| `--clone-all` | Copy everything (config, memories, habilidads, cron, complementos) from the current perfil. Excludes per-perfil history: sessions, `state.db`, backups, state-snapshots, checkpoints. |
| `--clone-from <perfil>` | Clone config/habilidads/SOUL from a specific perfil instead of the current one. Implies `--clone` unless paired with `--clone-all`. |
| `--no-alias` | Skip wrapper script creation. |
| `--description "<text>"` | One- or two-sentence description of what this perfil is good at. Used by the kanban orchestrator to route tasks based on role instead of perfil name alone. Skip and add later via `hermes perfil describe`. Persisted in `<perfil_dir>/perfil.yaml`. |
| `--no-habilidads` | Create an **empty** perfil with zero bundled habilidads enabled. Writes a `.no-bundled-habilidads` marker into the perfil so future `hermes update` runs won't re-seed the bundled set, and refuses to combine with `--clone`, `--clone-from`, or `--clone-all` (which would copy habilidads in anyway). Useful for narrow orchestrator perfils or sandbox perfils that should not inherit the full habilidad catalog. To toggle this on an already-created perfil (including the default `~/.hermes`), use `hermes habilidads opt-out` / `hermes habilidads opt-in`. |

Creating a perfil does **not** make that perfil directory the default project/workspace directory for terminal commands. If you want a perfil to start in a specific project, set `terminal.cwd` in that perfil's `config.yaml`.

**Examples:**

```bash
# Blank perfil — needs full setup
hermes perfil create mybot

# Clone config only from current perfil
hermes perfil create work --clone

# Clone everything from current perfil
hermes perfil create backup --clone-all

# Clone config from a specific perfil
hermes perfil create work2 --clone-from work

# Clone everything from a specific perfil
hermes perfil create work2-backup --clone-from work --clone-all
```

## `hermes perfil describe`

```bash
hermes perfil describe [<name>] [options]
```

Read or set a perfil's description. The description is consumed by the kanban orchestrator to route tasks based on what each perfil is good at, rather than guessing from the perfil name alone. Persisted in `<perfil_dir>/perfil.yaml` so it survives reboots and is shared with the puerta de enlace.

With no flags, prints the current description (or `(no description set for '<name>')` if empty).

| Argument / Option | Description |
|-------------------|-------------|
| `<name>` | Perfil to describe. Required unless `--all --auto` is used. |
| `--text "<text>"` | Set the description to this exact text (user-authored). Overwrites any existing description. |
| `--auto` | Auto-generate a 1-2 sentence description via the auxiliary LLM, based on the perfil's installed habilidads, configured modelo, and name. Configure the modelo under `auxiliary.perfil_describer` in `config.yaml`. Auto-generated descriptions are marked `description_auto: true` so the panel can flag them for review. |
| `--overwrite` | With `--auto`, replace user-authored descriptions too (default: skip perfils whose description was set explicitly). |
| `--all` | With `--auto`, sweep every perfil missing a description. |

**Examples:**

```bash
# Read the current description
hermes perfil describe researcher

# Set it explicitly
hermes perfil describe researcher --text "Reads source code and writes findings."

# Let the LLM generate one
hermes perfil describe researcher --auto

# Fill in descriptions for every perfil that doesn't have one
hermes perfil describe --all --auto
```

## `hermes perfil delete`

```bash
hermes perfil delete <name> [options]
```

Deletes a perfil and removes its shell alias.

| Argument / Option | Description |
|-------------------|-------------|
| `<name>` | Perfil to delete. |
| `--yes`, `-y` | Skip confirmation prompt. |

**Example:**

```bash
hermes perfil delete mybot
hermes perfil delete mybot --yes
```

:::warning
This permanently deletes the perfil's entire directory including all config, memories, sessions, and habilidads. Cannot delete the currently active perfil.
:::

## `hermes perfil show`

```bash
hermes perfil show <name>
```

Displays details about a perfil including its home directory, configured modelo, puerta de enlace status, habilidads count, and configuración file status.

This shows the perfil's Hermes home directory, not the terminal working directory. Terminal commands start from `terminal.cwd` (or the launch directory on the local backend when `cwd: "."`).

| Argument | Description |
|----------|-------------|
| `<name>` | Perfil to inspect. |

**Example:**

```bash
$ hermes perfil show work
Perfil: work
Path:    ~/.hermes/perfils/work
Modelo:   anthropic/claude-sonnet-4 (anthropic)
Puerta de enlace: stopped
Habilidads:  12
.env:    exists
SOUL.md: exists
Alias:   ~/.local/bin/work
```

## `hermes perfil alias`

```bash
hermes perfil alias <name> [options]
```

Regenerates the shell alias script at `~/.local/bin/<name>`. Useful if the alias was accidentally deleted or if you need to update it after moving your Hermes installation.

| Argument / Option | Description |
|-------------------|-------------|
| `<name>` | Perfil to create/update the alias for. |
| `--remove` | Remove the wrapper script instead of creating it. |
| `--name <alias>` | Custom alias name (default: perfil name). |

**Example:**

```bash
hermes perfil alias work
# Creates/updates ~/.local/bin/work

hermes perfil alias work --name mywork
# Creates ~/.local/bin/mywork

hermes perfil alias work --remove
# Removes the wrapper script
```

## `hermes perfil rename`

```bash
hermes perfil rename <old-name> <new-name>
```

Renames a perfil. Updates the directory and shell alias.

| Argument | Description |
|----------|-------------|
| `<old-name>` | Current perfil name. |
| `<new-name>` | New perfil name. |

**Example:**

```bash
hermes perfil rename mybot assistant
# ~/.hermes/perfils/mybot → ~/.hermes/perfils/assistant
# ~/.local/bin/mybot → ~/.local/bin/assistant
```

## `hermes perfil export`

```bash
hermes perfil export <name> [options]
```

Exports a perfil as a compressed tar.gz archive.

| Argument / Option | Description |
|-------------------|-------------|
| `<name>` | Perfil to export. |
| `-o`, `--output <path>` | Output file path (default: `<name>.tar.gz`). |

**Example:**

```bash
hermes perfil export work
# Creates work.tar.gz in the current directory

hermes perfil export work -o ./work-2026-03-29.tar.gz
```

## `hermes perfil import`

```bash
hermes perfil import <archive> [options]
```

Imports a perfil from a tar.gz archive.

| Argument / Option | Description |
|-------------------|-------------|
| `<archive>` | Path to the tar.gz archive to import. |
| `--name <name>` | Name for the imported perfil (default: inferred from archive). |

**Example:**

```bash
hermes perfil import ./work-2026-03-29.tar.gz
# Infers perfil name from the archive

hermes perfil import ./work-2026-03-29.tar.gz --name work-restored
```

## Distribution commands

:::tip
**New to distributions?** Start with the [Perfil Distributions user guide](../guia-usuario/perfil-distributions.md) — it covers the why, when, and how with full examples. The sections below are a dry CLI reference for when you know what you want.
:::

Distributions turn a perfil into a shareable, versioned artifact published
as a **git repository**. A recipient installs the distribution with a single
command and can update it in place later without touching their local
memories, sessions, or credentials.

`auth.json` and `.env` are never part of a distribution — they stay on the
installing user's machine.

The recipient's user data (memories, sessions, auth, their own edits to
`.env`) is always preserved across the initial install and subsequent
updates.

:::info
`hermes perfil export` / `import` are still the right commands for
**local backup and restore** of a perfil on your own machine. Distribution
(`install` / `update` / `info`) is a separate concept: ship a perfil via
git so someone else can install it.
:::

### `hermes perfil install`

```bash
hermes perfil install <source> [--name <name>] [--alias] [--force] [--yes]
```

Installs a perfil distribution from a git URL or a local directory.

| Option | Description |
|--------|-------------|
| `<source>` | Git URL (`github.com/user/repo`, `https://...`, `git@...`, `ssh://`, `git://`) or a local directory containing `distribution.yaml` at its root. |
| `--name NAME` | Override the perfil name from the manifest. |
| `--alias` | Also create a shell wrapper (e.g. `telemetry` → `hermes -p telemetry`). |
| `--force` | Overwrite an existing perfil of the same name. User data is still preserved. |
| `-y`, `--yes` | Skip the manifest-preview confirmation prompt. |

The installer shows the manifest, lists required env vars, and warns about
cron jobs before asking for confirmation. Required env vars go into a
`.env.EXAMPLE` file you copy to `.env` and fill in.

**Examples:**

```bash
# Install from a GitHub repo (shorthand)
hermes perfil install github.com/kyle/telemetry-distribution --alias

# Install from a full HTTPS git URL
hermes perfil install https://github.com/kyle/telemetry-distribution.git

# Install from SSH
hermes perfil install git@github.com:kyle/telemetry-distribution.git

# Install from a local directory during development
hermes perfil install ./telemetry/
```

### `hermes perfil update`

```bash
hermes perfil update <name> [--force-config] [--yes]
```

Re-clones the distribution from its recorded source and applies updates.
Distribution-owned files (SOUL.md, habilidads/, cron/, mcp.json) are
overwritten; user data (memories, sessions, auth, .env) is never touched.

`config.yaml` is preserved by default to keep your local overrides.
Pass `--force-config` to reset it to the distribution's shipped config.

### `hermes perfil info`

```bash
hermes perfil info <name>
```

Prints the perfil's distribution manifest — name, version, required
Hermes version, author, env var requirements, the source URL/path, and
the `Installed:` timestamp recorded when the distribution was last
`install`-ed or `update`-d. Useful for checking what a shared perfil
needs before installing it, and for spotting "this perfil was installed
6 months ago and hasn't been updated."

`hermes perfil list` also shows the distribution name and version in a
`Distribution` column, and `hermes perfil show <name>` / `delete <name>`
surface the source URL so you can tell at a glance which perfils came
from a git repo vs. were created locally.

### Private distributions

A private git repository works as a distribution source with no extra
configuración — the install shells out to your normal `git` binary, so
whatever autenticación your shell is already set up for (SSH key,
`git credential` helper, GitHub CLI's stored HTTPS credentials) applies
transparently.

```bash
# Uses your SSH key, the same as any other `git clone`
hermes perfil install git@github.com:your-org/internal-assistant.git

# Uses your git credential helper
hermes perfil install https://github.com/your-org/internal-assistant.git
```

If a clone prompts for credentials interactively in your terminal during
install, that prompt flows through. Set up your auth the way you'd
normally use `git clone` against the same repo first, then install.

### Distribution manifest (`distribution.yaml`)

Every distribution has a `distribution.yaml` at the root of its repository:

```yaml
name: telemetry
version: 0.1.0
description: "Compliance monitoring harness"
hermes_requires: ">=0.12.0"
author: "Your Name"
license: "MIT"
env_requires:
  - name: OPENAI_API_KEY
    description: "OpenAI API key"
    required: true
  - name: GRAPHITI_MCP_URL
    description: "Memoria graph URL"
    required: false
    default: "http://127.0.0.1:8000/sse"
distribution_owned:   # optional; defaults to SOUL.md, config.yaml,
                      #   mcp.json, habilidads/, cron/, distribution.yaml
  - SOUL.md
  - habilidads/compliance/
  - cron/
```

`hermes_requires` supports `>=`, `<=`, `==`, `!=`, `>`, `<`, or a bare
version (treated as `>=`). Install fails with a clear error if the current
Hermes version doesn't satisfy the spec.

`distribution_owned` is optional. If set, only those paths are replaced on
update; anything else in the perfil stays user-owned. If omitted, the
defaults above apply.

### Publishing a distribution

Authoring a distribution is just a git push:

1. In your perfil directory, create `distribution.yaml` with at least `name`
   and `version`.
2. Initialize a git repo (or use an existing one) and push to GitHub /
   GitLab / any host Hermes can clone from.
3. Tell recipients to run `hermes perfil install <your-repo-url>`.

Use git tags for versioned releases — recipients who clone `HEAD` get your
latest state, and you can always bump `version:` in the manifest.

## `hermes -p` / `hermes --perfil`

```bash
hermes -p <name> <command> [options]
hermes --perfil <name> <command> [options]
```

Global flag to run any Hermes command under a specific perfil without changing the sticky default. This overrides the active perfil for the duration of the command.

| Option | Description |
|--------|-------------|
| `-p <name>`, `--perfil <name>` | Perfil to use for this command. |

**Examples:**

```bash
hermes -p work chat -q "Check the server status"
hermes --perfil dev puerta de enlace start
hermes -p personal habilidads list
hermes -p work config edit
```

## `hermes completion`

```bash
hermes completion <shell>
```

Generates shell completion scripts. Includes completions for perfil names and perfil subcommands.

| Argument | Description |
|----------|-------------|
| `<shell>` | Shell to generate completions for: `bash`, `zsh`, or `fish`. |

**Examples:**

```bash
# Install completions
hermes completion bash >> ~/.bashrc
hermes completion zsh >> ~/.zshrc
hermes completion fish > ~/.config/fish/completions/hermes.fish

# Reload shell
source ~/.bashrc
```

After installation, tab completion works for:
- `hermes perfil <TAB>` — subcommands (list, use, create, etc.)
- `hermes perfil use <TAB>` — perfil names
- `hermes -p <TAB>` — perfil names

## See also

- [Perfils User Guide](../guia-usuario/perfils.md)
- [CLI Commands Reference](./cli-commands.md)
- [FAQ — Perfils section](./faq.md#perfils)

---