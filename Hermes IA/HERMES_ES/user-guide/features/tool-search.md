<!-- source: website/docs/guia-usuario/features/herramienta-search.md -->
# guia-usuario/features/herramienta-search.md

# Herramienta Search

When you have many MCP servers or non-core complemento herramientas attached to a
session, their JSON schemas can consume a substantial fraction of the
contexto window on every turn — even when only a few of them are relevant
to what the user actually asked for.

**Herramienta Search** is Hermes' opt-in progressive-disclosure layer for that
problem. When activated, MCP and complemento herramientas are replaced in the
modelo-visible herramientas array by three bridge herramientas, and the modelo loads each
specific herramienta's schema on demand.

:::info Built-in Hermes herramientas never defer
The herramientas that make up Hermes' core capability set (`terminal`,
`read_file`, `write_file`, `patch`, `search_files`, `todo`, `memoria`,
`browser_*`, `web_search`, `web_extract`, `clarify`, `execute_code`,
`delegate_task`, `session_search`, `send_message`, and the rest of
`_HERMES_CORE_TOOLS`) are *always* loaded directly. Only MCP herramientas and
non-core complemento herramientas are eligible for deferral.
:::

## How it works

When Herramienta Search activates for a turn, the modelo sees three new herramientas in
place of the deferred ones:

```
herramienta_search(query, limit?)     — search the deferred-herramienta catalog
herramienta_describe(name)            — load the full schema for one herramienta
herramienta_call(name, arguments)     — invoke a deferred herramienta
```

A typical interaction looks like:

```
Modelo: herramienta_search("create a github issue")
  → { matches: [{ name: "mcp_github_create_issue", ... }, ...] }
Modelo: herramienta_describe("mcp_github_create_issue")
  → { parameters: { type: "object", properties: { ... } } }
Modelo: herramienta_call("mcp_github_create_issue", { title: "...", body: "..." })
  → { ok: true, issue_number: 42 }
```

When the modelo invokes `herramienta_call`, Hermes **unwraps the bridge** and
dispatches the underlying herramienta exactly as if the modelo had called it
directly. Pre-herramienta-call hooks, guardrails, approval prompts, and
post-herramienta-call hooks all run against the real herramienta name — not against
`herramienta_call`. The activity feed in the CLI and puerta de enlace also unwraps so you
see the underlying herramienta, not the bridge.

## When does it activate?

By default Herramienta Search runs in `auto` mode: it activates only when the
deferrable herramienta schemas would consume at least 10% of the active modelo's
contexto window. Below that, the herramientas-array assembly is a pure
pass-through and you pay no overhead.

This decision is re-evaluated every time the herramientas array is built, so:

- A session with just a few MCP herramientas and a long contexto modelo never
  activates Herramienta Search.
- A session with many MCP servers attached (15+ herramientas typically) starts
  activating it.
- Removing MCP servers mid-session correctly returns to direct exposure
  on the next assembly.

## Configuración

```yaml
herramientas:
  herramienta_search:
    enabled: auto       # auto (default), on, or off
    threshold_pct: 10   # percentage of contexto — only used in auto mode
    search_default_limit: 5
    max_search_limit: 20
```

| Key | Default | Meaning |
| --- | --- | --- |
| `enabled` | `auto` | `auto` activates above threshold; `on` always activates if there's at least one deferrable herramienta; `off` disables entirely. |
| `threshold_pct` | `10` | Percentage of contexto length at which `auto` mode kicks in. Range 0–100. |
| `search_default_limit` | `5` | Hits returned when the modelo calls `herramienta_search` without a `limit`. |
| `max_search_limit` | `20` | Hard upper bound the modelo can request via `limit`. Range 1–50. |

You can also flip the legacy boolean shape:

```yaml
herramientas:
  herramienta_search: true   # equivalent to {enabled: auto}
```

## When NOT to use it

Herramienta Search trades a fixed per-turn token cost (the three bridge herramienta
schemas, ~300 tokens) and at least one extra round trip (search →
describe → call) for the savings on the deferred schemas. It's a clear
win when you have many herramientas and use few per turn; it's overhead when
you have few herramientas total.

The `auto` default handles this for you. If you set `enabled: on`
unconditionally, expect a slight per-turn cost on small herramientasets.

## Trade-offs that don't go away

These come from the prompt-cache integrity invariant — they are inherent
to any progressive-disclosure design, not specific to this implementation:

- **One extra round trip on cold herramientas.** The first time the modelo needs
  a deferred herramienta, it spends one or two extra modelo calls to find and
  load the schema. The token savings on the static side are real, but a
  portion is paid back at runtime.
- **No cache benefit on deferred schemas.** A loaded `herramienta_describe`
  result enters the conversation history (so it does get cached on
  subsequent turns) but it never benefits from the system-prompt cache
  prefix.
- **Modelo-quality dependence.** Herramienta Search assumes the modelo can write a
  reasonable search query for the herramienta it wants. Smaller modelos do this
  less well; the published Anthropic numbers (49% → 74% on Opus 4 with
  vs. without herramienta search) show the upside but also that ~26 points of
  accuracy is still retrieval failure.
- **Herramientaset edits invalidate cache.** Adding or removing a herramienta mid-
  session changes the bridge herramientas' descriptions (which include the
  count of deferred herramientas) and the catalog, so the prompt cache is
  invalidated. This is the same trade-off as any herramientaset edit.

## Implementation details

- **Retrieval:** BM25 over tokenized herramienta name + description + parameter
  names. Falls back to a literal substring match on the herramienta name when
  BM25 returns no positive-score hits, which protects against
  zero-IDF degenerate cases (e.g. searching `"github"` against a
  catalog where every herramienta name contains "github").
- **Catalog is stateless across turns.** It rebuilds from the current
  herramienta-defs list every assembly — no session-keyed `Map`. This avoids
  the class of bug where a stored catalog drifts out of sync with the
  live herramienta registry.
- **The catalog is scoped to the session's herramientasets.** `herramienta_search`,
  `herramienta_describe`, and `herramienta_call` only ever see and invoke herramientas the
  session was actually granted. A subagente, kanban worker, or puerta de enlace
  session restricted to a subset of herramientasets cannot use the bridge to
  discover or call a herramienta outside that subset — the deferred catalog is
  the deferrable slice of the session's own enabled/disabled herramientasets,
  not the whole process registry.
- **No JS sandbox.** Hermes uses the simpler "structured herramientas" mode
  (search / describe / call as plain functions). The JS-sandbox "code
  mode" some other implementations offer is a large surface area; we
  skip it.

## See also

- `herramientas/herramienta_search.py` — the implementation
- `tests/herramientas/test_herramienta_search.py` — the regression suite
- The `openclaw-herramienta-search-report` PDF in the original implementation
  PR for the research that shaped the design

---