<!-- source: website/docs/guia-desarrollador/agente-loop.md -->
# Agente Loop Internals

# Agente Loop Internals

The core orchestration engine is `run_agente.py`'s `AIAgente` class — a large file that handles everything from prompt assembly to herramienta dispatch to proveedor failover.

## Core Responsibilities

`AIAgente` is responsible for:

- Assembling the effective system prompt and herramienta schemas via `prompt_builder.py`
- Selecting the correct proveedor/API mode (chat_completions, codex_responses, anthropic_messages)
- Making interruptible modelo calls with cancellation support
- Executing herramienta calls (sequentially or concurrently via thread pool)
- Maintaining conversation history in OpenAI message format
- Handling compression, retries, and fallback modelo switching
- Tracking iteration budgets across parent and child agentes
- Flushing persistent memoria before contexto is lost

## Two Entry Points

```python
# Simple interface — returns final response string
response = agente.chat("Fix the bug in main.py")

# Full interface — returns dict with messages, metadata, usage stats
result = agente.run_conversation(
    user_message="Fix the bug in main.py",
    system_message=None,           # auto-built if omitted
    conversation_history=None,      # auto-loaded from session if omitted
    task_id="task_abc123"
)
```

`chat()` is a thin wrapper around `run_conversation()` that extracts the `final_response` field from the result dict.

## API Modes

Hermes supports three API execution modes, resolved from proveedor selection, explicit args, and base URL heuristics:

| API mode | Used for | Client type |
|----------|----------|-------------|
| `chat_completions` | OpenAI-compatible endpoints (OpenRouter, custom, most proveedors) | `openai.OpenAI` |
| `codex_responses` | OpenAI Codex / Responses API | `openai.OpenAI` with Responses format |
| `anthropic_messages` | Native Anthropic Messages API | `anthropic.Anthropic` via adapter |

The mode determines how messages are formatted, how herramienta calls are structured, how responses are parsed, and how caching/streaming works. All three converge on the same internal message format (OpenAI-style `role`/`content`/`herramienta_calls` dicts) before and after API calls.

**Mode resolution order:**
1. Explicit `api_mode` constructor arg (highest priority)
2. Proveedor-specific detection (e.g., `anthropic` proveedor → `anthropic_messages`)
3. Base URL heuristics (e.g., `api.anthropic.com` → `anthropic_messages`)
4. Default: `chat_completions`

## Turn Lifecycle

Each iteration of the agente loop follows this sequence:

```text
run_conversation()
  1. Generate task_id if not provided
  2. Append user message to conversation history
  3. Build or reuse cached system prompt (prompt_builder.py)
  4. Check if preflight compression is needed (>50% contexto)
  5. Build API messages from conversation history
     - chat_completions: OpenAI format as-is
     - codex_responses: convert to Responses API input items
     - anthropic_messages: convert via anthropic_adapter.py
  6. Inject ephemeral prompt layers (budget warnings, contexto pressure)
  7. Apply prompt caching markers if on Anthropic
  8. Make interruptible API call (_interruptible_api_call)
  9. Parse response:
     - If herramienta_calls: execute them, append results, loop back to step 5
     - If text response: persist session, flush memoria if needed, return
```

### Message Format

All messages use OpenAI-compatible format internally:

```python
{"role": "system", "content": "..."}
{"role": "user", "content": "..."}
{"role": "assistant", "content": "...", "herramienta_calls": [...]}
{"role": "herramienta", "herramienta_call_id": "...", "content": "..."}
```

Reasoning content (from modelos that support extended thinking) is stored in `assistant_msg["reasoning"]` and optionally displayed via the `reasoning_callback`.

### Message Alternation Rules

The agente loop enforces strict message role alternation:

- After the system message: `User → Assistant → User → Assistant → ...`
- During herramienta calling: `Assistant (with herramienta_calls) → Herramienta → Herramienta → ... → Assistant`
- **Never** two assistant messages in a row
- **Never** two user messages in a row
- **Only** `herramienta` role can have consecutive entries (parallel herramienta results)

Proveedors validate these sequences and will reject malformed histories.

## Interruptible API Calls

API requests are wrapped in `_interruptible_api_call()` which runs the actual HTTP call in a background thread while monitoring an interrupt event:

```text
┌────────────────────────────────────────────────────┐
│  Main thread                  API thread           │
│                                                    │
│   wait on:                     HTTP POST           │
│    - response ready     ───▶   to proveedor         │
│    - interrupt event                               │
│    - timeout                                       │
└────────────────────────────────────────────────────┘
```

When interrupted (user sends new message, `/stop` command, or signal):
- The API thread is abandoned (response discarded)
- The agente can process the new input or shut down cleanly
- No partial response is injected into conversation history

## Herramienta Execution

### Sequential vs Concurrent

When the modelo returns herramienta calls:

- **Single herramienta call** → executed directly in the main thread
- **Multiple herramienta calls** → executed concurrently via `ThreadPoolExecutor`
  - Exception: herramientas marked as interactive (e.g., `clarify`) force sequential execution
  - Results are reinserted in the original herramienta call order regardless of completion order

### Execution Flow

```text
for each herramienta_call in response.herramienta_calls:
    1. Resolve handler from herramientas/registry.py
    2. Fire pre_herramienta_call complemento hook
    3. Check if dangerous command (herramientas/approval.py)
       - If dangerous: invoke approval_callback, wait for user
    4. Execute handler with args + task_id
    5. Fire post_herramienta_call complemento hook
    6. Append {"role": "herramienta", "content": result} to history
```

### Agente-Level Herramientas

Some herramientas are intercepted by `run_agente.py` *before* reaching `handle_function_call()`:

| Herramienta | Why intercepted |
|------|--------------------|
| `todo` | Reads/writes agente-local task state |
| `memoria` | Writes to persistent memoria files with character limits |
| `session_search` | Queries session history via the agente's session DB |
| `delegate_task` | Spawns subagente(s) with isolated contexto |

These herramientas modify agente state directly and return synthetic herramienta results without going through the registry.

## Callback Surfaces

`AIAgente` supports platform-specific callbacks that enable real-time progress in the CLI, puerta de enlace, and ACP integrations:

| Callback | When fired | Used by |
|----------|-----------|---------|
| `herramienta_progress_callback` | Before/after each herramienta execution | CLI spinner, puerta de enlace progress messages |
| `thinking_callback` | When modelo starts/stops thinking | CLI "thinking..." indicator |
| `reasoning_callback` | When modelo returns reasoning content | CLI reasoning display, puerta de enlace reasoning blocks |
| `clarify_callback` | When `clarify` herramienta is called | CLI input prompt, puerta de enlace interactive message |
| `step_callback` | After each complete agente turn | Puerta de enlace step tracking, ACP progress |
| `stream_delta_callback` | Each streaming token (when enabled) | CLI streaming display |
| `herramienta_gen_callback` | When herramienta call is parsed from stream | CLI herramienta preview in spinner |
| `status_callback` | State changes (thinking, executing, etc.) | ACP status updates |

## Budget and Fallback Behavior

### Iteration Budget

The agente tracks iterations via `IterationBudget`:

- Default: 90 iterations (configurable via `agente.max_turns`)
- Each agente gets its own budget. Subagentes get independent budgets capped at `delegation.max_iterations` (default 50) — total iterations across parent + subagentes can exceed the parent's cap
- At 100%, the agente stops and returns a summary of work done

### Fallback Modelo

When the primary modelo fails (429 rate limit, 5xx server error, 401/403 auth error):

1. Check `fallback_proveedors` list in config
2. Try each fallback in order
3. On success, continue the conversation with the new proveedor
4. On 401/403, attempt credential refresh before failing over

The fallback system also covers auxiliary tasks independently — vision, compression, and web extraction each have their own fallback chain configurable via the `auxiliary.*` config section.

## Compression and Persistence

### When Compression Triggers

- **Preflight** (before API call): If conversation exceeds 50% of modelo's contexto window
- **Puerta de enlace auto-compression**: If conversation exceeds 85% (more aggressive, runs between turns)

### What Happens During Compression

1. Memoria is flushed to disk first (preventing data loss)
2. Middle conversation turns are summarized into a compact summary
3. The last N messages are preserved intact (`compression.protect_last_n`, default: 20)
4. Herramienta call/result message pairs are kept together (never split)
5. A new session lineage ID is generated (compression creates a "child" session)

### Session Persistence

After each turn:
- Messages are saved to the session store (SQLite via `hermes_state.py`)
- Memoria changes are flushed to `MEMORY.md` / `USER.md`
- The session can be resumed later via `/resume` or `hermes chat --resume`

## Key Source Files

| File | Purpose |
|------|---------|
| `run_agente.py` | AIAgente class — the complete agente loop |
| `agente/prompt_builder.py` | System prompt assembly from memoria, habilidads, contexto files, personality |
| `agente/contexto_engine.py` | ContextoEngine ABC — pluggable contexto management |
| `agente/contexto_compressor.py` | Default engine — lossy summarization algorithm |
| `agente/prompt_caching.py` | Anthropic prompt caching markers and cache metrics |
| `agente/auxiliary_client.py` | Auxiliary LLM client for side tasks (vision, summarization) |
| `modelo_herramientas.py` | Herramienta schema collection, `handle_function_call()` dispatch |

## Related Docs

- [Proveedor Runtime Resolution](./proveedor-runtime.md)
- [Prompt Assembly](./prompt-assembly.md)
- [Contexto Compression & Prompt Caching](./contexto-compression-and-caching.md)
- [Herramientas Runtime](./herramientas-runtime.md)
- [Architecture Overview](./architecture.md)

---