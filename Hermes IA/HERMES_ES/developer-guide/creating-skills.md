<!-- source: website/docs/guia-desarrollador/creating-habilidads.md -->
# Creating Habilidads

# Creating Habilidads

Habilidads are the preferred way to add new capabilities to Hermes Agente. They're easier to create than herramientas, require no code changes to the agente, and can be shared with the community.

## Should it be a Habilidad or a Herramienta?

Make it a **Habilidad** when:
- The capability can be expressed as instructions + shell commands + existing herramientas
- It wraps an external CLI or API that the agente can call via `terminal` or `web_extract`
- It doesn't need custom Python integration or API key management baked into the agente
- Examples: arXiv search, git workflows, Docker management, PDF processing, email via CLI herramientas

Make it a **Herramienta** when:
- It requires end-to-end integration with API keys, auth flows, or multi-component configuración
- It needs custom processing logic that must execute precisely every time
- It handles binary data, streaming, or real-time events
- Examples: browser automation, TTS, vision analysis

## Habilidad Directory Structure

Bundled habilidads live in `habilidads/` organized by category. Official optional habilidads use the same structure in `optional-habilidads/`:

```text
habilidads/
├── research/
│   └── arxiv/
│       ├── SKILL.md              # Required: main instructions
│       └── scripts/              # Optional: helper scripts
│           └── search_arxiv.py
├── productivity/
│   └── ocr-and-documents/
│       ├── SKILL.md
│       ├── scripts/
│       └── references/
└── ...
```

## SKILL.md Format

```markdown
---
name: my-habilidad
description: Brief description (shown in habilidad search results)
version: 1.0.0
author: Your Name
license: MIT
platforms: [macos, linux]          # Optional — restrict to specific OS platforms
                                   #   Valid: macos, linux, windows
                                   #   Omit to load on all platforms (default)
metadata:
  hermes:
    tags: [Category, Subcategory, Keywords]
    related_habilidads: [other-habilidad-name]
    requires_herramientasets: [web]            # Optional — only show when these herramientasets are active
    requires_herramientas: [web_search]        # Optional — only show when these herramientas are available
    fallback_for_herramientasets: [browser]    # Optional — hide when these herramientasets are active
    fallback_for_herramientas: [browser_navigate]  # Optional — hide when these herramientas exist
    config:                              # Optional — config.yaml settings the habilidad needs
      - key: my.setting
        description: "What this setting controls"
        default: "sensible-default"
        prompt: "Display prompt for setup"
    blueprint:                              # Optional — marks this habilidad a runnable automation
      schedule: "0 9 * * *"              #   cron expr / "every 2h" / ISO timestamp
      deliver: origin                    #   optional (default origin)
      prompt: "Task instruction for each run"  # optional
      no_agente: false                    # optional
required_environment_variables:          # Optional — env vars the habilidad needs
  - name: MY_API_KEY
    prompt: "Enter your API key"
    help: "Get one at https://example.com"
    required_for: "API access"
---

# Habilidad Title

Brief intro.

## When to Use
Trigger conditions — when should the agente load this habilidad?

## Quick Reference
Table of common commands or API calls.

## Procedure
Step-by-step instructions the agente follows.

## Pitfalls
Known failure modes and how to handle them.

## Verification
How the agente confirms it worked.
```

### Platform-Specific Habilidads

Habilidads can restrict themselves to specific operating systems using the `platforms` field:

```yaml
platforms: [macos]            # macOS only (e.g., iMessage, Apple Reminders)
platforms: [macos, linux]     # macOS and Linux
platforms: [windows]          # Windows only
```

When set, the habilidad is automatically hidden from the system prompt, `habilidads_list()`, and slash commands on incompatible platforms. If omitted or empty, the habilidad loads on all platforms (backward compatible).

### Conditional Habilidad Activation

Habilidads can declare dependencies on specific herramientas or herramientasets. This controls whether the habilidad appears in the system prompt for a given session.

```yaml
metadata:
  hermes:
    requires_herramientasets: [web]           # Hide if the web herramientaset is NOT active
    requires_herramientas: [web_search]       # Hide if web_search herramienta is NOT available
    fallback_for_herramientasets: [browser]   # Hide if the browser herramientaset IS active
    fallback_for_herramientas: [browser_navigate]  # Hide if browser_navigate IS available
```

| Field | Behavior |
|-------|----------|
| `requires_herramientasets` | Habilidad is **hidden** when ANY listed herramientaset is **not** available |
| `requires_herramientas` | Habilidad is **hidden** when ANY listed herramienta is **not** available |
| `fallback_for_herramientasets` | Habilidad is **hidden** when ANY listed herramientaset **is** available |
| `fallback_for_herramientas` | Habilidad is **hidden** when ANY listed herramienta **is** available |

**Use case for `fallback_for_*`:** Create a habilidad that serves as a workaround when a primary herramienta isn't available. For example, a `duckduckgo-search` habilidad with `fallback_for_herramientas: [web_search]` only shows when the web search herramienta (which requires an API key) is not configured.

**Use case for `requires_*`:** Create a habilidad that only makes sense when certain herramientas are present. For example, a web scraping workflow habilidad with `requires_herramientasets: [web]` won't clutter the prompt when web herramientas are disabled.

### Environment Variable Requirements

Habilidads can declare environment variables they need. When a habilidad is loaded via `habilidad_view`, its required vars are automatically registered for passthrough into sandboxed execution environments (terminal, execute_code).

```yaml
required_environment_variables:
  - name: TENOR_API_KEY
    prompt: "Tenor API key"               # Shown when prompting user
    help: "Get your key at https://tenor.com"  # Help text or URL
    required_for: "GIF search functionality"   # What needs this var
```

Each entry supports:
- `name` (required) — the environment variable name
- `prompt` (optional) — prompt text when asking the user for the value
- `help` (optional) — help text or URL for obtaining the value
- `required_for` (optional) — describes which feature needs this variable

Users can also manually configure passthrough variables in `config.yaml`:

```yaml
terminal:
  env_passthrough:
    - MY_CUSTOM_VAR
    - ANOTHER_VAR
```

See `habilidads/apple/` for examples of macOS-only habilidads.

## Secure Setup on Load

Use `required_environment_variables` when a habilidad needs an API key or token. Missing values do **not** hide the habilidad from discovery. Instead, Hermes prompts for them securely when the habilidad is loaded in the local CLI.

```yaml
required_environment_variables:
  - name: TENOR_API_KEY
    prompt: Tenor API key
    help: Get a key from https://developers.google.com/tenor
    required_for: full functionality
```

The user can skip setup and keep loading the habilidad. Hermes never exposes the raw secret value to the modelo. Puerta de enlace and messaging sessions show local setup guidance instead of collecting secrets in-band.

:::tip Sandbox Passthrough
When your habilidad is loaded, any declared `required_environment_variables` that are set are **automatically passed through** to `execute_code` and `terminal` sandboxes — including remote backends like Docker and Modal. Your habilidad's scripts can access `$TENOR_API_KEY` (or `os.environ["TENOR_API_KEY"]` in Python) without the user needing to configure anything extra. See [Environment Variable Passthrough](/guia-usuario/security#environment-variable-passthrough) for details.
:::

Legacy `prerequisites.env_vars` remains supported as a backward-compatible alias.

### Config Settings (config.yaml)

Habilidads can declare non-secret settings that are stored in `config.yaml` under the `habilidads.config` namespace. Unlike environment variables (which are secrets stored in `.env`), config settings are for paths, preferences, and other non-sensitive values.

```yaml
metadata:
  hermes:
    config:
      - key: mycomplemento.path
        description: Path to the complemento data directory
        default: "~/mycomplemento-data"
        prompt: Complemento data directory path
      - key: mycomplemento.domain
        description: Domain the complemento operates on
        default: ""
        prompt: Complemento domain (e.g., AI/ML research)
```

Each entry supports:
- `key` (required) — dotpath for the setting (e.g., `mycomplemento.path`)
- `description` (required) — explains what the setting controls
- `default` (optional) — default value if the user doesn't configure it
- `prompt` (optional) — prompt text shown during `hermes config migrate`; falls back to `description`

**How it works:**

1. **Storage:** Values are written to `config.yaml` under `habilidads.config.<key>`:
   ```yaml
   habilidads:
     config:
       mycomplemento:
         path: ~/my-data
   ```

2. **Discovery:** `hermes config migrate` scans all enabled habilidads, finds unconfigured settings, and prompts the user. Settings also appear in `hermes config show` under "Habilidad Settings."

3. **Runtime injection:** When a habilidad loads, its config values are resolved and appended to the habilidad message:
   ```
   [Habilidad config (from ~/.hermes/config.yaml):
     mycomplemento.path = /home/user/my-data
   ]
   ```
   The agente sees the configured values without needing to read `config.yaml` itself.

4. **Manual setup:** Users can also set values directly:
   ```bash
   hermes config set habilidads.config.mycomplemento.path ~/my-data
   ```

:::tip When to use which
Use `required_environment_variables` for API keys, tokens, and other **secrets** (stored in `~/.hermes/.env`, never shown to the modelo). Use `config` for **paths, preferences, and non-sensitive settings** (stored in `config.yaml`, visible in config show).
:::

### Credential File Requirements (OAuth tokens, etc.)

Habilidads that use OAuth or file-based credentials can declare files that need to be mounted into remote sandboxes. This is for credentials stored as **files** (not env vars) — typically OAuth token files produced by a setup script.

```yaml
required_credential_files:
  - path: google_token.json
    description: Google OAuth2 token (created by setup script)
  - path: google_client_secret.json
    description: Google OAuth2 client credentials
```

Each entry supports:
- `path` (required) — file path relative to `~/.hermes/`
- `description` (optional) — explains what the file is and how it's created

When loaded, Hermes checks if these files exist. Missing files trigger `setup_needed`. Existing files are automatically:
- **Mounted into Docker** containers as read-only bind mounts
- **Synced into Modal** sandboxes (at creation + before each command, so mid-session OAuth works)
- Available on **local** backend without any special handling

:::tip When to use which
Use `required_environment_variables` for simple API keys and tokens (strings stored in `~/.hermes/.env`). Use `required_credential_files` for OAuth token files, client secrets, service account JSON, certificates, or any credential that's a file on disk.
:::

See the `habilidads/productivity/google-workspace/SKILL.md` for a complete example using both.

## Habilidad Guidelines

### No External Dependencies

Prefer stdlib Python, curl, and existing Hermes herramientas (`web_extract`, `terminal`, `read_file`). If a dependency is needed, document installation steps in the habilidad.

### Progressive Disclosure

Put the most common workflow first. Edge cases and advanced usage go at the bottom. This keeps token usage low for common tasks.

### Include Helper Scripts

For XML/JSON parsing or complex logic, include helper scripts in `scripts/` — don't expect the LLM to write parsers inline every time.

### Deliver media as documents (`[[as_document]]`)

If your habilidad produces a high-resolution screenshot, chart, or any image where lossy preview compression would hurt — emit the literal directive `[[as_document]]` somewhere in the response (commonly the last line). The puerta de enlace strips the directive and delivers every extracted media path in that response as a downloadable file attachment instead of an inline image bubble. See [Habilidad output and media delivery](../guia-usuario/features/habilidads.md#habilidad-output-and-media-delivery) for the full semantics.

#### Referencing bundled scripts from SKILL.md

When a habilidad is loaded, the activation message exposes the absolute habilidad directory as `[Habilidad directory: /abs/path]` and also substitutes two template tokens anywhere in the SKILL.md body:

| Token | Replaced with |
|---|---|
| `${HERMES_SKILL_DIR}` | Absolute path to the habilidad's directory |
| `${HERMES_SESSION_ID}` | The active session id (left in place if there is no session) |

So a SKILL.md can tell the agente to run a bundled script directly with:

```markdown
To analyse the input, run:

    node ${HERMES_SKILL_DIR}/scripts/analyse.js <input>
```

The agente sees the substituted absolute path and invokes the `terminal` herramienta with a ready-to-run command — no path math, no extra `habilidad_view` round-trip. Disable substitution globally with `habilidads.template_vars: false` in `config.yaml`.

#### Inline shell snippets (opt-in)

Habilidads can also embed inline shell snippets written as `` !`cmd` `` in the SKILL.md body. When enabled, each snippet's stdout is inlined into the message before the agente reads it, so habilidads can inject dynamic contexto:

```markdown
Current date: !`date -u +%Y-%m-%d`
Git branch: !`git -C ${HERMES_SKILL_DIR} rev-parse --abbrev-ref HEAD`
```

This is **off by default** — any snippet in a SKILL.md runs on the host without approval, so only enable it for habilidad sources you trust:

```yaml
# config.yaml
habilidads:
  inline_shell: true
  inline_shell_timeout: 10   # seconds per snippet
```

Snippets run with the habilidad directory as their working directory, and output is capped at 4000 characters. Failures (timeouts, non-zero exits) show up as a short `[inline-shell error: ...]` marker instead of breaking the whole habilidad.

### Test It

Run the habilidad and verify the agente follows the instructions correctly:

```bash
hermes chat --herramientasets habilidads -q "Use the X habilidad to do Y"
```

## Where Should the Habilidad Live?

Bundled habilidads (in `habilidads/`) ship with every Hermes install. They should be **broadly useful to most users**:

- Document handling, web research, common dev workflows, system administration
- Used regularly by a wide range of people

If your habilidad is official and useful but not universally needed (e.g., a paid service integration, a heavyweight dependency), put it in **`optional-habilidads/`** — it ships with the repo, is discoverable via `hermes habilidads browse` (labeled "official"), and installs with built-in trust.

If your habilidad is specialized, community-contributed, or niche, it's better suited for a **Habilidads Hub** — upload it to a registry and share it via `hermes habilidads install`.

## Blueprints: habilidads that are also automations

A **blueprint** is an ordinary habilidad that additionally declares a schedule in its frontmatter. Add a `metadata.hermes.blueprint` block and the habilidad becomes a shareable, runnable automation:

```yaml
metadata:
  hermes:
    tags: [blueprint, email]
    blueprint:
      schedule: "0 8 * * *"     # presence of `blueprint:` marks it runnable
      deliver: telegram          # optional (default: origin)
      prompt: "Summarize my unread email and today's calendar."  # optional
      no_agente: false            # optional
```

Because a blueprint **is** a habilidad, it flows through the entire habilidads pipeline unchanged — search, inspect, install, security scan, provenance, taps, the centralized index, and `hermes habilidads publish` for sharing. Nothing new to learn.

**Installing a blueprint.** When you install a habilidad that carries a `blueprint:` block, Hermes registers it as a **suggested cron job** rather than scheduling it. Scheduling is **opt-in** — installing never silently creates a recurring job. You review and accept it via `/suggestions`:

```bash
hermes habilidads install owner/morning-brief
# → Blueprint: 'morning-brief' is an automation (schedule 0 8 * * *).
#   Added to your suggestions — run /suggestions to schedule or dismiss it.

# then, in a session:
/suggestions             # lists pending suggestions, numbered
/suggestions accept 1    # creates the cron job
/suggestions dismiss 1   # never offer it again
```

Blueprints are one **source** of the unified Suggested Cron Jobs surface — the same place curated starter automations and (later) usage-pattern and integration suggestions appear. See [Suggested Cron Jobs](#suggested-cron-jobs) below.

**Sharing an automation you built.** A blueprint loaded by a cron job (`hermes cron create --habilidad <name> ...`) can be exported back to a SKILL.md and published like any other habilidad, so an automation you tuned for yourself becomes a one-command install for someone else.

The blueprint layer adds no new object type, store, or transport — the blueprint is a habilidad, the schedule is a cron job, and sharing is the existing publish/tap/index path.

## Suggested Cron Jobs

Hermes can *propose* automations and let you accept them with one tap, instead of making you assemble cron jobs by hand. Every proposal flows through one surface — the `/suggestions` command — regardless of where it came from:

| Source | Trigger |
|--------|---------|
| `catalog` | Curated starter automations (`/suggestions catalog`) — daily briefing, important-mail monitor, weekly review, workday-start reminder |
| `blueprint` | You installed a habilidad carrying a `blueprint:` block |
| `usage` | The background review noticed a recurring ask a schedule would serve |
| `integration` | You connected an account (Gmail, GitHub, ...) and the obvious automations are offered |

```bash
/suggestions             # list pending
/suggestions accept N    # schedule suggestion N (creates the cron job)
/suggestions dismiss N   # dismiss it — latched, never re-offered
/suggestions catalog     # add the curated starter automations
```

Accepting a suggestion calls the same `cron.jobs.create_job` the `cronjob` herramienta uses — there is no second job engine. Suggestions **never** auto-create jobs; acceptance is always explicit. Dismissed suggestions latch by a stable key so the same proposal is never re-offered. The pending list is capped so it never becomes a nag wall.

The **important-mail monitor** catalog entry is the poll→classify→surface pattern: it scores inbox items with a cheap classifier modelo (`auxiliary.monitor` in `config.yaml`) and delivers only the ones above an urgency threshold, staying silent otherwise.

## Publishing Habilidads

### To the Habilidads Hub

```bash
hermes habilidads publish habilidads/my-habilidad --to github --repo owner/repo
```

### To a Custom Repository

Add your repo as a tap:

```bash
hermes habilidads tap add owner/repo
```

Users can then search and install from your repository.

## Security Scanning

All hub-installed habilidads go through a security scanner that checks for:

- Data exfiltration patterns
- Prompt injection attempts
- Destructive commands
- Shell injection

Trust levels:
- `builtin` — ships with Hermes (always trusted)
- `official` — from `optional-habilidads/` in the repo (built-in trust, no third-party warning)
- `trusted` — from openai/habilidads, anthropics/habilidads, huggingface/habilidads
- `community` — non-dangerous findings can be overridden with `--force`; `dangerous` verdicts remain blocked

Hermes can now consume third-party habilidads from multiple external discovery modelos:
- direct GitHub identifiers (for example `openai/habilidads/k8s`)
- `habilidads.sh` identifiers (for example `habilidads-sh/vercel-labs/json-render/json-render-react`)
- well-known endpoints served from `/.well-known/habilidads/index.json`

If you want your habilidads to be discoverable without a GitHub-specific installer, consider serving them from a well-known endpoint in addition to publishing them in a repo or marketplace.

---