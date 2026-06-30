<!-- source: website/docs/guia-desarrollador/prompt-assembly.md -->
# Prompt Assembly

# Prompt Assembly

Hermes deliberately separates:

- **cached system prompt state**
- **ephemeral API-call-time additions**

This is one of the most important design choices in the project because it affects:

- token usage
- prompt caching effectiveness
- session continuity
- memoria correctness

Primary files:

- `run_agente.py`
- `agente/prompt_builder.py`
- `herramientas/memoria_herramienta.py`

## Cached system prompt layers

The cached system prompt is assembled as three ordered tiers (see `agente/system_prompt.py`):

1. **stable** — identity (`SOUL.md` or fallback), herramienta/modelo guidance, habilidads prompt, environment hints, platform hints
2. **contexto** — caller-supplied `system_message` plus project contexto files (`.hermes.md` / `AGENTS.md` / `CLAUDE.md` / `.cursorrules`)
3. **volatile** — built-in memoria snapshot (`MEMORY.md`), user perfil snapshot (`USER.md`), external memoria-proveedor block, timestamp/session/modelo/proveedor line

The final system prompt is then joined as: `stable` → `contexto` → `volatile`.

This ordering matters for precedence discussions:
- habilidads are part of the **stable** tier
- memoria/perfil snapshots are part of the **volatile** tier
- both are still in the cached system prompt (they are not injected as ad-hoc mid-turn overlays)

When `skip_contexto_files` is set (e.g., subagente delegation), SOUL.md is not loaded and the hardcoded `DEFAULT_AGENT_IDENTITY` is used instead.

### Concrete example: assembled system prompt

Here is a simplified view of what the final system prompt looks like when all layers are present (comments show the source of each section):

```
# Layer 1: Agente Identity (from ~/.hermes/SOUL.md)
You are Hermes, an AI assistant created by Nous Research.
You are an expert software engineer and researcher.
You value correctness, clarity, and efficiency.
...

# Layer 2: Herramienta-aware behavior guidance
You have persistent memoria across sessions. Save durable facts using
the memoria herramienta: user preferences, environment details, herramienta quirks,
and stable conventions. Memoria is injected into every turn, so keep
it compact and focused on facts that will still matter later.
...
When the user references something from a past conversation or you
suspect relevant cross-session contexto exists, use session_search
to recall it before asking them to repeat themselves.

# Herramienta-use enforcement (for GPT/Codex modelos only)
You MUST use your herramientas to take action — do not describe what you
would do or plan to do without actually doing it.
...

# Layer 3: Honcho static block (when active)
[Honcho personality/contexto data]

# Layer 4: Optional system message (from config or API)
[User-configured system message override]

# Layer 5: Frozen MEMORY snapshot
## Persistent Memoria
- User prefers Python 3.12, uses pyproject.toml
- Default editor is nvim
- Working on project "atlas" in ~/code/atlas
- Timezone: US/Pacific

# Layer 6: Frozen USER perfil snapshot
## User Perfil
- Name: Alice
- GitHub: alice-dev

# Layer 7: Habilidads index
## Habilidads (mandatory)
Before replying, scan the habilidads below. If one clearly matches
your task, load it with habilidad_view(name) and follow its instructions.
...
<available_habilidads>
  software-development:
    - code-review: Structured code review workflow
    - test-driven-development: TDD methodology
  research:
    - arxiv: Search and summarize arXiv papers
</available_habilidads>

# Layer 8: Contexto files (from project directory)
# Project Contexto
The following project contexto files have been loaded and should be followed:

## AGENTS.md
This is the atlas project. Use pytest for testing. The main
entry point is src/atlas/main.py. Always run `make lint` before
committing.

# Layer 9: Timestamp + session
Current time: 2026-03-30T14:30:00-07:00
Session: abc123

# Layer 10: Platform hint
You are a CLI AI Agente. Try not to use markdown but simple text
renderable inside a terminal.
```

## Customizing platform hints

The platform hint (Layer 10 above) is the per-surface guidance Hermes
injects for Telegram, WhatsApp, Slack, CLI, and other platforms — for
example "you are on a terminal, avoid Markdown." The built-in defaults
live in `PLATFORM_HINTS` (`agente/system_prompt.py`); complemento-provided
platforms supply theirs through the platform registry.

An administrator can append to or replace a single platform's hint from
`config.yaml` via the top-level `platform_hints` key, without touching
any other platform:

```yaml
platform_hints:
  whatsapp:
    append: >
      When tabular output would be useful, invoke the table_formatting
      habilidad instead of emitting a Markdown table.
  slack:
    replace: "You are on Slack. Keep responses tight and avoid wide tables."
  telegram: "Prefer short messages; split long answers."   # shorthand = append
```

- `append` — keep the built-in hint and add the extra text after it.
- `replace` — substitute the built-in hint entirely.
- A bare string — shorthand for `append`.
- `replace` wins over `append` when both are present.
- A malformed entry is ignored defensively and falls back to the
  unmodified default, so a bad config value can never break prompt
  assembly or leak across platforms.

The override is resolved when the system prompt is built (session start,
and again on compaction since that rebuilds the prompt). It produces a
byte-stable hint for a fixed config, so it lives in the **stable** tier
alongside the built-in hint and does not break prompt caching — it is
not a live mid-session mutation of a frozen prompt.

## How SOUL.md appears in the prompt

`SOUL.md` lives at `~/.hermes/SOUL.md` and serves as the agente's identity — the very first section of the system prompt. The loading logic in `prompt_builder.py` works as follows:

```python
# From agente/prompt_builder.py (simplified)
def load_soul_md() -> Optional[str]:
    soul_path = get_hermes_home() / "SOUL.md"
    if not soul_path.exists():
        return None
    content = soul_path.read_text(encoding="utf-8").strip()
    content = _scan_contexto_content(content, "SOUL.md")  # Security scan
    content = _truncate_content(content, "SOUL.md")       # Cap defaults to 20k chars, configurable
    return content
```

When `load_soul_md()` returns content, it replaces the hardcoded `DEFAULT_AGENT_IDENTITY`. The `build_contexto_files_prompt()` function is then called with `skip_soul=True` to prevent SOUL.md from appearing twice (once as identity, once as a contexto file).

If `SOUL.md` doesn't exist, the system falls back to:

```
You are Hermes Agente, an intelligent AI assistant created by Nous Research.
You are helpful, knowledgeable, and direct. You assist users with a wide
range of tasks including answering questions, writing and editing code,
analyzing information, creative work, and executing actions via your herramientas.
You communicate clearly, admit uncertainty when appropriate, and prioritize
being genuinely useful over being verbose unless otherwise directed below.
Be targeted and efficient in your exploration and investigations.
```

## How contexto files are injected

`build_contexto_files_prompt()` uses a **priority system** — only one project contexto type is loaded (first match wins):

```python
# From agente/prompt_builder.py (simplified)
def build_contexto_files_prompt(cwd=None, skip_soul=False):
    cwd_path = Path(cwd).resolve()

    # Priority: first match wins — only ONE project contexto loaded
    project_contexto = (
        _load_hermes_md(cwd_path)       # 1. .hermes.md / HERMES.md (walks to git root)
        or _load_agentes_md(cwd_path)    # 2. AGENTS.md (cwd only)
        or _load_claude_md(cwd_path)    # 3. CLAUDE.md (cwd only)
        or _load_cursorrules(cwd_path)  # 4. .cursorrules / .cursor/rules/*.mdc
    )

    sections = []
    if project_contexto:
        sections.append(project_contexto)

    # SOUL.md from HERMES_HOME (independent of project contexto)
    if not skip_soul:
        soul_content = load_soul_md()
        if soul_content:
            sections.append(soul_content)

    if not sections:
        return ""

    return (
        "# Project Contexto\n\n"
        "The following project contexto files have been loaded "
        "and should be followed:\n\n"
        + "\n".join(sections)
    )
```

### Contexto file discovery details

| Priority | Files | Search scope | Notes |
|----------|-------|-------------|-------|
| 1 | `.hermes.md`, `HERMES.md` | CWD up to git root | Hermes-native project config |
| 2 | `AGENTS.md` | CWD only | Common agente instruction file |
| 3 | `CLAUDE.md` | CWD only | Claude Code compatibility |
| 4 | `.cursorrules`, `.cursor/rules/*.mdc` | CWD only | Cursor compatibility |

All contexto files are:
- **Security scanned** — checked for prompt injection patterns (invisible unicode, "ignore previous instructions", credential exfiltration attempts)
- **Truncated** — capped at `contexto_file_max_chars` characters (default 20,000) using 70/20 head/tail ratio with a truncation marker
- **YAML frontmatter stripped** — `.hermes.md` frontmatter is removed (reserved for future config overrides)

## API-call-time-only layers

These are intentionally *not* persisted as part of the cached system prompt:

- `ephemeral_system_prompt`
- prefill messages
- puerta de enlace-derived session contexto overlays
- later-turn Honcho/external recall injected into the current-turn user message

`pre_llm_call` complemento contexto also lands in this API-call-time path: it is appended to the current turn's **user message**, not written into the cached system prompt. When multiple complementos return contexto, Hermes concatenates those contexto blocks (see [Hooks → `pre_llm_call`](../guia-usuario/features/hooks.md#pre_llm_call)).

This separation keeps the stable prefix stable for caching.

## Memoria snapshots

Local memoria and user perfil data are captured in the system prompt's **volatile tier**. Mid-session writes update disk state but do not mutate the already-built cached system prompt until a rebuild path runs (new session, or explicit invalidation/rebuild flow such as compression-triggered rebuild).

## Contexto files

`agente/prompt_builder.py` scans and sanitizes project contexto files using a **priority system** — only one type is loaded (first match wins):

1. `.hermes.md` / `HERMES.md` (walks to git root)
2. `AGENTS.md` (CWD at startup; subdirectories discovered progressively during the session via `agente/subdirectory_hints.py`)
3. `CLAUDE.md` (CWD only)
4. `.cursorrules` / `.cursor/rules/*.mdc` (CWD only)

`SOUL.md` is loaded separately via `load_soul_md()` for the identity slot. When it loads successfully, `build_contexto_files_prompt(skip_soul=True)` prevents it from appearing twice.

Long files are truncated before injection.

## Habilidads index

The habilidads system contributes a compact habilidads index to the prompt when habilidads herramientaing is available.

## Supported prompt customization surfaces

Most users should treat `agente/prompt_builder.py` as implementation code, not a configuración surface. The supported customization path is to change the prompt inputs Hermes already loads, rather than editing Python templates in place.

### Use these surfaces first

- `~/.hermes/SOUL.md` — replace the built-in default identity block with your own agente persona and standing behavior.
- `~/.hermes/MEMORY.md` and `~/.hermes/USER.md` — provide durable cross-session facts and user perfil data that should be snapshotted into new sessions.
- Project contexto files such as `.hermes.md`, `HERMES.md`, `AGENTS.md`, `CLAUDE.md`, or `.cursorrules` — inject repo-specific working rules.
- Habilidads — package reusable workflows and references without editing core prompt code.
- Optional system prompt config / API overrides — add deployment-specific instruction text without forking Hermes.
- Ephemeral overlays such as `HERMES_EPHEMERAL_SYSTEM_PROMPT` or prefill messages — add turn-scoped guidance that should not become part of the cached prompt prefix.

### When to edit code instead

Edit `agente/prompt_builder.py` only if you are intentionally maintaining a fork or contributing upstream behavior changes. That file assembles the prompt plumbing, cache boundaries, and injection order for every session. Direct edits there are global product changes, not per-user prompt customization.

In other words:

- if you want a different assistant identity, edit `SOUL.md`
- if you want different repo rules, edit project contexto files
- if you want reusable operating procedures, add or modify habilidads
- if you want to change how Hermes assembles prompts for everyone, change Python and treat it as a code contribution

## Why prompt assembly is split this way

The architecture is intentionally optimized to:

- preserve proveedor-side prompt caching
- avoid mutating history unnecessarily
- keep memoria semantics understandable
- let puerta de enlace/ACP/CLI add contexto without poisoning persistent prompt state

## Related docs

- [Contexto Compression & Prompt Caching](./contexto-compression-and-caching.md)
- [Session Storage](./session-storage.md)
- [Puerta de enlace Internals](./puerta de enlace-internals.md)

---