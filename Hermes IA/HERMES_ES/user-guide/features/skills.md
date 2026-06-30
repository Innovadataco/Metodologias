<!-- source: website/docs/guia-usuario/features/habilidads.md -->
# Habilidads System

# Habilidads System

Habilidads are on-demand knowledge documents the agente can load when needed. They follow a **progressive disclosure** pattern to minimize token usage and are compatible with the [agentehabilidads.io](https://agentehabilidads.io/specification) open standard.

All habilidads live in **`~/.hermes/habilidads/`** — the primary directory and source of truth. On fresh install, bundled habilidads are copied from the repo. Hub-installed and agente-created habilidads also go here. The agente can modify or delete any habilidad.

You can also point Hermes at **external habilidad directories** — additional folders scanned alongside the local one. See [External Habilidad Directories](#external-habilidad-directories) below.

See also:

- [Bundled Habilidads Catalog](/reference/habilidads-catalog)
- [Official Optional Habilidads Catalog](/reference/optional-habilidads-catalog)

## Starting with a blank slate

By default every perfil is seeded with the bundled habilidad catalog, and each `hermes update` adds any newly bundled habilidads. If you want a perfil with **no bundled habilidads** — and that stays empty across updates — you have two paths:

**At install time** (applies to the default `~/.hermes` perfil):

```bash
curl -fsSL https://hermes-agente.nousresearch.com/install.sh | bash -s -- --no-habilidads
```

**At perfil-create time** (named perfils):

```bash
hermes perfil create research --no-habilidads
```

**On an already-installed perfil** (default or named), toggle it at runtime:

```bash
hermes habilidads opt-out            # stop future seeding — nothing on disk is touched
hermes habilidads opt-out --remove   # also delete UNMODIFIED bundled habilidads (confirms first)
hermes habilidads opt-in --sync      # undo: remove the marker and re-seed now
```

All three paths write a `.no-bundled-habilidads` marker into the perfil directory. While the marker is present, the installer, `hermes update`, and any habilidad sync all skip bundled-habilidad seeding for that perfil. Delete the marker (or run `hermes habilidads opt-in`) to re-enable.

:::note Safe by default
`hermes habilidads opt-out` only stops *future* seeding — it never deletes anything already on disk. The optional `--remove` flag deletes bundled habilidads **only** when they are unmodified (byte-identical to the version Hermes installed). Habilidads you have edited, habilidads installed from the hub, and habilidads you wrote yourself are always kept.
:::

## Using Habilidads

Every installed habilidad is automatically available as a slash command:

```bash
# In the CLI or any messaging platform:
/gif-search funny cats
/axolotl help me fine-tune Llama 3 on my dataset
/github-pr-workflow create a PR for the auth refactor
/plan design a rollout for migrating our auth proveedor

# Just the habilidad name loads it and lets the agente ask what you need:
/excalidraw
```

The bundled `plan` habilidad is a good example. Running `/plan [request]` loads the habilidad's instructions, telling Hermes to inspect contexto if needed, write a markdown implementation plan instead of executing the task, and save the result under `.hermes/plans/` relative to the active workspace/backend working directory.

You can also interact with habilidads through natural conversation:

```bash
hermes chat --herramientasets habilidads -q "What habilidads do you have?"
hermes chat --herramientasets habilidads -q "Show me the axolotl habilidad"
```

## Learning a habilidad from sources (`/learn`)

`/learn` is the fast way to turn something you already know — or a pile of
reference material — into a reusable habilidad, without hand-writing the
`SKILL.md`. It is open-ended: point it at *anything you can describe* and the
agente gathers the material with the herramientas it already has, then authors a habilidad
that follows the [house authoring standards](#habilidadmd-format) (≤60-char
description, the standard section order, Hermes-herramienta framing, no invented
commands).

```bash
# A local SDK or doc directory — read with read_file / search_files
/learn the REST client in ~/projects/acme-sdk, focus on auth + pagination

# An online doc page — fetched with web_extract
/learn https://docs.example.com/api/quickstart

# The workflow you just walked the agente through in this conversation
/learn how I just deployed the staging server

# Pasted notes / a described procedure
/learn filing an expense: open the portal, New > Expense, attach the receipt, submit
```

Because the live agente does the sourcing, `/learn` works the same in the CLI,
the messaging puerta de enlace, the TUI, and the panel — and on any terminal backend
(local, Docker, remote), since there is no separate ingestion engine. In the
**panel**, the Habilidads page has a **Learn a habilidad** button that opens a panel
with a directory field, a URL field, and an open-ended text box; it composes a
`/learn` request and runs it in chat.

There is no modelo-herramienta footprint: `/learn` builds a standards-guided prompt and
hands it to the agente as a normal turn. The agente saves the result with the
`habilidad_manage` herramienta, so the [write-approval gate](#gating-agente-habilidad-writes-habilidadswrite_approval)
applies if you have it on.

## Progressive Disclosure

Habilidads use a token-efficient loading pattern:

```
Level 0: habilidads_list()           → [{name, description, category}, ...]   (~3k tokens)
Level 1: habilidad_view(name)        → Full content + metadata       (varies)
Level 2: habilidad_view(name, path)  → Specific reference file       (varies)
```

The agente only loads the full habilidad content when it actually needs it.

## SKILL.md Format

```markdown
---
name: my-habilidad
description: Brief description of what this habilidad does
version: 1.0.0
platforms: [macos, linux]     # Optional — restrict to specific OS platforms
metadata:
  hermes:
    tags: [python, automation]
    category: devops
    fallback_for_herramientasets: [web]    # Optional — conditional activation (see below)
    requires_herramientasets: [terminal]   # Optional — conditional activation (see below)
    config:                          # Optional — config.yaml settings
      - key: my.setting
        description: "What this controls"
        default: "value"
        prompt: "Prompt for setup"
---

# Habilidad Title

## When to Use
Trigger conditions for this habilidad.

## Procedure
1. Step one
2. Step two

## Pitfalls
- Known failure modes and fixes

## Verification
How to confirm it worked.
```

### Platform-Specific Habilidads

Habilidads can restrict themselves to specific operating systems using the `platforms` field:

| Value | Matches |
|-------|---------|
| `macos` | macOS (Darwin) |
| `linux` | Linux |
| `windows` | Windows |

```yaml
platforms: [macos]            # macOS only (e.g., iMessage, Apple Reminders, FindMy)
platforms: [macos, linux]     # macOS and Linux
```

When set, the habilidad is automatically hidden from the system prompt, `habilidads_list()`, and slash commands on incompatible platforms. If omitted, the habilidad loads on all platforms.

## Habilidad output and media delivery

When a habilidad response (or any agente response) includes a bare absolute path to a media file — for example `/home/user/screenshots/diagram.png` — the puerta de enlace auto-detects it, strips it from the visible text, and delivers the file natively to the user's chat (Telegram photo, Discord attachment, etc.) instead of leaving the raw path in the message.

For audio specifically, the `[[audio_as_voice]]` directive promotes audio files to native voice-message bubbles on platforms that support them (Telegram, WhatsApp).

### Forcing document-style delivery: `[[as_document]]`

Sometimes you want the **opposite** of inline preview: you want the file delivered as a downloadable attachment, not a re-compressed image bubble. The classic example is a high-resolution screenshot or chart — Telegram's `sendPhoto` recompresses it to ~200 KB at 1280 px, destroying readability. A 1-2 MB PNG sent via `sendDocument` keeps the original bytes intact.

If a response (or any text inside it — typically the last line) contains the literal directive `[[as_document]]`, every media path extracted from that response is delivered as a document/file attachment rather than an image bubble:

```
Here is your rendered chart:

/home/user/.hermes/cache/chart-q4-2025.png

[[as_document]]
```

The directive is stripped before delivery, so users never see it. Granularity is intentionally all-or-nothing per response: emit `[[as_document]]` once and every image path in the same response is delivered as a document. This mirrors the scope of `[[audio_as_voice]]`.

Use it from a habilidad when:

- You produce screenshots or charts the user needs as files (for editing in another herramienta, archiving, sharing intact).
- The default lossy preview would obscure detail (small text, pixel-accurate diagrams, color-sensitive renders).

Platforms without a separate document path (e.g. SMS) fall back to whatever attachment mechanism they have.

### Conditional Activation (Fallback Habilidads)

Habilidads can automatically show or hide themselves based on which herramientas are available in the current session. This is most useful for **fallback habilidads** — free or local alternatives that should only appear when a premium herramienta is unavailable.

```yaml
metadata:
  hermes:
    fallback_for_herramientasets: [web]      # Show ONLY when these herramientasets are unavailable
    requires_herramientasets: [terminal]     # Show ONLY when these herramientasets are available
    fallback_for_herramientas: [web_search]  # Show ONLY when these specific herramientas are unavailable
    requires_herramientas: [terminal]        # Show ONLY when these specific herramientas are available
```

| Field | Behavior |
|-------|----------|
| `fallback_for_herramientasets` | Habilidad is **hidden** when the listed herramientasets are available. Shown when they're missing. |
| `fallback_for_herramientas` | Same, but checks individual herramientas instead of herramientasets. |
| `requires_herramientasets` | Habilidad is **hidden** when the listed herramientasets are unavailable. Shown when they're present. |
| `requires_herramientas` | Same, but checks individual herramientas. |

**Example:** The built-in `duckduckgo-search` habilidad uses `fallback_for_herramientasets: [web]`. When you have `FIRECRAWL_API_KEY` set, the web herramientaset is available and the agente uses `web_search` — the DuckDuckGo habilidad stays hidden. If the API key is missing, the web herramientaset is unavailable and the DuckDuckGo habilidad automatically appears as a fallback.

Habilidads without any conditional fields behave exactly as before — they're always shown.

## Secure Setup on Load

Habilidads can declare required environment variables without disappearing from discovery:

```yaml
required_environment_variables:
  - name: TENOR_API_KEY
    prompt: Tenor API key
    help: Get a key from https://developers.google.com/tenor
    required_for: full functionality
```

When a missing value is encountered, Hermes asks for it securely only when the habilidad is actually loaded in the local CLI. You can skip setup and keep using the habilidad. Messaging surfaces never ask for secrets in chat — they tell you to use `hermes setup` or `~/.hermes/.env` locally instead.

Once set, declared env vars are **automatically passed through** to `execute_code` and `terminal` sandboxes — the habilidad's scripts can use `$TENOR_API_KEY` directly. For non-habilidad env vars, use the `terminal.env_passthrough` config option. See [Environment Variable Passthrough](/guia-usuario/security#environment-variable-passthrough) for details.

### Habilidad Config Settings

Habilidads can also declare non-secret config settings (paths, preferences) stored in `config.yaml`:

```yaml
metadata:
  hermes:
    config:
      - key: mycomplemento.path
        description: Path to the complemento data directory
        default: "~/mycomplemento-data"
        prompt: Complemento data directory path
```

Settings are stored under `habilidads.config` in your config.yaml. `hermes config migrate` prompts for unconfigured settings, and `hermes config show` displays them. When a habilidad loads, its resolved config values are injected into the contexto so the agente knows the configured values automatically.

See [Habilidad Settings](/guia-usuario/configuración#habilidad-settings) and [Creating Habilidads — Config Settings](/guia-desarrollador/creating-habilidads#config-settings-configyaml) for details.

## Habilidad Directory Structure

```text
~/.hermes/habilidads/                  # Single source of truth
├── mlops/                         # Category directory
│   ├── axolotl/
│   │   ├── SKILL.md               # Main instructions (required)
│   │   ├── references/            # Additional docs
│   │   ├── templates/             # Output formats
│   │   ├── scripts/               # Helper scripts callable from the habilidad
│   │   └── assets/                # Supplementary files
│   └── vllm/
│       └── SKILL.md
├── devops/
│   └── deploy-k8s/                # Agente-created habilidad
│       ├── SKILL.md
│       └── references/
├── .hub/                          # Habilidads Hub state
│   ├── lock.json
│   ├── quarantine/
│   └── audit.log
└── .bundled_manifest              # Tracks seeded bundled habilidads
```

## External Habilidad Directories

If you maintain habilidads outside of Hermes — for example, a shared `~/.agentes/habilidads/` directory used by multiple AI herramientas — you can tell Hermes to scan those directories too.

Add `external_dirs` under the `habilidads` section in `~/.hermes/config.yaml`:

```yaml
habilidads:
  external_dirs:
    - ~/.agentes/habilidads
    - /home/shared/team-habilidads
    - ${SKILLS_REPO}/habilidads
```

Paths support `~` expansion and `${VAR}` environment variable substitution.

### How it works

- **Create locally, update in place**: New agente-created habilidads are written to `~/.hermes/habilidads/`. Existing habilidads are modified where they are found, including habilidads under `external_dirs`, when the agente uses `habilidad_manage` actions such as `patch`, `edit`, `write_file`, `remove_file`, or `delete`.
- **External dirs are not a write-protection boundary**: If an external habilidad directory is writable by the Hermes process, agente-managed habilidad updates can change files in that directory. Use filesystem permissions or a separate perfil/herramientaset setup if shared external habilidads must stay read-only.
- **Local precedence**: If the same habilidad name exists in both the local dir and an external dir, the local version wins.
- **Full integration**: External habilidads appear in the system prompt index, `habilidads_list`, `habilidad_view`, and as `/habilidad-name` slash commands — no different from local habilidads.
- **Non-existent paths are silently skipped**: If a configured directory doesn't exist, Hermes ignores it without errors. Useful for optional shared directories that may not be present on every machine.

### Example

```text
~/.hermes/habilidads/               # Local (primary, read-write)
├── devops/deploy-k8s/
│   └── SKILL.md
└── mlops/axolotl/
    └── SKILL.md

~/.agentes/habilidads/               # External (shared, mutable if writable)
├── my-custom-workflow/
│   └── SKILL.md
└── team-conventions/
    └── SKILL.md
```

All four habilidads appear in your habilidad index. If you create a new habilidad called `my-custom-workflow` locally, it shadows the external version.

## Habilidad Bundles

Habilidad bundles are tiny YAML files that group several habilidads under a single slash command. When you run `/<bundle-name>`, every habilidad listed in the bundle loads at once — useful when a particular task always benefits from the same set of habilidads together.

### Quick example

```bash
# Create a bundle for backend feature work
hermes bundles create backend-dev \
  --habilidad github-code-review \
  --habilidad test-driven-development \
  --habilidad github-pr-workflow \
  -d "Backend feature work — review, test, PR workflow"
```

Then in the CLI or any puerta de enlace platform:

```
/backend-dev refactor the auth middleware
```

The agente receives all three habilidads loaded into one user message, with any text after the slash command attached as a user instruction.

### YAML schema

Bundles live in **`~/.hermes/habilidad-bundles/<slug>.yaml`** and look like this:

```yaml
name: backend-dev
description: Backend feature work — review, test, PR workflow.
habilidads:
  - github-code-review
  - test-driven-development
  - github-pr-workflow
instruction: |
  Always start by writing failing tests, then implement.
  Open the PR through the standard workflow with co-author tags.
```

Fields:
- `name` (optional — defaults to the filename stem) — the bundle's display name. Normalized to a hyphen slug for the slash command (`Backend Dev` → `/backend-dev`).
- `description` (optional) — short text shown in `/bundles` and `hermes bundles list`.
- `habilidads` (required, non-empty list) — habilidad names or paths relative to your habilidads directory. Use the same identifier you'd pass to `/<habilidad-name>`.
- `instruction` (optional) — extra guidance prepended to the loaded habilidad content. Useful for codifying "how we always use these together."

### Managing bundles

```bash
# List all installed bundles
hermes bundles list

# Inspect one bundle
hermes bundles show backend-dev

# Create a bundle interactively (omit --habilidad flags to enter them one per line)
hermes bundles create research

# Overwrite an existing bundle
hermes bundles create backend-dev --habilidad ... --force

# Delete a bundle
hermes bundles delete backend-dev

# Re-scan ~/.hermes/habilidad-bundles/ and report changes
hermes bundles reload
```

From inside a chat session, `/bundles` lists every installed bundle and its habilidads.

### Behavior

- **Bundles take precedence over individual habilidads** when slugs collide. If you name a bundle `research` and you also have a habilidad called `research`, `/research` invokes the bundle. This is intentional — you opted into the bundle by naming it.
- **Missing habilidads are skipped, not fatal.** If a bundle lists `habilidad-foo` and you haven't installed it, the bundle still loads the habilidads that do resolve, and the agente gets a note listing what was skipped.
- **Bundles work in every surface** — interactive CLI, TUI, panel chat, and every puerta de enlace platform (Telegram, Discord, Slack, …) — because dispatch is centralized in the same place as individual habilidad commands.
- **Bundles do not invalidate the prompt cache.** They generate a fresh user message at invocation time, the same way `/<habilidad-name>` does — no system prompt mutation.

### When bundles beat installing each habilidad manually

Use a bundle when:
- You always pair the same habilidads for a recurring task (`/backend-dev`, `/release-prep`, `/incident-response`).
- You want a one-character-shorter mental modelo than typing several `/habilidad` invocations in a row.
- You want to ship a team-wide "task perfil" by checking the bundle YAML into a shared dotfiles repo and symlinking it into `~/.hermes/habilidad-bundles/`.

A bundle is just a YAML alias — it doesn't install habilidads for you. The habilidads themselves must already be present (in `~/.hermes/habilidads/` or an external habilidad directory). Otherwise the bundle invocation just skips the missing ones.

## Agente-Managed Habilidads (habilidad_manage herramienta)

The agente can create, update, and delete its own habilidads via the `habilidad_manage` herramienta. This is the agente's **procedural memoria** — when it figures out a non-trivial workflow, it saves the approach as a habilidad for future reuse.

Habilidads and memoria work together in the self-improvement loop: memoria stores
small durable facts that should always be in contexto, while habilidads store longer
procedures that should load only when relevant. The background review can
suggest or stage habilidad changes after a session, but the write-approval gate
below lets you require human review before those changes land.

### When the Agente Creates Habilidads

- After completing a complex task (5+ herramienta calls) successfully
- When it hit errors or dead ends and found the working path
- When the user corrected its approach
- When it discovered a non-trivial workflow

### Actions

| Action | Use for | Key params |
|--------|---------|------------|
| `create` | New habilidad from scratch | `name`, `content` (full SKILL.md), optional `category` |
| `patch` | Targeted fixes (preferred) | `name`, `old_string`, `new_string` |
| `edit` | Major structural rewrites | `name`, `content` (full SKILL.md replacement) |
| `delete` | Remove a habilidad entirely | `name` |
| `write_file` | Add/update supporting files | `name`, `file_path`, `file_content` |
| `remove_file` | Remove a supporting file | `name`, `file_path` |

:::tip
The `patch` action is preferred for updates — it's more token-efficient than `edit` because only the changed text appears in the herramienta call.
:::

### Gating agente habilidad writes (`habilidads.write_approval`)

By default the agente writes habilidads freely — including from the [background
self-improvement review](/guia-usuario/features/memoria#controlling-memoria-writes-write_approval)
that runs after a turn. If you'd rather approve every habilidad write first
(small modelos that misjudge what they learned, secure environments, or just
wanting eyes on the self-improvement loop), turn on the write-approval gate:

```yaml
habilidads:
  write_approval: false     # false = write freely (default) | true = require approval
```

When `write_approval: true`, every `habilidad_manage` write (create / edit /
patch / delete / write_file / remove_file) is **staged** instead of committed —
a SKILL.md is too large to review inline, so staging applies regardless of
whether the write came from a foreground turn or the background review.
Staged writes survive restarts under `~/.hermes/pending/habilidads/` and are
reviewed with the same familiar approve/deny flow as dangerous commands:

```
/habilidads pending             # list staged habilidad writes + a one-line gist each
/habilidads diff <id>           # full unified diff (best viewed in CLI or panel)
/habilidads approve <id>        # apply it (or 'all')
/habilidads reject <id>         # drop it (or 'all')
/habilidads approval on         # turn the gate on (or 'off') and persist it
```

The review surface works in the interactive CLI and on messaging platforms
(diff output is truncated for chat bubbles — read the full diff on the CLI or
in the pending JSON file). Memoria writes have the same gate under
`memoria.write_approval` — see [Controlling memoria writes](/guia-usuario/features/memoria#controlling-memoria-writes-write_approval).

> The separate `habilidads.guard_agente_created` setting is a content scanner
> (dangerous-pattern heuristics), not an approval gate — the two are
> independent. See [Guard on agente-created habilidad writes](/guia-usuario/configuración#guard-on-agente-created-habilidad-writes).

## Habilidads Hub

Browse, search, install, and manage habilidads from online registries, `habilidads.sh`, direct well-known habilidad endpoints, and official optional habilidads.

### Common commands

```bash
hermes habilidads browse                              # Browse all hub habilidads (official first)
hermes habilidads browse --source official            # Browse only official optional habilidads
hermes habilidads search kubernetes                   # Search all sources
hermes habilidads search react --source habilidads-sh     # Search the habilidads.sh directory
hermes habilidads search https://mintlify.com/docs --source well-known
hermes habilidads inspect openai/habilidads/k8s           # Preview before installing
hermes habilidads install openai/habilidads/k8s           # Install with security scan
hermes habilidads install official/security/1password
hermes habilidads install habilidads-sh/vercel-labs/json-render/json-render-react --force
hermes habilidads install well-known:https://mintlify.com/docs/.well-known/habilidads/mintlify
hermes habilidads install https://sharethis.chat/SKILL.md              # Direct URL (single-file SKILL.md)
hermes habilidads install https://example.com/SKILL.md --name my-habilidad # Override name when frontmatter has none
hermes habilidads list --source hub                   # List hub-installed habilidads
hermes habilidads check                               # Check installed hub habilidads for upstream updates
hermes habilidads update                              # Reinstall hub habilidads with upstream changes when needed
hermes habilidads audit                               # Re-scan all hub habilidads for security
hermes habilidads uninstall k8s                       # Remove a hub habilidad
hermes habilidads reset google-workspace              # Un-stick a bundled habilidad from "user-modified" (see below)
hermes habilidads reset google-workspace --restore    # Also restore the bundled version, deleting your local edits
hermes habilidads publish habilidads/my-habilidad --to github --repo owner/repo
hermes habilidads snapshot export setup.json          # Export habilidad config
hermes habilidads tap add myorg/habilidads-repo           # Add a custom GitHub source
```

### Supported hub sources

| Source | Example | Notes |
|--------|---------|-------|
| `official` | `official/security/1password` | Optional habilidads shipped with Hermes. |
| `habilidads-sh` | `habilidads-sh/vercel-labs/agente-habilidads/vercel-react-best-practices` | Searchable via `hermes habilidads search <query> --source habilidads-sh`. Hermes resolves alias-style habilidads when the habilidads.sh slug differs from the repo folder. |
| `well-known` | `well-known:https://mintlify.com/docs/.well-known/habilidads/mintlify` | Habilidads served directly from `/.well-known/habilidads/index.json` on a website. Search using the site or docs URL. |
| `url` | `https://sharethis.chat/SKILL.md` | Direct HTTP(S) URL to a single-file `SKILL.md`. Name resolution: frontmatter → URL slug → interactive prompt → `--name` flag. |
| `github` | `openai/habilidads/k8s` | Direct GitHub repo/path installs and custom taps. |
| `clawhub`, `lobehub`, `browse-sh` | Source-specific identifiers | Community or marketplace integrations. |

### Integrated hubs and registries

Hermes currently integrates with these habilidads ecosystems and discovery sources:

#### 1. Official optional habilidads (`official`)

These are maintained in the Hermes repository itself and install with built-in trust.

- Catalog: [Official Optional Habilidads Catalog](../../reference/optional-habilidads-catalog)
- Source in repo: `optional-habilidads/`
- Example:

```bash
hermes habilidads browse --source official
hermes habilidads install official/security/1password
```

#### 2. habilidads.sh (`habilidads-sh`)

This is Vercel's public habilidads directory. Hermes can search it directly, inspect habilidad detail pages, resolve alias-style slugs, and install from the underlying source repo.

- Directory: [habilidads.sh](https://habilidads.sh/)
- CLI/herramientaing repo: [vercel-labs/habilidads](https://github.com/vercel-labs/habilidads)
- Official Vercel habilidads repo: [vercel-labs/agente-habilidads](https://github.com/vercel-labs/agente-habilidads)
- Example:

```bash
hermes habilidads search react --source habilidads-sh
hermes habilidads inspect habilidads-sh/vercel-labs/json-render/json-render-react
hermes habilidads install habilidads-sh/vercel-labs/json-render/json-render-react --force
```

#### 3. Well-known habilidad endpoints (`well-known`)

This is URL-based discovery from sites that publish `/.well-known/habilidads/index.json`. It is not a single centralized hub — it is a web discovery convention.

- Example live endpoint: [Mintlify docs habilidads index](https://mintlify.com/docs/.well-known/habilidads/index.json)
- Reference server implementation: [vercel-labs/habilidads-handler](https://github.com/vercel-labs/habilidads-handler)
- Example:

```bash
hermes habilidads search https://mintlify.com/docs --source well-known
hermes habilidads inspect well-known:https://mintlify.com/docs/.well-known/habilidads/mintlify
hermes habilidads install well-known:https://mintlify.com/docs/.well-known/habilidads/mintlify
```

#### 4. Direct GitHub habilidads (`github`)

Hermes can install directly from GitHub repositories and GitHub-based taps. This is useful when you already know the repo/path or want to add your own custom source repo.

Default taps (browsable without any setup):
- [openai/habilidads](https://github.com/openai/habilidads)
- [anthropics/habilidads](https://github.com/anthropics/habilidads)
- [huggingface/habilidads](https://github.com/huggingface/habilidads)
- [NVIDIA/habilidads](https://github.com/NVIDIA/habilidads) — NVIDIA-verified habilidads (signed `habilidad.oms.sig` + governance `habilidad-card.md`)
- [garrytan/gstack](https://github.com/garrytan/gstack)

- Example:

```bash
hermes habilidads install openai/habilidads/k8s
hermes habilidads tap add myorg/habilidads-repo
```

**Category groupings (`habilidads.sh.json`).** A GitHub tap may ship a
`habilidads.sh.json` file at its repo root following the
[habilidads.sh schema](https://habilidads.sh/schemas/habilidads.sh.schema.json). Its
`groupings` (each with a `title` and a list of habilidad names) are read at index
time and become the category labels shown in the
[Habilidads Hub](https://hermes-agente.nousresearch.com/docs) page — instead of a
tag-derived guess. This is generic: any tap that ships the file gets real
categorization, no Hermes-side changes required.

```json
{
  "$schema": "https://habilidads.sh/schemas/habilidads.sh.schema.json",
  "groupings": [
    { "title": "Inference AI", "habilidads": ["dynamo-recipe-runner", "dynamo-router-sla"] },
    { "title": "Decision Optimization", "habilidads": ["cuopt-developer", "cuopt-install"] }
  ]
}
```

#### 5. ClawHub (`clawhub`)

A third-party habilidads marketplace integrated as a community source.

- Site: [clawhub.ai](https://clawhub.ai/)
- Hermes source id: `clawhub`

#### 6. Claude marketplace-style repos (`claude-marketplace`)

Hermes supports marketplace repos that publish Claude-compatible complemento/marketplace manifests.

Known integrated sources include:
- [anthropics/habilidads](https://github.com/anthropics/habilidads)
- [aihabilidadstore/marketplace](https://github.com/aihabilidadstore/marketplace)

Hermes source id: `claude-marketplace`

#### 7. LobeHub (`lobehub`)

Hermes can search and convert agente entries from LobeHub's public catalog into installable Hermes habilidads.

- Site: [LobeHub](https://lobehub.com/)
- Public agentes index: [chat-agentes.lobehub.com](https://chat-agentes.lobehub.com/)
- Backing repo: [lobehub/lobe-chat-agentes](https://github.com/lobehub/lobe-chat-agentes)
- Hermes source id: `lobehub`

#### 8. browse.sh (`browse-sh`)

Hermes integrates with [browse.sh](https://browse.sh), Browserbase's catalog of 200+ site-specific browser-automation SKILL.md files (Airbnb, Amazon, arXiv, 12306.cn, Etsy, Xero, and many more). Each habilidad describes how to drive one website end-to-end and is suitable for use with Hermes' browser herramientas and any browser-automation habilidads you already have installed.

- Site: [browse.sh](https://browse.sh/)
- Catalog API: `https://browse.sh/api/habilidads`
- Hermes source id: `browse-sh`
- Trust level: `community`

```bash
hermes habilidads search airbnb --source browse-sh
hermes habilidads inspect browse-sh/airbnb.com/search-listings-ddgioa
hermes habilidads install browse-sh/airbnb.com/search-listings-ddgioa
```

Identifiers use the form `browse-sh/<hostname>/<task-id>` and match the slug exposed by the browse.sh catalog. Content is resolved through the per-habilidad detail endpoint (`/api/habilidads/<slug>` → `habilidadMdUrl`), not through the catalog's GitHub `sourceUrl`.

#### 9. Direct URL (`url`)

Install a single-file `SKILL.md` directly from any HTTP(S) URL — useful when an author hosts a habilidad on their own site (no hub listing, no GitHub path to type). Hermes fetches the URL, parses the YAML frontmatter, security-scans it, and installs.

- Hermes source id: `url`
- Identifier: the URL itself (no prefix needed)
- Scope: **single-file `SKILL.md`** only. Multi-file habilidads with `references/` or `scripts/` need a manifest and should be published via one of the other sources above.

```bash
hermes habilidads install https://sharethis.chat/SKILL.md
hermes habilidads install https://example.com/my-habilidad/SKILL.md --category productivity
```

Name resolution, in order:
1. `name:` field in the SKILL.md YAML frontmatter (recommended — every well-formed habilidad has one).
2. Parent directory name from the URL path (e.g. `.../my-habilidad/SKILL.md` → `my-habilidad`, or `.../my-habilidad.md` → `my-habilidad`), when it's a valid identifier (`^[a-z][a-z0-9_-]*$`).
3. Interactive prompt on a terminal with a TTY.
4. On non-interactive surfaces (the `/habilidads install` slash command inside the TUI, puerta de enlace platforms, scripts), a clean error pointing at the `--name` override.

```bash
# Frontmatter has no name and the URL slug is unhelpful — supply one:
hermes habilidads install https://example.com/SKILL.md --name sharethis-chat

# Or inside a chat session:
/habilidads install https://example.com/SKILL.md --name sharethis-chat
```

Trust level is always `community` — the same security scan runs as for every other source. The URL is stored as the install identifier, so `hermes habilidads update` re-fetches from the same URL automatically when you want to refresh.

### Security scanning and `--force`

All hub-installed habilidads go through a **security scanner** that checks for data exfiltration, prompt injection, destructive commands, supply-chain signals, and other threats.

`hermes habilidads inspect ...` now also surfaces upstream metadata when available:
- repo URL
- habilidads.sh detail page URL
- install command
- weekly installs
- upstream security audit statuses
- well-known index/endpoint URLs

Use `--force` when you have reviewed a third-party habilidad and want to override a non-dangerous policy block:

```bash
hermes habilidads install habilidads-sh/anthropics/habilidads/pdf --force
```

Important behavior:
- `--force` can override policy blocks for caution/warn-style findings.
- `--force` does **not** override a `dangerous` scan verdict.
- Official optional habilidads (`official/...`) are treated as built-in trust and do not show the third-party warning panel.

### Trust levels

| Level | Source | Policy |
|-------|--------|--------|
| `builtin` | Ships with Hermes | Always trusted |
| `official` | `optional-habilidads/` in the repo | Built-in trust, no third-party warning |
| `trusted` | Trusted registries/repos such as `openai/habilidads`, `anthropics/habilidads`, `huggingface/habilidads`, `NVIDIA/habilidads` | More permissive policy than community sources |
| `community` | Everything else (`habilidads.sh`, well-known endpoints, custom GitHub repos, most marketplaces) | Non-dangerous findings can be overridden with `--force`; `dangerous` verdicts stay blocked |

### Update lifecycle

The hub now tracks enough provenance to re-check upstream copies of installed habilidads:

```bash
hermes habilidads check          # Report which installed hub habilidads changed upstream
hermes habilidads update         # Reinstall only the habilidads with updates available
hermes habilidads update react   # Update one specific installed hub habilidad
```

This uses the stored source identifier plus the current upstream bundle content hash to detect drift.

:::tip GitHub rate limits
Habilidads hub operations use the GitHub API, which has a rate limit of 60 requests/hour for unauthenticated users. If you see rate-limit errors during install or search, set `GITHUB_TOKEN` in your `.env` file to increase the limit to 5,000 requests/hour. The error message includes an actionable hint when this happens.
:::

### Publishing a custom habilidad tap

If you want to share a curated set of habilidads — for your team, your org, or publicly — you can publish them as a **tap**: a GitHub repository other Hermes users add with `hermes habilidads tap add <owner/repo>`. No server, no registry sign-up, no release pipeline. Just a directory of `SKILL.md` files.

#### Repo layout

A tap is any GitHub repo (public or private — private needs `GITHUB_TOKEN`) laid out like this:

```
owner/repo
├── habilidads/                       # default path; configurable per-tap
│   ├── my-workflow/
│   │   ├── SKILL.md              # required
│   │   ├── references/           # optional supporting files
│   │   ├── templates/
│   │   └── scripts/
│   ├── another-habilidad/
│   │   └── SKILL.md
│   └── third-habilidad/
│       └── SKILL.md
└── README.md                     # optional but helpful
```

Rules:
- Each habilidad lives in its own directory under the tap's root path (default `habilidads/`).
- The directory name becomes the habilidad's install slug.
- Each habilidad directory must contain a `SKILL.md` with standard [SKILL.md frontmatter](#habilidadmd-format) (`name`, `description`, plus optional `metadata.hermes.tags`, `version`, `author`, `platforms`, `metadata.hermes.config`).
- Subdirectories like `references/`, `templates/`, `scripts/`, `assets/` are downloaded alongside `SKILL.md` at install time.
- Habilidads whose directory name starts with `.` or `_` are ignored.

Hermes discovers habilidads by listing every subdirectory of the tap path and probing each for `SKILL.md`.

#### Minimal tap example

```
my-org/hermes-habilidads
└── habilidads/
    └── deploy-runbook/
        └── SKILL.md
```

`habilidads/deploy-runbook/SKILL.md`:

```markdown
---
name: deploy-runbook
description: Our deployment runbook — services, rollback, Slack channels
version: 1.0.0
author: My Org Platform Team
metadata:
  hermes:
    tags: [deployment, runbook, internal]
---

# Deploy Runbook

Step 1: ...
```

After pushing that to GitHub, any Hermes user can subscribe and install:

```bash
hermes habilidads tap add my-org/hermes-habilidads
hermes habilidads search deploy
hermes habilidads install my-org/hermes-habilidads/deploy-runbook
```

#### Non-default paths

If your habilidads don't live under `habilidads/` (common when you're adding a `habilidads/` subtree to an existing project), edit the tap entry in `~/.hermes/.hub/taps.json`:

```json
{
  "taps": [
    {"repo": "my-org/platform-docs", "path": "internal/habilidads/"}
  ]
}
```

The `hermes habilidads tap add` CLI defaults new taps to `path: "habilidads/"`; edit the file directly if you need a different path. `hermes habilidads tap list` shows the effective path per tap.

#### Installing individual habilidads directly (without adding a tap)

Users can also install a single habilidad from any public GitHub repo without adding the whole repo as a tap:

```bash
hermes habilidads install owner/repo/habilidads/my-workflow
```

Useful when you want to share one habilidad without asking the user to subscribe to your whole registry.

#### Trust levels for taps

New taps are assigned `community` trust by default. Habilidads installed from them run through the standard security scan and show the third-party warning panel on first install. If your org or a widely-trusted source should get higher trust, add its repo to `TRUSTED_REPOS` in `herramientas/habilidads_hub.py` (requires a Hermes core PR).

#### Tap management

```bash
hermes habilidads tap list                                # show all configured taps
hermes habilidads tap add myorg/habilidads-repo               # add (default path: habilidads/)
hermes habilidads tap remove myorg/habilidads-repo            # remove
```

Inside a running session:

```
/habilidads tap list
/habilidads tap add myorg/habilidads-repo
/habilidads tap remove myorg/habilidads-repo
```

Taps are stored in `~/.hermes/.hub/taps.json` (created on demand).

## Bundled habilidad updates (`hermes habilidads reset`)

Hermes ships with a set of bundled habilidads in `habilidads/` inside the repo. On install and on every `hermes update`, a sync pass copies those into `~/.hermes/habilidads/` and records a manifest at `~/.hermes/habilidads/.bundled_manifest` mapping each habilidad name to the content hash at the time it was synced (the **origin hash**).

On each sync, Hermes recomputes the hash of your local copy and compares it to the origin hash:

- **Unchanged** → safe to pull upstream changes, copy the new bundled version in, record the new origin hash.
- **Changed** → treated as **user-modified** and skipped forever, so your edits never get stomped.

The protection is good, but it has one sharp edge. If you edit a bundled habilidad and then later want to abandon your changes and go back to the bundled version by just copy-pasting from `~/.hermes/hermes-agente/habilidads/`, the manifest still holds the *old* origin hash from whenever the last successful sync ran. Your fresh copy-paste contents (current bundled hash) won't match that stale origin hash, so sync keeps flagging it as user-modified.

`hermes habilidads reset` is the escape hatch:

```bash
# Safe: clears the manifest entry for this habilidad. Your current copy is preserved,
# but the next sync re-baselines against it so future updates work normally.
hermes habilidads reset google-workspace

# Full restore: also deletes your local copy and re-copies the current bundled
# version. Use this when you want the pristine upstream habilidad back.
hermes habilidads reset google-workspace --restore

# Non-interactive (e.g. in scripts or TUI mode) — skip the --restore confirmation.
hermes habilidads reset google-workspace --restore --yes
```

The same command works in chat as a slash command:

```text
/habilidads reset google-workspace
/habilidads reset google-workspace --restore
```

:::note Perfils
Each perfil has its own `.bundled_manifest` under its own `HERMES_HOME`, so `hermes -p coder habilidads reset <name>` only affects that perfil.
:::

### Slash commands (inside chat)

All the same commands work with `/habilidads`:

```text
/habilidads browse
/habilidads search react --source habilidads-sh
/habilidads search https://mintlify.com/docs --source well-known
/habilidads inspect habilidads-sh/vercel-labs/json-render/json-render-react
/habilidads install openai/habilidads/habilidad-creator --force
/habilidads check
/habilidads update
/habilidads reset google-workspace
/habilidads list
```

Official optional habilidads still use identifiers like `official/security/1password` and `official/migration/openclaw-migration`.

---