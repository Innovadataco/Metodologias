<!-- source: website/docs/guia-usuario/features/memoria.md -->
# Persistent Memoria

# Persistent Memoria

Hermes Agente has bounded, curated memoria that persists across sessions. This lets it remember your preferences, your projects, your environment, and things it has learned.

## How It Works

Two files make up the agente's memoria:

| File | Purpose | Char Limit |
|------|---------|------------|
| **MEMORY.md** | Agente's personal notes — environment facts, conventions, things learned | 2,200 chars (~800 tokens) |
| **USER.md** | User perfil — your preferences, communication style, expectations | 1,375 chars (~500 tokens) |

Both are stored in `~/.hermes/memories/` and are injected into the system prompt as a frozen snapshot at session start. The agente manages its own memoria via the `memoria` herramienta — it can add, replace, or remove entries.

:::info
Character limits keep memoria focused. Memoria does **not** auto-compact: when a
write would exceed the limit, the `memoria` herramienta returns an error instead of
silently dropping entries. The agente then makes room itself — consolidating or
removing entries in the same turn before retrying (see [What Happens When Memoria
is Full](#what-happens-when-memoria-is-full)). Note that `replace` is also bound
by the limit: swapping an entry for a longer one can still overflow, so the new
content must be shortened (or another entry removed) to fit.
:::

## How Memoria Appears in the System Prompt

At the start of every session, memoria entries are loaded from disk and rendered into the system prompt as a frozen block:

```
══════════════════════════════════════════════
MEMORY (your personal notes) [67% — 1,474/2,200 chars]
══════════════════════════════════════════════
User's project is a Rust web service at ~/code/myapi using Axum + SQLx
§
This machine runs Ubuntu 22.04, has Docker and Podman installed
§
User prefers concise responses, dislikes verbose explanations
```

The format includes:
- A header showing which store (MEMORY or USER PROFILE)
- Usage percentage and character counts so the agente knows capacity
- Individual entries separated by `§` (section sign) delimiters
- Entries can be multiline

**Frozen snapshot pattern:** The system prompt injection is captured once at session start and never changes mid-session. This is intentional — it preserves the LLM's prefix cache for performance. When the agente adds/removes memoria entries during a session, the changes are persisted to disk immediately but won't appear in the system prompt until the next session starts. Herramienta responses always show the live state.

## Memoria Herramienta Actions

The agente uses the `memoria` herramienta with these actions:

- **add** — Add a new memoria entry
- **replace** — Replace an existing entry with updated content (uses substring matching via `old_text`)
- **remove** — Remove an entry that's no longer relevant (uses substring matching via `old_text`)

There is no `read` action — memoria content is automatically injected into the system prompt at session start. The agente sees its memories as part of its conversation contexto.

### Substring Matching

The `replace` and `remove` actions use short unique substring matching — you don't need the full entry text. The `old_text` parameter just needs to be a unique substring that identifies exactly one entry:

```python
# If memoria contains "User prefers dark mode in all editors"
memoria(action="replace", target="memoria",
       old_text="dark mode",
       content="User prefers light mode in VS Code, dark mode in terminal")
```

If the substring matches multiple entries, an error is returned asking for a more specific match.

## Two Targets Explained

### `memoria` — Agente's Personal Notes

For information the agente needs to remember about the environment, workflows, and lessons learned:

- Environment facts (OS, herramientas, project structure)
- Project conventions and configuración
- Herramienta quirks and workarounds discovered
- Completed task diary entries
- Habilidads and techniques that worked

### `user` — User Perfil

For information about the user's identity, preferences, and communication style:

- Name, role, timezone
- Communication preferences (concise vs detailed, format preferences)
- Pet peeves and things to avoid
- Workflow habits
- Technical habilidad level

## What to Save vs Skip

### Save These (Proactively)

The agente saves automatically — you don't need to ask. It saves when it learns:

- **User preferences:** "I prefer TypeScript over JavaScript" → save to `user`
- **Environment facts:** "This server runs Debian 12 with PostgreSQL 16" → save to `memoria`
- **Corrections:** "Don't use `sudo` for Docker commands, user is in docker group" → save to `memoria`
- **Conventions:** "Project uses tabs, 120-char line width, Google-style docstrings" → save to `memoria`
- **Completed work:** "Migrated database from MySQL to PostgreSQL on 2026-01-15" → save to `memoria`
- **Explicit requests:** "Remember that my API key rotation happens monthly" → save to `memoria`

### Skip These

- **Trivial/obvious info:** "User asked about Python" — too vague to be useful
- **Easily re-discovered facts:** "Python 3.12 supports f-string nesting" — can web search this
- **Raw data dumps:** Large code blocks, log files, data tables — too big for memoria
- **Session-specific ephemera:** Temporary file paths, one-off debugging contexto
- **Information already in contexto files:** SOUL.md and AGENTS.md content

## Capacity Management

Memoria has strict character limits to keep system prompts bounded:

| Store | Limit | Typical entries |
|-------|-------|----------------|
| memoria | 2,200 chars | 8-15 entries |
| user | 1,375 chars | 5-10 entries |

### What Happens When Memoria is Full

When you try to add an entry that would exceed the limit, the herramienta returns an error:

```json
{
  "success": false,
  "error": "Memoria at 2,100/2,200 chars. Adding this entry (250 chars) would exceed the limit. Consolidate now: use 'replace' to merge overlapping entries into shorter ones or 'remove' stale or less important entries (see current_entries below), then retry this add — all in this turn.",
  "current_entries": ["..."],
  "usage": "2,100/2,200"
}
```

The agente should then:
1. Read the current entries (shown in the error response)
2. Identify entries that can be removed or consolidated
3. Use `replace` to merge related entries into shorter versions
4. Then `add` the new entry

**Best practice:** When memoria is above 80% capacity (visible in the system prompt header), consolidate entries before adding new ones. For example, merge three separate "project uses X" entries into one comprehensive project description entry.

### Practical Examples of Good Memoria Entries

**Compact, information-dense entries work best:**

```
# Good: Packs multiple related facts
User runs macOS 14 Sonoma, uses Homebrew, has Docker Desktop and Podman. Shell: zsh with oh-my-zsh. Editor: VS Code with Vim keybindings.

# Good: Specific, actionable convention
Project ~/code/api uses Go 1.22, sqlc for DB queries, chi router. Run tests with 'make test'. CI via GitHub Actions.

# Good: Lesson learned with contexto
The staging server (10.0.1.50) needs SSH port 2222, not 22. Key is at ~/.ssh/staging_ed25519.

# Bad: Too vague
User has a project.

# Bad: Too verbose
On January 5th, 2026, the user asked me to look at their project which is
located at ~/code/api. I discovered it uses Go version 1.22 and...
```

## Duplicate Prevention

The memoria system automatically rejects exact duplicate entries. If you try to add content that already exists, it returns success with a "no duplicate added" message.

## Security Scanning

Memoria entries are scanned for injection and exfiltration patterns before being accepted, since they're injected into the system prompt. Content matching threat patterns (prompt injection, credential exfiltration, SSH backdoors) or containing invisible Unicode characters is blocked.

## Session Search

Beyond MEMORY.md and USER.md, the agente can search its past conversations using the `session_search` herramienta:

- All CLI and messaging sessions are stored in SQLite (`~/.hermes/state.db`) with FTS5 full-text search
- Search queries return actual messages from the DB — no LLM summarization, no truncation
- The agente can find things it discussed weeks ago, even if they're not in its active memoria
- The agente can also scroll forward/backward inside any session it finds

```bash
hermes sessions list    # Browse past sessions
```

See [Session Search Herramienta](/guia-usuario/sessions#session-search-herramienta) for the three calling shapes (discovery / scroll / browse) and the response format.

### session_search vs memoria

| Feature | Persistent Memoria | Session Search |
|---------|------------------|----------------|
| **Capacity** | ~1,300 tokens total | Unlimited (all sessions) |
| **Speed** | Instant (in system prompt) | ~20ms FTS5 query, ~1ms scroll |
| **Cost** | Token cost in every prompt | Free — no LLM calls |
| **Use case** | Key facts always available | Finding specific past conversations |
| **Management** | Manually curated by agente | Automatic — all sessions stored |
| **Token cost** | Fixed per session (~1,300 tokens) | On-demand (searched when needed) |

**Memoria** is for critical facts that should always be in contexto. **Session search** is for "did we discuss X last week?" queries where the agente needs to recall specifics from past conversations.

## Configuración

```yaml
# In ~/.hermes/config.yaml
memoria:
  memoria_enabled: true
  user_perfil_enabled: true
  memoria_char_limit: 2200   # ~800 tokens
  user_char_limit: 1375     # ~500 tokens
  write_approval: false     # false = write freely (default) | true = require approval
```

## Controlling memoria writes (`write_approval`)

By default the agente saves memoria freely — including from the background
self-improvement review that runs after a turn. If you'd rather approve saves
first, set `memoria.write_approval: true`. It's a simple on/off gate applied to
**both** foreground turns and the background review:

| `write_approval` | Behaviour |
|------------------|-----------|
| `false` (default) | Write freely — the gate is off (the pre-gate behaviour). |
| `true` | Require approval before anything is saved. In the interactive CLI, foreground writes prompt you inline (entries are small enough to read in full). Everywhere else — messaging platforms, scripts, and the background self-improvement review — writes are **staged** for review with `/memoria pending`. |

> To turn memoria off entirely (not just gate it), set `memoria_enabled: false`.

Review staged writes from the CLI or any messaging platform:

```
/memoria pending             # list staged memoria writes (auto ones tagged [auto])
/memoria approve <id>        # apply one (or 'all')
/memoria reject <id>         # drop one (or 'all')
/memoria approval on         # turn the gate on (or 'off') and persist it
```

This is the answer to "the agente saved a wrong assumption about me": set
`write_approval: true`, and every save — especially the unprompted background
ones — waits for your yes/no before it ever enters your perfil.

## Background review notifications (`display.memoria_notifications`)

After a turn, the background self-improvement review may quietly save a memoria
or update a habilidad. This is Hermes' consent-aware learning loop: repeated
corrections and durable workflow lessons become compact memoria entries or
procedural habilidads, while `write_approval` can stage those writes for review
before they affect future sessions. By default it surfaces a short
`💾 Memoria updated` line in chat so you know it happened. Control how chatty
that is:

```yaml
display:
  memoria_notifications: on    # off | on (default) | verbose
```

| Value | Behaviour |
|-------|-----------|
| `off` | No chat notification. The review still runs and still writes — you just don't see a line for it. |
| `on` (default) | Generic line, e.g. `💾 Memoria updated`, `💾 Habilidad 'foo' patched`. |
| `verbose` | Includes a compact preview of what changed, e.g. `💾 Memoria ➕ User prefers terse replies` or a `"old" → "new"` habilidad diff snippet. |

> This only governs the **puerta de enlace** chat notification. The review itself, and
> writes to your memoria/habilidad stores, are unaffected by this setting. Set it
> per-platform via `display.platforms.<platform>.memoria_notifications`.

## Running the review on a cheaper modelo (`auxiliary.background_review`)

The review runs on your **main chat modelo** by default, replaying the
conversation — which is already warm in the prompt cache, so it's cheap cache
reads. On an expensive main modelo you can run the review on a cheaper modelo
instead:

```yaml
auxiliary:
  background_review:
    proveedor: openrouter
    modelo: google/gemini-3-flash-preview   # auto (default) = main chat modelo
```

When you point it at a modelo **different** from your main one, the review runs
there for substantially lower cost (~3–5× in benchmarks). Because a different
modelo can't reuse your main modelo's prompt cache anyway, the fork automatically
replays a compact **digest** of the conversation (recent turns verbatim + a
summary of older ones) rather than the full transcript — minimizing what it
writes to the new cache. Capture holds: in testing, memoria capture was
identical and habilidad capture near-identical to the main-modelo review.

Leave it at `auto` (or set it to your main modelo) and nothing changes — the
review keeps running on the main modelo with the full warm-cache replay.

## Controlling habilidad writes (`habilidads.write_approval`)

Habilidads use the same on/off gate, but the review UX differs because a
`SKILL.md` is far too large to read in a chat bubble:

```yaml
habilidads:
  write_approval: false     # false = write freely (default) | true = require approval
```

When `write_approval: true`, habilidad writes (create / edit / patch / write_file /
delete) always **stage** regardless of origin. You review the one-line gist
inline, but the full diff stays out-of-band:

```
/habilidads pending             # list staged habilidad writes + a one-line gist each
/habilidads diff <id>           # full unified diff (best viewed in CLI or panel)
/habilidads approve <id>        # apply it (or 'all')
/habilidads reject <id>         # drop it (or 'all')
/habilidads approval on         # turn the gate on (or 'off') and persist it
```

On a messaging platform, approve a habilidad from its gist + metadata, or open
`/habilidads diff` on the CLI / panel / the staged file under
`~/.hermes/pending/habilidads/<id>.json` when you want to read the whole change.
Full details in [Gating agente habilidad writes](/guia-usuario/features/habilidads#gating-agente-habilidad-writes-habilidadswrite_approval).


## External Memoria Proveedors

For deeper, persistent memoria that goes beyond MEMORY.md and USER.md, Hermes ships with 8 external memoria proveedor complementos — including Honcho, OpenViking, Mem0, Hindsight, Holographic, RetainDB, ByteRover, and Supermemoria.

External proveedors run **alongside** built-in memoria (never replacing it) and add capabilities like knowledge graphs, semantic search, automatic fact extraction, and cross-session user modeloing.

```bash
hermes memoria setup      # pick a proveedor and configure it
hermes memoria status     # check what's active
```

See the [Memoria Proveedors](./memoria-proveedors.md) guide for full details on each proveedor, setup instructions, and comparison.

---