<!-- source: website/docs/guia-usuario/features/delegation.md -->
# Subagente Delegation

# Subagente Delegation

The `delegate_task` herramienta spawns child AIAgente instances with isolated contexto, restricted herramientasets, and their own terminal sessions. Each child gets a fresh conversation and works independently — only its final summary enters the parent's contexto.

## Single Task

```python
delegate_task(
    goal="Debug why tests fail",
    contexto="Error: assertion in test_foo.py line 42",
    herramientasets=["terminal", "file"]
)
```

## Parallel Batch

Up to 3 concurrent subagentes by default (configurable, no hard ceiling):

```python
delegate_task(tasks=[
    {"goal": "Research topic A", "herramientasets": ["web"]},
    {"goal": "Research topic B", "herramientasets": ["web"]},
    {"goal": "Fix the build", "herramientasets": ["terminal", "file"]}
])
```

## How Subagente Contexto Works

:::warning Critical: Subagentes Know Nothing
Subagentes start with a **completely fresh conversation**. They have zero knowledge of the parent's conversation history, prior herramienta calls, or anything discussed before delegation. The subagente's only contexto comes from the `goal` and `contexto` fields the parent agente populates when it calls `delegate_task`.
:::

This means the parent agente must pass **everything** the subagente needs in the call:

```python
# BAD - subagente has no idea what "the error" is
delegate_task(goal="Fix the error")

# GOOD - subagente has all contexto it needs
delegate_task(
    goal="Fix the TypeError in api/handlers.py",
    contexto="""The file api/handlers.py has a TypeError on line 47:
    'NoneType' object has no attribute 'get'.
    The function process_request() receives a dict from parse_body(),
    but parse_body() returns None when Content-Type is missing.
    The project is at /home/user/myproject and uses Python 3.11."""
)
```

The subagente receives a focused system prompt built from your goal and contexto, instructing it to complete the task and provide a structured summary of what it did, what it found, any files modified, and any issues encountered.

## Practical Examples

### Parallel Research

Research multiple topics simultaneously and collect summaries:

```python
delegate_task(tasks=[
    {
        "goal": "Research the current state of WebAssembly in 2025",
        "contexto": "Focus on: browser support, non-browser runtimes, language support",
        "herramientasets": ["web"]
    },
    {
        "goal": "Research the current state of RISC-V adoption in 2025",
        "contexto": "Focus on: server chips, embedded systems, software ecosystem",
        "herramientasets": ["web"]
    },
    {
        "goal": "Research quantum computing progress in 2025",
        "contexto": "Focus on: error correction breakthroughs, practical applications, key players",
        "herramientasets": ["web"]
    }
])
```

### Code Review + Fix

Delegate a review-and-fix workflow to a fresh contexto:

```python
delegate_task(
    goal="Review the autenticación module for security issues and fix any found",
    contexto="""Project at /home/user/webapp.
    Auth module files: src/auth/login.py, src/auth/jwt.py, src/auth/middleware.py.
    The project uses Flask, PyJWT, and bcrypt.
    Focus on: SQL injection, JWT validation, password handling, session management.
    Fix any issues found and run the test suite (pytest tests/auth/).""",
    herramientasets=["terminal", "file"]
)
```

### Multi-File Refactoring

Delegate a large refactoring task that would flood the parent's contexto:

```python
delegate_task(
    goal="Refactor all Python files in src/ to replace print() with proper logging",
    contexto="""Project at /home/user/myproject.
    Use the 'logging' module with logger = logging.getLogger(__name__).
    Replace print() calls with appropriate log levels:
    - print(f"Error: ...") -> logger.error(...)
    - print(f"Warning: ...") -> logger.warning(...)
    - print(f"Debug: ...") -> logger.debug(...)
    - Other prints -> logger.info(...)
    Don't change print() in test files or CLI output.
    Run pytest after to verify nothing broke.""",
    herramientasets=["terminal", "file"]
)
```

## Batch Mode Details

When you provide a `tasks` array, subagentes run in **parallel** using a thread pool:

- **Maximum concurrency:** 3 tasks by default (configurable via `delegation.max_concurrent_children` or the `DELEGATION_MAX_CONCURRENT_CHILDREN` env var; floor of 1, no hard ceiling). Batches larger than the limit return a herramienta error rather than being silently truncated.
- **Thread pool:** Uses `ThreadPoolExecutor` with the configured concurrency limit as max workers
- **Progress display:** In CLI mode, a tree-view shows herramienta calls from each subagente in real-time with per-task completion lines. In puerta de enlace mode, progress is batched and relayed to the parent's progress callback
- **Result ordering:** Results are sorted by task index to match input order regardless of completion order
- **Interrupt propagation:** Interrupting the parent (e.g., sending a new message) interrupts all active children

Single-task delegation runs directly without thread pool overhead.

## Modelo Override

You can configure a different modelo for subagentes via `config.yaml` — useful for delegating simple tasks to cheaper/faster modelos:

```yaml
# In ~/.hermes/config.yaml
delegation:
  modelo: "google/gemini-flash-2.0"    # Cheaper modelo for subagentes
  proveedor: "openrouter"              # Optional: route subagentes to a different proveedor
```

If omitted, subagentes use the same modelo as the parent.

## Herramientaset Selection Tips

The `herramientasets` parameter controls what herramientas the subagente has access to. Choose based on the task:

| Herramientaset Pattern | Use Case |
|----------------|----------|
| `["terminal", "file"]` | Code work, debugging, file editing, builds |
| `["web"]` | Research, fact-checking, documentation lookup |
| `["terminal", "file", "web"]` | Full-stack tasks (default) |
| `["file"]` | Read-only analysis, code review without execution |
| `["terminal"]` | System administration, process management |

Certain herramientasets are blocked for subagentes regardless of what you specify:
- `delegation` — blocked for leaf subagentes (the default). Retained for `role="orchestrator"` children, bounded by `max_spawn_depth` — see [Depth Limit and Nested Orchestration](#depth-limit-and-nested-orchestration) below.
- `clarify` — subagentes cannot interact with the user
- `memoria` — no writes to shared persistent memoria
- `code_execution` — children should reason step-by-step
- `send_message` — no cross-platform side effects (e.g., sending Telegram messages)

## Max Iterations

Each subagente has an iteration limit (default: 50) that controls how many herramienta-calling turns it can take:

```python
delegate_task(
    goal="Quick file check",
    contexto="Check if /etc/nginx/nginx.conf exists and print its first 10 lines",
    max_iterations=10  # Simple task, don't need many turns
)
```

## Child Timeout

By default there is **no wall-clock timeout** on subagentes. Children fail only from what they're actually doing — API errors, herramienta errors, or hitting their iteration budget — never from a delegation-level stopwatch. Earlier releases shipped a hard cap (300s, later 600s), which kept killing legitimately busy children mid-task: deep code reviews, large research fan-outs, and slow reasoning modelos routinely need more than 10 minutes while making steady progress the whole time.

Genuinely stuck children are still detected: the heartbeat staleness monitor stops refreshing the parent's activity when a child makes no progress (no API calls, no herramienta starts), letting the puerta de enlace inactivity timeout fire on a truly wedged worker.

If you want a hard cap anyway (e.g. cost control on unattended cron-driven delegation), opt in per-install:

```yaml
delegation:
  child_timeout_seconds: 0     # default: 0 = no timeout
  # child_timeout_seconds: 1800  # opt-in hard cap (floor 30s)
```

A positive value enforces a hard wall-clock limit on each child; `0` or a negative value disables it.

:::tip Diagnostic dump on zero-call timeout
With a hard cap configured, if a subagente times out having made **zero** API calls (usually: proveedor unreachable, auth failure, or herramienta-schema rejection), `delegate_task` writes a structured diagnostic to `~/.hermes/logs/subagente-timeout-<session>-<timestamp>.log` containing the subagente's config snapshot, credential-resolution trace, and any early error messages. Much easier to root-cause than the previous silent-timeout behavior.
:::

## Monitoring Running Subagentes (`/agentes`)

The TUI ships a `/agentes` overlay (alias `/tasks`) that turns recursive `delegate_task` fan-out into a first-class audit surface:

- Live tree view of running and recently-finished subagentes, grouped by parent
- Per-branch cost, token, and file-touched rollups
- Kill and pause controls — cancel a specific subagente mid-flight without interrupting its siblings
- Post-hoc review: step through each subagente's turn-by-turn history even after they've returned to the parent

The classic CLI just prints `/agentes` as a text summary; the TUI is where the overlay shines. See [TUI — Slash commands](/guia-usuario/tui#slash-commands).

## Depth Limit and Nested Orchestration

By default, delegation is **flat**: a parent (depth 0) spawns children (depth 1), and those children cannot delegate further. This prevents runaway recursive delegation.

For multi-stage workflows (research → synthesis, or parallel orchestration over sub-problems), a parent can spawn **orchestrator** children that *can* delegate their own workers:

```python
delegate_task(
    goal="Survey three code review approaches and recommend one",
    role="orchestrator",  # Allows this child to spawn its own workers
    contexto="...",
)
```

- `role="leaf"` (default): child cannot delegate further — identical to the flat-delegation behavior.
- `role="orchestrator"`: child retains the `delegation` herramientaset. Gated by `delegation.max_spawn_depth` (default **1** = flat, so `role="orchestrator"` is a no-op at defaults). Raise `max_spawn_depth` to 2 to allow orchestrator children to spawn leaf grandchildren; 3+ for deeper trees. There is no upper ceiling — cost is the practical limit.
- `delegation.orchestrator_enabled: false`: global kill switch that forces every child to `leaf` regardless of the `role` parameter.

**Cost warning:** With `max_spawn_depth: 3` and `max_concurrent_children: 3`, the tree can reach 3×3×3 = 27 concurrent leaf agentes. Each extra level multiplies spend — raise `max_spawn_depth` intentionally.

## Lifetime and Durability

:::warning delegate_task is synchronous — not durable
`delegate_task` runs **inside the parent's current turn**. It blocks the parent until every child finishes (or is cancelled). It is **not** a background job queue:

- If the parent is interrupted (user sends a new message, `/stop`, `/new`), all active children are cancelled and return `status="interrupted"`. Their in-progress work is discarded.
- Children do **not** continue running after the parent turn ends.
- Cancelled children return a structured result (`status="interrupted"`, `exit_reason="interrupted"`), but because the parent was interrupted too, that result often never makes it into a user-visible reply.

For **durable long-running work** that must survive interrupts or outlive the current turn, use:

- `cronjob` (action=`create`) — schedules a separate agente run; immune to parent-turn interrupts.
- `terminal(background=True, notify_on_complete=True)` — long-running shell commands that keep running while the agente does other things.
:::

## Key Properties

- Each subagente gets its **own terminal session** (separate from the parent)
- **Nested delegation is opt-in** — only `role="orchestrator"` children can delegate further, and only when `max_spawn_depth` is raised from its default of 1 (flat). Disable globally with `orchestrator_enabled: false`.
- Leaf subagentes **cannot** call: `delegate_task`, `clarify`, `memoria`, `send_message`, `execute_code`. Orchestrator subagentes retain `delegate_task` but still cannot use the other four.
- **Interrupt propagation** — interrupting the parent interrupts all active children (including grandchildren under orchestrators)
- Only the final summary enters the parent's contexto, keeping token usage efficient
- Subagentes inherit the parent's **API key, proveedor configuración, and credential pool** (enabling key rotation on rate limits)

## Delegation vs execute_code

| Factor | delegate_task | execute_code |
|--------|--------------|-------------|
| **Reasoning** | Full LLM reasoning loop | Just Python code execution |
| **Contexto** | Fresh isolated conversation | No conversation, just script |
| **Herramienta access** | All non-blocked herramientas with reasoning | 7 herramientas via RPC, no reasoning |
| **Parallelism** | 3 concurrent subagentes by default (configurable) | Single script |
| **Best for** | Complex tasks needing judgment | Mechanical multi-step pipelines |
| **Token cost** | Higher (full LLM loop) | Lower (only stdout returned) |
| **User interaction** | None (subagentes can't clarify) | None |

**Rule of thumb:** Use `delegate_task` when the subtask requires reasoning, judgment, or multi-step problem solving. Use `execute_code` when you need mechanical data processing or scripted workflows.

## Configuración

```yaml
# In ~/.hermes/config.yaml
delegation:
  max_iterations: 50                        # Max turns per child (default: 50)
  # max_concurrent_children: 3              # Parallel children per batch (default: 3)
  # max_spawn_depth: 1                      # Tree depth (floor 1, no ceiling, default 1 = flat). Raise to 2 to allow orchestrator children to spawn leaves; 3+ for deeper trees.
  # orchestrator_enabled: true              # Disable to force all children to leaf role.
  modelo: "google/gemini-3-flash-preview"             # Optional proveedor/modelo override
  proveedor: "openrouter"                             # Optional built-in proveedor
  api_mode: anthropic_messages                       # optional; auto-detected from base_url for anthropic_messages endpoints

# Or use a direct custom endpoint instead of proveedor:
delegation:
  modelo: "qwen2.5-coder"
  base_url: "http://localhost:1234/v1"
  api_key: "local-key"
  # api_mode: "anthropic_messages"  # Optional. Wire protocol override for base_url ("chat_completions", "codex_responses", or "anthropic_messages"). Empty = auto-detect from URL (e.g. /anthropic suffix). Set explicitly for endpoints the heuristic can't classify (Azure AI Foundry, MiniMax, Zhipu GLM, LiteLLM proxies, …).
```

When `base_url` points at an Anthropic-compatible endpoint — for example a path ending in `/anthropic`, an Azure Foundry Claude route, or a MiniMax `/anthropic` proxy — `api_mode` is auto-detected as `anthropic_messages` so the subagente uses the right wire format without you setting anything. Set `api_mode` explicitly when the auto-detection guess is wrong (rare).

:::tip
The agente handles delegation automatically based on the task complexity. You don't need to explicitly ask it to delegate — it will do so when it makes sense.
:::

---