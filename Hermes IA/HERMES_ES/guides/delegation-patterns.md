<!-- source: website/docs/guides/delegation-patterns.md -->
# Delegation & Parallel Work

# Delegation & Parallel Work

Hermes can spawn isolated child agentes to work on tasks in parallel. Each subagente gets its own conversation, terminal session, and herramientaset. Only the final summary comes back — intermediate herramienta calls never enter your contexto window.

For the full feature reference, see [Subagente Delegation](/guia-usuario/features/delegation).

---

## When to Delegate

**Good candidates for delegation:**
- Reasoning-heavy subtasks (debugging, code review, research synthesis)
- Tasks that would flood your contexto with intermediate data
- Parallel independent workstreams (research A and B simultaneously)
- Fresh-contexto tasks where you want the agente to approach without bias

**Use something else:**
- Single herramienta call → just use the herramienta directly
- Mechanical multi-step work with logic between steps → `execute_code`
- Tasks needing user interaction → subagentes can't use `clarify`
- Quick file edits → do them directly
- Durable long-running work that must outlive the current turn → `cronjob` or `terminal(background=True, notify_on_complete=True)`. `delegate_task` is **synchronous**: if the parent turn is interrupted, active children are cancelled and their work is discarded.

---

## Pattern: Parallel Research

Research three topics simultaneously and get structured summaries back:

```
Research these three topics in parallel:
1. Current state of WebAssembly outside the browser
2. RISC-V server chip adoption in 2025
3. Practical quantum computing applications

Focus on recent developments and key players.
```

Behind the scenes, Hermes uses:

```python
delegate_task(tasks=[
    {
        "goal": "Research WebAssembly outside the browser in 2025",
        "contexto": "Focus on: runtimes (Wasmtime, Wasmer), cloud/edge use cases, WASI progress",
        "herramientasets": ["web"]
    },
    {
        "goal": "Research RISC-V server chip adoption",
        "contexto": "Focus on: server chips shipping, cloud proveedors adopting, software ecosystem",
        "herramientasets": ["web"]
    },
    {
        "goal": "Research practical quantum computing applications",
        "contexto": "Focus on: error correction breakthroughs, real-world use cases, key companies",
        "herramientasets": ["web"]
    }
])
```

All three run concurrently. Each subagente searches the web independently and returns a summary. The parent agente then synthesizes them into a coherent briefing.

---

## Pattern: Code Review

Delegate a security review to a fresh-contexto subagente that approaches the code without preconceptions:

```
Review the autenticación module at src/auth/ for security issues.
Check for SQL injection, JWT validation problems, password handling,
and session management. Fix anything you find and run the tests.
```

The key is the `contexto` field — it must include everything the subagente needs:

```python
delegate_task(
    goal="Review src/auth/ for security issues and fix any found",
    contexto="""Project at /home/user/webapp. Python 3.11, Flask, PyJWT, bcrypt.
    Auth files: src/auth/login.py, src/auth/jwt.py, src/auth/middleware.py
    Test command: pytest tests/auth/ -v
    Focus on: SQL injection, JWT validation, password hashing, session management.
    Fix issues found and verify tests pass.""",
    herramientasets=["terminal", "file"]
)
```

:::warning The Contexto Problem
Subagentes know **absolutely nothing** about your conversation. They start completely fresh. If you delegate "fix the bug we were discussing," the subagente has no idea what bug you mean. Always pass file paths, error messages, project structure, and constraints explicitly.
:::

---

## Pattern: Compare Alternatives

Evaluate multiple approaches to the same problem in parallel, then pick the best:

```
I need to add full-text search to our Django app. Evaluate three approaches
in parallel:
1. PostgreSQL tsvector (built-in)
2. Elasticsearch via django-elasticsearch-dsl
3. Meilisearch via meilisearch-python

For each: setup complexity, query capabilities, resource requirements,
and maintenance overhead. Compare them and recommend one.
```

Each subagente researches one option independently. Because they're isolated, there's no cross-contamination — each evaluation stands on its own merits. The parent agente gets all three summaries and makes the comparison.

---

## Pattern: Multi-File Refactoring

Split a large refactoring task across parallel subagentes, each handling a different part of the codebase:

```python
delegate_task(tasks=[
    {
        "goal": "Refactor all API endpoint handlers to use the new response format",
        "contexto": """Project at /home/user/api-server.
        Files: src/handlers/users.py, src/handlers/auth.py, src/handlers/billing.py
        Old format: return {"data": result, "status": "ok"}
        New format: return APIResponse(data=result, status=200).to_dict()
        Import: from src.responses import APIResponse
        Run tests after: pytest tests/handlers/ -v""",
        "herramientasets": ["terminal", "file"]
    },
    {
        "goal": "Update all client SDK methods to handle the new response format",
        "contexto": """Project at /home/user/api-server.
        Files: sdk/python/client.py, sdk/python/modelos.py
        Old parsing: result = response.json()["data"]
        New parsing: result = response.json()["data"] (same key, but add status code checking)
        Also update sdk/python/tests/test_client.py""",
        "herramientasets": ["terminal", "file"]
    },
    {
        "goal": "Update API documentation to reflect the new response format",
        "contexto": """Project at /home/user/api-server.
        Docs at: docs/api/. Format: Markdown with code examples.
        Update all response examples from old format to new format.
        Add a 'Response Format' section to docs/api/overview.md explaining the schema.""",
        "herramientasets": ["terminal", "file"]
    }
])
```

:::tip
Each subagente gets its own terminal session. They can work on the same project directory without stepping on each other — as long as they're editing different files. If two subagentes might touch the same file, handle that file yourself after the parallel work completes.
:::

---

## Pattern: Gather Then Analyze

Use `execute_code` for mechanical data gathering, then delegate the reasoning-heavy analysis:

```python
# Step 1: Mechanical gathering (execute_code is better here — no reasoning needed)
execute_code("""
from hermes_herramientas import web_search, web_extract

results = []
for query in ["AI funding Q1 2026", "AI startup acquisitions 2026", "AI IPOs 2026"]:
    r = web_search(query, limit=5)
    for item in r["data"]["web"]:
        results.append({"title": item["title"], "url": item["url"], "desc": item["description"]})

# Extract full content from top 5 most relevant
urls = [r["url"] for r in results[:5]]
content = web_extract(urls)

# Save for the analysis step
import json
with open("/tmp/ai-funding-data.json", "w") as f:
    json.dump({"search_results": results, "extracted": content["results"]}, f)
print(f"Collected {len(results)} results, extracted {len(content['results'])} pages")
""")

# Step 2: Reasoning-heavy analysis (delegation is better here)
delegate_task(
    goal="Analyze AI funding data and write a market report",
    contexto="""Raw data at /tmp/ai-funding-data.json contains search results and
    extracted web pages about AI funding, acquisitions, and IPOs in Q1 2026.
    Write a structured market report: key deals, trends, notable players,
    and outlook. Focus on deals over $100M.""",
    herramientasets=["terminal", "file"]
)
```

This is often the most efficient pattern: `execute_code` handles the 10+ sequential herramienta calls cheaply, then a subagente does the single expensive reasoning task with a clean contexto.

---

## Herramientaset Selection

Choose herramientasets based on what the subagente needs:

| Task type | Herramientasets | Why |
|-----------|----------|-----|
| Web research | `["web"]` | web_search + web_extract only |
| Code work | `["terminal", "file"]` | Shell access + file operations |
| Full-stack | `["terminal", "file", "web"]` | Everything except messaging |
| Read-only analysis | `["file"]` | Can only read files, no shell |

Restricting herramientasets keeps the subagente focused and prevents accidental side effects (like a research subagente running shell commands).

---

## Constraints

- **Default 3 parallel tasks**: batches default to 3 concurrent subagentes (configurable via `delegation.max_concurrent_children` in config.yaml, no hard ceiling, only a floor of 1)
- **Nested delegation is opt-in**: leaf subagentes (default) cannot call `delegate_task`, `clarify`, `memoria`, `send_message`, or `execute_code`. Orchestrator subagentes (`role="orchestrator"`) retain `delegate_task` for further delegation, but only when `delegation.max_spawn_depth` is raised above the default of 1 (floor 1, no ceiling); the other four remain blocked. Disable globally via `delegation.orchestrator_enabled: false`.

### Tuning Concurrency and Depth

| Config | Default | Range | Effect |
|--------|---------|-------|--------|
| `max_concurrent_children` | 3 | >=1 | Parallel batch size per `delegate_task` call |
| `max_spawn_depth` | 1 | >=1 | How many delegation levels can spawn further |

Example: running 30 parallel workers with nested subagentes:

```yaml
delegation:
  max_concurrent_children: 30
  max_spawn_depth: 2
```

- **Separate terminals** — each subagente gets its own terminal session with separate working directory and state
- **No conversation history** — subagentes see only the `goal` and `contexto` the parent agente passes when calling `delegate_task`
- **Default 50 iterations** — set `max_iterations` lower for simple tasks to save cost
- **Not durable** — `delegate_task` is synchronous and runs inside the parent turn. If the parent is interrupted (new user message, `/stop`, `/new`), all active children are cancelled (`status="interrupted"`) and their work is discarded. For work that must outlive the current turn, use `cronjob` or `terminal(background=True, notify_on_complete=True)`.

---

## Tips

**Be specific in goals.** "Fix the bug" is too vague. "Fix the TypeError in api/handlers.py line 47 where process_request() receives None from parse_body()" gives the subagente enough to work with.

**Include file paths.** Subagentes don't know your project structure. Always include absolute paths to relevant files, the project root, and the test command.

**Use delegation for contexto isolation.** Sometimes you want a fresh perspective. Delegating forces you to articulate the problem clearly, and the subagente approaches it without the assumptions that built up in your conversation.

**Check results.** Subagente summaries are just that — summaries. If a subagente says "fixed the bug and tests pass," verify by running the tests yourself or reading the diff.

---

*For the complete delegation reference — all parameters, ACP integration, and advanced configuración — see [Subagente Delegation](/guia-usuario/features/delegation).*

---