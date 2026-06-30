<!-- source: website/docs/guia-usuario/features/hooks.md -->
# Event Hooks

# Event Hooks

Hermes has three hook systems that run custom code at key lifecycle points:

| System | Registered via | Runs in | Use case |
|--------|---------------|---------|----------|
| **[Puerta de enlace hooks](#puerta de enlace-event-hooks)** | `HOOK.yaml` + `handler.py` in `~/.hermes/hooks/` | Puerta de enlace only | Logging, alerts, webhooks |
| **[Complemento hooks](#complemento-hooks)** | `ctx.register_hook()` in a [complemento](/guia-usuario/features/complementos) | CLI + Puerta de enlace | Herramienta interception, metrics, guardrails |
| **[Shell hooks](#shell-hooks)** | `hooks:` block in `~/.hermes/config.yaml` pointing at shell scripts | CLI + Puerta de enlace | Drop-in scripts for blocking, auto-formatting, contexto injection |

All three systems are non-blocking — errors in any hook are caught and logged, never crashing the agente.

## Puerta de enlace Event Hooks

Puerta de enlace hooks fire automatically during puerta de enlace operation (Telegram, Discord, Slack, WhatsApp, Teams) without blocking the main agente pipeline.

### Creating a Hook

Each hook is a directory under `~/.hermes/hooks/` containing two files:

```text
~/.hermes/hooks/
└── my-hook/
    ├── HOOK.yaml      # Declares which events to listen for
    └── handler.py     # Python handler function
```

#### HOOK.yaml

```yaml
name: my-hook
description: Log all agente activity to a file
events:
  - agente:start
  - agente:end
  - agente:step
```

The `events` list determines which events trigger your handler. You can subscribe to any combination of events, including wildcards like `command:*`.

#### handler.py

```python
import json
from datetime import datetime
from pathlib import Path

LOG_FILE = Path.home() / ".hermes" / "hooks" / "my-hook" / "activity.log"

async def handle(event_type: str, contexto: dict):
    """Called for each subscribed event. Must be named 'handle'."""
    entry = {
        "timestamp": datetime.now().isoformat(),
        "event": event_type,
        **contexto,
    }
    with open(LOG_FILE, "a") as f:
        f.write(json.dumps(entry) + "\n")
```

**Handler rules:**
- Must be named `handle`
- Receives `event_type` (string) and `contexto` (dict)
- Can be `async def` or regular `def` — both work
- Errors are caught and logged, never crashing the agente

### Available Events

| Event | When it fires | Contexto keys |
|-------|---------------|--------------|
| `puerta de enlace:startup` | Puerta de enlace process starts | `platforms` (list of active platform names) |
| `session:start` | New messaging session created | `platform`, `user_id`, `session_id`, `session_key` |
| `session:end` | Session ended (before reset) | `platform`, `user_id`, `session_key` |
| `session:reset` | User ran `/new` or `/reset` | `platform`, `user_id`, `session_key` |
| `agente:start` | Agente begins processing a message | `platform`, `user_id`, `session_id`, `message` |
| `agente:step` | Each iteration of the herramienta-calling loop | `platform`, `user_id`, `session_id`, `iteration`, `herramienta_names` |
| `agente:end` | Agente finishes processing | `platform`, `user_id`, `session_id`, `message`, `response` |
| `command:*` | Any slash command executed | `platform`, `user_id`, `command`, `args` |

#### Wildcard Matching

Handlers registered for `command:*` fire for any `command:` event (`command:modelo`, `command:reset`, etc.). Monitor all slash commands with a single subscription.

### Ejemplos

#### Telegram Alert on Long Tasks

Send yourself a message when the agente takes more than 10 steps:

```yaml
# ~/.hermes/hooks/long-task-alert/HOOK.yaml
name: long-task-alert
description: Alert when agente is taking many steps
events:
  - agente:step
```

```python
# ~/.hermes/hooks/long-task-alert/handler.py
import os
import httpx

THRESHOLD = 10
BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
CHAT_ID = os.getenv("TELEGRAM_HOME_CHANNEL")

async def handle(event_type: str, contexto: dict):
    iteration = contexto.get("iteration", 0)
    if iteration == THRESHOLD and BOT_TOKEN and CHAT_ID:
        herramientas = ", ".join(contexto.get("herramienta_names", []))
        text = f"⚠️ Agente has been running for {iteration} steps. Last herramientas: {herramientas}"
        async with httpx.AsyncClient() as client:
            await client.post(
                f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage",
                json={"chat_id": CHAT_ID, "text": text},
            )
```

#### Command Usage Logger

Track which slash commands are used:

```yaml
# ~/.hermes/hooks/command-logger/HOOK.yaml
name: command-logger
description: Log slash command usage
events:
  - command:*
```

```python
# ~/.hermes/hooks/command-logger/handler.py
import json
from datetime import datetime
from pathlib import Path

LOG = Path.home() / ".hermes" / "logs" / "command_usage.jsonl"

def handle(event_type: str, contexto: dict):
    LOG.parent.mkdir(parents=True, exist_ok=True)
    entry = {
        "ts": datetime.now().isoformat(),
        "command": contexto.get("command"),
        "args": contexto.get("args"),
        "platform": contexto.get("platform"),
        "user": contexto.get("user_id"),
    }
    with open(LOG, "a") as f:
        f.write(json.dumps(entry) + "\n")
```

#### Session Start Webhook

POST to an external service on new sessions:

```yaml
# ~/.hermes/hooks/session-webhook/HOOK.yaml
name: session-webhook
description: Notify external service on new sessions
events:
  - session:start
  - session:reset
```

```python
# ~/.hermes/hooks/session-webhook/handler.py
import httpx

WEBHOOK_URL = "https://your-service.example.com/hermes-events"

async def handle(event_type: str, contexto: dict):
    async with httpx.AsyncClient() as client:
        await client.post(WEBHOOK_URL, json={
            "event": event_type,
            **contexto,
        }, timeout=5)
```

### Tutorial: BOOT.md — Run a Startup Checklist on Every Puerta de enlace Boot

A popular pattern from the community: drop a Markdown checklist at `~/.hermes/BOOT.md`, and have the agente run it once every time the puerta de enlace starts. Useful for "on every boot, check overnight cron failures and ping me on Discord if anything failed," or "summarize the last 24h of deploy.log and post it to Slack #ops."

This tutorial shows how to build it yourself as a user-defined hook. Hermes does not ship a built-in BOOT.md hook — you wire up exactly the behavior you want.

#### What we're building

1. A file at `~/.hermes/BOOT.md` with natural-language startup instructions.
2. A puerta de enlace hook that fires on `puerta de enlace:startup`, spawns a one-shot agente with your puerta de enlace's resolved modelo/credentials, and runs the BOOT.md instructions.
3. A `[SILENT]` convention so the agente can opt out of sending a message when there's nothing to report.

#### Step 1: Write your checklist

Create `~/.hermes/BOOT.md`. Write it as if you were giving instructions to a human assistant:

```markdown
# Startup Checklist

1. Run `hermes cron list` and check if any scheduled jobs failed overnight.
2. If any failed, send a summary to Discord #ops using the `send_message` herramienta.
3. Check if `/opt/app/deploy.log` has any ERROR lines from the last 24 hours. If yes, summarize them and include in the same Discord message.
4. If nothing went wrong, reply with only `[SILENT]` so no message is sent.
```

The agente sees this as part of its prompt, so anything you can describe in plain language works — herramienta calls, shell commands, sending messages, summarizing files.

#### Step 2: Create the hook

```text
~/.hermes/hooks/boot-md/
├── HOOK.yaml
└── handler.py
```

**`~/.hermes/hooks/boot-md/HOOK.yaml`**

```yaml
name: boot-md
description: Run ~/.hermes/BOOT.md on puerta de enlace startup
events:
  - puerta de enlace:startup
```

**`~/.hermes/hooks/boot-md/handler.py`**

```python
"""Run ~/.hermes/BOOT.md on every puerta de enlace startup."""

import logging
import threading
from pathlib import Path

logger = logging.getLogger("hooks.boot-md")

BOOT_FILE = Path.home() / ".hermes" / "BOOT.md"


def _build_prompt(content: str) -> str:
    return (
        "You are running a startup boot checklist. Follow the instructions "
        "below exactly.\n\n"
        "---\n"
        f"{content}\n"
        "---\n\n"
        "Execute each instruction. Use the send_message herramienta to deliver any "
        "messages to platforms like Discord or Slack.\n"
        "If nothing needs attention and there is nothing to report, reply "
        "with ONLY: [SILENT]"
    )


def _run_boot_agente(content: str) -> None:
    """Spawn a one-shot agente and execute the checklist.

    Uses the puerta de enlace's resolved modelo and runtime credentials so this works
    against custom endpoints, aggregators, and OAuth-based proveedors alike.
    """
    try:
        from puerta de enlace.run import _resolve_puerta de enlace_modelo, _resolve_runtime_agente_kwargs
        from run_agente import AIAgente

        agente = AIAgente(
            modelo=_resolve_puerta de enlace_modelo(),
            **_resolve_runtime_agente_kwargs(),
            platform="puerta de enlace",
            quiet_mode=True,
            skip_contexto_files=True,
            skip_memoria=True,
            max_iterations=20,
        )
        result = agente.run_conversation(_build_prompt(content))
        response = (result.get("final_response", "") or "").strip()
        if response.upper() not in {"[SILENT]", "SILENT", "NO_REPLY", "NO REPLY"}:
            logger.info("boot-md completed: %s", response[:200])
        else:
            logger.info("boot-md completed (nothing to report)")
    except Exception as e:
        logger.error("boot-md agente failed: %s", e)


async def handle(event_type: str, contexto: dict) -> None:
    if not BOOT_FILE.exists():
        return
    content = BOOT_FILE.read_text(encoding="utf-8").strip()
    if not content:
        return

    logger.info("Running BOOT.md (%d chars)", len(content))

    # Background thread so puerta de enlace startup isn't blocked on a full agente turn.
    thread = threading.Thread(
        target=_run_boot_agente,
        args=(content,),
        name="boot-md",
        daemon=True,
    )
    thread.start()
```

The two key lines:

- `_resolve_puerta de enlace_modelo()` reads the puerta de enlace's currently-configured modelo.
- `_resolve_runtime_agente_kwargs()` resolves proveedor credentials the same way a normal puerta de enlace turn does — including API keys, base URLs, OAuth tokens, and credential pools.

Without these, a bare `AIAgente()` falls back to built-in defaults and will 401 against any non-default endpoint.

#### Step 3: Test it

Restart the puerta de enlace:

```bash
hermes puerta de enlace restart
```

Watch the logs:

```bash
hermes logs --follow --level INFO | grep boot-md
```

You should see `Running BOOT.md (N chars)` followed by either `boot-md completed: ...` (summary of what the agente did) or `boot-md completed (nothing to report)` when the agente replied with an exact silence token such as `[SILENT]`.

Delete `~/.hermes/BOOT.md` to disable the checklist — the hook stays loaded but silently skips when the file isn't there.

#### Extending the pattern

- **Schedule-aware checklists:** key off `datetime.now().weekday()` inside BOOT.md's instructions ("if it's Monday, also check the weekly deploy log"). The instructions are free-form text, so anything the agente can reason about is fair game.
- **Multiple checklists:** point the hook at a different file (`STARTUP.md`, `MORNING.md`, etc.) and register separate hook directories for each.
- **Non-agente variant:** if you don't need a full agente loop, skip `AIAgente` entirely and have the handler post a fixed notification directly via `httpx`. Cheaper, faster, and has no proveedor dependency.

#### Why this isn't a built-in

An earlier version of Hermes shipped this as a built-in hook and silently spawned an agente with bare defaults on every puerta de enlace boot. That surprised users with custom endpoints and made the feature invisible to users who didn't know it was running. Keeping it as a documented pattern — built by you, in your hooks directory — means you see exactly what it does and opt in by writing the files.

### How It Works

1. On puerta de enlace startup, `HookRegistry.discover_and_load()` scans `~/.hermes/hooks/`
2. Each subdirectory with `HOOK.yaml` + `handler.py` is loaded dynamically
3. Handlers are registered for their declared events
4. At each lifecycle point, `hooks.emit()` fires all matching handlers
5. Errors in any handler are caught and logged — a broken hook never crashes the agente

:::info
Puerta de enlace hooks only fire in the **puerta de enlace** (Telegram, Discord, Slack, WhatsApp, Teams). The CLI does not load puerta de enlace hooks. For hooks that work everywhere, use [complemento hooks](#complemento-hooks).
:::

## Complemento Hooks

[Complementos](/guia-usuario/features/complementos) can register hooks that fire in **both CLI and puerta de enlace** sessions. These are registered programmatically via `ctx.register_hook()` in your complemento's `register()` function.

For complemento packaging and registration details, see
the [Complementos guide](/docs/guia-usuario/features/complementos).

```python
def register(ctx):
    ctx.register_hook("pre_herramienta_call", my_herramienta_observer)
    ctx.register_hook("post_herramienta_call", my_herramienta_logger)
    ctx.register_hook("pre_llm_call", my_memoria_callback)
    ctx.register_hook("post_llm_call", my_sync_callback)
    ctx.register_hook("on_session_start", my_init_callback)
    ctx.register_hook("on_session_end", my_cleanup_callback)
```

**General rules for all hooks:**

- Callbacks receive **keyword arguments**. Always accept `**kwargs` for forward compatibility — new parameters may be added in future versions without breaking your complemento.
- If a callback **crashes**, it's logged and skipped. Other hooks and the agente continue normally. A misbehaving complemento can never break the agente.
- Two hooks' return values affect behavior: [`pre_herramienta_call`](#pre_herramienta_call) can **block** the herramienta, and [`pre_llm_call`](#pre_llm_call) can **inject contexto** into the LLM call. All other hooks are fire-and-forget observers.
- Observer callbacks receive `telemetry_schema_version` automatically. When present, `turn_id`, `api_request_id`, `task_id`, `session_id`, and `api_call_count` are separate correlation fields. Treat `api_request_id` as an opaque identifier; do not parse its string format.

### Quick reference

| Hook | Fires when | Returns |
|------|-----------|---------|
| [`pre_herramienta_call`](#pre_herramienta_call) | Before any herramienta executes | `{"action": "block", "message": str}` to veto the call |
| [`post_herramienta_call`](#post_herramienta_call) | After any herramienta returns | ignored |
| [`pre_llm_call`](#pre_llm_call) | Once per turn, before the herramienta-calling loop | `{"contexto": str}` to prepend contexto to the user message |
| [`post_llm_call`](#post_llm_call) | Once per turn, after the herramienta-calling loop | ignored |
| [`on_session_start`](#on_session_start) | New session created (first turn only) | ignored |
| [`on_session_end`](#on_session_end) | Session ends | ignored |
| [`on_session_finalize`](#on_session_finalize) | CLI/puerta de enlace tears down an active session (flush, save, stats) | ignored |
| [`on_session_reset`](#on_session_reset) | Puerta de enlace swaps in a fresh session key (e.g. `/new`, `/reset`) | ignored |
| [`subagente_start`](#subagente_start) | A `delegate_task` child has been constructed and is about to run | ignored |
| [`subagente_stop`](#subagente_stop) | A `delegate_task` child has exited | ignored |
| [`pre_puerta de enlace_dispatch`](#pre_puerta de enlace_dispatch) | Puerta de enlace received a user message, before auth + dispatch | `{"action": "skip" \| "rewrite" \| "allow", ...}` to influence flow |
| [`pre_approval_request`](#pre_approval_request) | Dangerous command needs user approval, before the prompt/notification is sent | ignored |
| [`post_approval_response`](#post_approval_response) | User responded to an approval prompt (or it timed out) | ignored |
| [`transform_herramienta_result`](#transform_herramienta_result) | After any herramienta returns, before the result is handed back to the modelo | `str` to replace the result, `None` to leave unchanged |
| [`transform_terminal_output`](#transform_terminal_output) | Inside the `terminal` herramienta, before truncation/ANSI-strip/redact | `str` to replace the raw output, `None` to leave unchanged |
| [`transform_llm_output`](#transform_llm_output) | After the herramienta-calling loop completes, before the final response is delivered | `str` to replace the response text, `None`/empty to leave unchanged |

---

### `pre_herramienta_call`

Fires **immediately before** every herramienta execution — built-in herramientas and complemento herramientas alike.

**Callback signature:**

```python
def my_callback(herramienta_name: str, args: dict, task_id: str, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `herramienta_name` | `str` | Name of the herramienta about to execute (e.g. `"terminal"`, `"web_search"`, `"read_file"`) |
| `args` | `dict` | The arguments the modelo passed to the herramienta |
| `task_id` | `str` | Session/task identifier. Empty string if not set. |

**Fires:** In `modelo_herramientas.py`, inside `handle_function_call()`, before the herramienta's handler runs. Fires once per herramienta call — if the modelo calls 3 herramientas in parallel, this fires 3 times.

**Return value — veto the call:**

```python
return {"action": "block", "message": "Reason the herramienta call was blocked"}
```

The agente short-circuits the herramienta with `message` as the error returned to the modelo. The first matching block directive wins (Python complementos registered first, then shell hooks). Any other return value is ignored, so existing observer-only callbacks keep working unchanged.

**Use cases:** Logging, audit trails, herramienta call counters, blocking dangerous operations, rate limiting, per-user policy enforcement.

**Example — herramienta call audit log:**

```python
import json, logging
from datetime import datetime

logger = logging.getLogger(__name__)

def audit_herramienta_call(herramienta_name, args, task_id, **kwargs):
    logger.info("TOOL_CALL session=%s herramienta=%s args=%s",
                task_id, herramienta_name, json.dumps(args)[:200])

def register(ctx):
    ctx.register_hook("pre_herramienta_call", audit_herramienta_call)
```

**Example — warn on dangerous herramientas:**

```python
DANGEROUS = {"terminal", "write_file", "patch"}

def warn_dangerous(herramienta_name, **kwargs):
    if herramienta_name in DANGEROUS:
        print(f"⚠ Executing potentially dangerous herramienta: {herramienta_name}")

def register(ctx):
    ctx.register_hook("pre_herramienta_call", warn_dangerous)
```

---

### `post_herramienta_call`

Fires **immediately after** every herramienta execution returns.

**Callback signature:**

```python
def my_callback(herramienta_name: str, args: dict, result: str, task_id: str,
                duration_ms: int, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `herramienta_name` | `str` | Name of the herramienta that just executed |
| `args` | `dict` | The arguments the modelo passed to the herramienta |
| `result` | `str` | The herramienta's return value (always a JSON string) |
| `task_id` | `str` | Session/task identifier. Empty string if not set. |
| `duration_ms` | `int` | How long the herramienta's dispatch took, in milliseconds (measured with `time.monotonic()` around `registry.dispatch()`). |

**Fires:** In `modelo_herramientas.py`, inside `handle_function_call()`, after the herramienta's handler returns. Fires once per herramienta call. Does **not** fire if the herramienta raised an unhandled exception (the error is caught and returned as an error JSON string instead, and `post_herramienta_call` fires with that error string as `result`).

**Return value:** Ignored.

**Use cases:** Logging herramienta results, metrics collection, tracking herramienta success/failure rates, latency panels, per-herramienta budget alerts, sending notifications when specific herramientas complete.

**Example — track herramienta usage metrics:**

```python
from collections import Counter, defaultdict
import json

_herramienta_counts = Counter()
_error_counts = Counter()
_latency_ms = defaultdict(list)

def track_metrics(herramienta_name, result, duration_ms=0, **kwargs):
    _herramienta_counts[herramienta_name] += 1
    _latency_ms[herramienta_name].append(duration_ms)
    try:
        parsed = json.loads(result)
        if "error" in parsed:
            _error_counts[herramienta_name] += 1
    except (json.JSONDecodeError, TypeError):
        pass

def register(ctx):
    ctx.register_hook("post_herramienta_call", track_metrics)
```

---

### `pre_llm_call`

Fires **once per turn**, before the herramienta-calling loop begins. This is the **only hook whose return value is used** — it can inject contexto into the current turn's user message.

**Callback signature:**

```python
def my_callback(session_id: str, user_message: str, conversation_history: list,
                is_first_turn: bool, modelo: str, platform: str, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `session_id` | `str` | Unique identifier for the current session |
| `user_message` | `str` | The user's original message for this turn (before any habilidad injection) |
| `conversation_history` | `list` | Copy of the full message list (OpenAI format: `[{"role": "user", "content": "..."}]`) |
| `is_first_turn` | `bool` | `True` if this is the first turn of a new session, `False` on subsequent turns |
| `modelo` | `str` | The modelo identifier (e.g. `"anthropic/claude-sonnet-4.6"`) |
| `platform` | `str` | Where the session is running: `"cli"`, `"telegram"`, `"discord"`, etc. |

**Fires:** In `run_agente.py`, inside `run_conversation()`, after contexto compression but before the main `while` loop. Fires once per `run_conversation()` call (i.e. once per user turn), not once per API call within the herramienta loop.

**Return value:** If the callback returns a dict with a `"contexto"` key, or a plain non-empty string, the text is appended to the current turn's user message. Return `None` for no injection.

```python
# Inject contexto
return {"contexto": "Recalled memories:\n- User likes Python\n- Working on hermes-agente"}

# Plain string (equivalent)
return "Recalled memories:\n- User likes Python"

# No injection
return None
```

**Where contexto is injected:** Always the **user message**, never the system prompt. This preserves the prompt cache — the system prompt stays identical across turns, so cached tokens are reused. The system prompt is Hermes's territory (modelo guidance, herramienta enforcement, personality, habilidads). Complementos contribute contexto alongside the user's input.

All injected contexto is **ephemeral** — added at API call time only. The original user message in the conversation history is never mutated, and nothing is persisted to the session database.

When **multiple complementos** return contexto, their outputs are joined with double newlines in complemento discovery order (alphabetical by directory name).

**Use cases:** Memoria recall, RAG contexto injection, guardrails, per-turn analytics.

**Example — memoria recall:**

```python
import httpx

MEMORY_API = "https://your-memoria-api.example.com"

def recall(session_id, user_message, is_first_turn, **kwargs):
    try:
        resp = httpx.post(f"{MEMORY_API}/recall", json={
            "session_id": session_id,
            "query": user_message,
        }, timeout=3)
        memories = resp.json().get("results", [])
        if not memories:
            return None
        text = "Recalled contexto:\n" + "\n".join(f"- {m['text']}" for m in memories)
        return {"contexto": text}
    except Exception:
        return None

def register(ctx):
    ctx.register_hook("pre_llm_call", recall)
```

**Example — guardrails:**

```python
POLICY = "Never execute commands that delete files without explicit user confirmation."

def guardrails(**kwargs):
    return {"contexto": POLICY}

def register(ctx):
    ctx.register_hook("pre_llm_call", guardrails)
```

---

### `post_llm_call`

Fires **once per turn**, after the herramienta-calling loop completes and the agente has produced a final response. Only fires on **successful** turns — does not fire if the turn was interrupted.

**Callback signature:**

```python
def my_callback(session_id: str, user_message: str, assistant_response: str,
                conversation_history: list, modelo: str, platform: str, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `session_id` | `str` | Unique identifier for the current session |
| `user_message` | `str` | The user's original message for this turn |
| `assistant_response` | `str` | The agente's final text response for this turn |
| `conversation_history` | `list` | Copy of the full message list after the turn completed |
| `modelo` | `str` | The modelo identifier |
| `platform` | `str` | Where the session is running |

**Fires:** In `run_agente.py`, inside `run_conversation()`, after the herramienta loop exits with a final response. Guarded by `if final_response and not interrupted` — so it does **not** fire when the user interrupts mid-turn or the agente hits the iteration limit without producing a response.

**Return value:** Ignored.

**Use cases:** Syncing conversation data to an external memoria system, computing response quality metrics, logging turn summaries, triggering follow-up actions.

**Example — sync to external memoria:**

```python
import httpx

MEMORY_API = "https://your-memoria-api.example.com"

def sync_memoria(session_id, user_message, assistant_response, **kwargs):
    try:
        httpx.post(f"{MEMORY_API}/store", json={
            "session_id": session_id,
            "user": user_message,
            "assistant": assistant_response,
        }, timeout=5)
    except Exception:
        pass  # best-effort

def register(ctx):
    ctx.register_hook("post_llm_call", sync_memoria)
```

**Example — track response lengths:**

```python
import logging
logger = logging.getLogger(__name__)

def log_response_length(session_id, assistant_response, modelo, **kwargs):
    logger.info("RESPONSE session=%s modelo=%s chars=%d",
                session_id, modelo, len(assistant_response or ""))

def register(ctx):
    ctx.register_hook("post_llm_call", log_response_length)
```

---

### `on_session_start`

Fires **once** when a brand-new session is created. Does **not** fire on session continuation (when the user sends a second message in an existing session).

**Callback signature:**

```python
def my_callback(session_id: str, modelo: str, platform: str, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `session_id` | `str` | Unique identifier for the new session |
| `modelo` | `str` | The modelo identifier |
| `platform` | `str` | Where the session is running |

**Fires:** In `run_agente.py`, inside `run_conversation()`, during the first turn of a new session — specifically after the system prompt is built but before the herramienta loop starts. The check is `if not conversation_history` (no prior messages = new session).

**Return value:** Ignored.

**Use cases:** Initializing session-scoped state, warming caches, registering the session with an external service, logging session starts.

**Example — initialize a session cache:**

```python
_session_caches = {}

def init_session(session_id, modelo, platform, **kwargs):
    _session_caches[session_id] = {
        "modelo": modelo,
        "platform": platform,
        "herramienta_calls": 0,
        "started": __import__("datetime").datetime.now().isoformat(),
    }

def register(ctx):
    ctx.register_hook("on_session_start", init_session)
```

---

### `on_session_end`

Fires at the **very end** of every `run_conversation()` call, regardless of outcome. Also fires from the CLI's exit handler if the agente was mid-turn when the user quit.

**Callback signature:**

```python
def my_callback(session_id: str, completed: bool, interrupted: bool,
                modelo: str, platform: str, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `session_id` | `str` | Unique identifier for the session |
| `completed` | `bool` | `True` if the agente produced a final response, `False` otherwise |
| `interrupted` | `bool` | `True` if the turn was interrupted (user sent new message, `/stop`, or quit) |
| `modelo` | `str` | The modelo identifier |
| `platform` | `str` | Where the session is running |

**Fires:** In two places:
1. **`run_agente.py`** — at the end of every `run_conversation()` call, after all cleanup. Always fires, even if the turn errored.
2. **`cli.py`** — in the CLI's atexit handler, but **only** if the agente was mid-turn (`_agente_running=True`) when the exit occurred. This catches Ctrl+C and `/exit` during processing. In this case, `completed=False` and `interrupted=True`.

**Return value:** Ignored.

**Use cases:** Flushing buffers, closing connections, persisting session state, logging session duration, cleanup of resources initialized in `on_session_start`.

**Example — flush and cleanup:**

```python
_session_caches = {}

def cleanup_session(session_id, completed, interrupted, **kwargs):
    cache = _session_caches.pop(session_id, None)
    if cache:
        # Flush accumulated data to disk or external service
        status = "completed" if completed else ("interrupted" if interrupted else "failed")
        print(f"Session {session_id} ended: {status}, {cache['herramienta_calls']} herramienta calls")

def register(ctx):
    ctx.register_hook("on_session_end", cleanup_session)
```

**Example — session duration tracking:**

```python
import time, logging
logger = logging.getLogger(__name__)

_start_times = {}

def on_start(session_id, **kwargs):
    _start_times[session_id] = time.time()

def on_end(session_id, completed, interrupted, **kwargs):
    start = _start_times.pop(session_id, None)
    if start:
        duration = time.time() - start
        logger.info("SESSION_DURATION session=%s seconds=%.1f completed=%s interrupted=%s",
                     session_id, duration, completed, interrupted)

def register(ctx):
    ctx.register_hook("on_session_start", on_start)
    ctx.register_hook("on_session_end", on_end)
```

---

### `on_session_finalize`

Fires when the CLI or puerta de enlace **tears down** an active session — for example, when the user runs `/new`, the puerta de enlace GC'd an idle session, or the CLI quit with an active agente. This is the last chance to flush state tied to the outgoing session before its identity is gone.

**Callback signature:**

```python
def my_callback(session_id: str | None, platform: str, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `session_id` | `str` or `None` | The outgoing session ID. May be `None` if no active session existed. |
| `platform` | `str` | `"cli"` or the messaging platform name (`"telegram"`, `"discord"`, etc.). |

**Fires:** In `cli.py` (on `/new` / CLI exit) and `puerta de enlace/run.py` (when a session is reset or GC'd). Always paired with `on_session_reset` on the puerta de enlace side.

**Return value:** Ignored.

**Use cases:** Persist final session metrics before the session ID is discarded, close per-session resources, emit a final telemetry event, drain queued writes.

---

### `on_session_reset`

Fires when the puerta de enlace **swaps in a new session key** for an active chat — the user invoked `/new`, `/reset`, `/clear`, or the adapter picked a fresh session after an idle window. This lets complementos react to the fact that conversation state has been wiped without waiting for the next `on_session_start`.

**Callback signature:**

```python
def my_callback(session_id: str, platform: str, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `session_id` | `str` | The new session's ID (already rotated to the fresh value). |
| `platform` | `str` | The messaging platform name. |

**Fires:** In `puerta de enlace/run.py`, immediately after the new session key is allocated but before the next inbound message is processed. On the puerta de enlace, the order is: `on_session_finalize(old_id)` → swap → `on_session_reset(new_id)` → `on_session_start(new_id)` on the first inbound turn.

**Return value:** Ignored.

**Use cases:** Reset per-session caches keyed by `session_id`, emit "session rotated" analytics, prime a fresh state bucket.

---

See the **[Build a Complemento guide](/guides/build-a-hermes-complemento)** for the full walkthrough including herramienta schemas, handlers, and advanced hook patterns.

---

### `subagente_start`

Fires **once per child agente** after `delegate_task` has constructed the child `AIAgente` and before that child is run. Whether you delegate a single task or a batch of three, this hook fires once for each child.

This hook is specific to delegation/subagente lifecycle. It is not a universal "before any agente invocation" gate for puerta de enlace, CLI, cron, batch, MoA, or other runner-originated agente executions.

**Callback signature:**

```python
def my_callback(parent_session_id: str | None,
                parent_turn_id: str,
                parent_subagente_id: str | None,
                child_session_id: str | None,
                child_subagente_id: str,
                child_role: str,
                child_goal: str,
                **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `parent_session_id` | `str \| None` | Session ID of the delegating parent agente. |
| `parent_turn_id` | `str` | Turn ID of the parent agente turn that requested delegation, if available. |
| `parent_subagente_id` | `str \| None` | Parent subagente ID when this child was spawned by another subagente; `None` for top-level parent agentes. |
| `child_session_id` | `str \| None` | Session ID allocated for the child agente. |
| `child_subagente_id` | `str` | Stable subagente ID used by delegation observability and controls. |
| `child_role` | `str` | Effective child role after delegation policy is applied, for example `"leaf"` or `"orchestrator"`. |
| `child_goal` | `str` | Delegated goal/prompt that the child agente will execute. |

**Fires:** In `herramientas/delegate_herramienta.py`, inside `_build_child_agente()`, after the child `AIAgente` has been constructed and annotated with subagente identity metadata, and before `_run_single_child()` runs the child.

**Return value:** Ignored. This is an observer hook only; returning a value does not block or mutate the child agente run.

**Use cases:** Logging subagente creation, mapping parent/child session relationships, tracking nested delegation trees, emitting pre-run audit records, pre-allocating per-child observability resources.

**Example — log subagente creation:**

```python
import logging

logger = logging.getLogger(__name__)

def log_subagente_start(
    parent_session_id,
    parent_turn_id,
    child_session_id,
    child_subagente_id,
    child_role,
    child_goal,
    **kwargs,
):
    logger.info(
        "SUBAGENT_START parent=%s turn=%s child_session=%s child=%s role=%s goal=%r",
        parent_session_id,
        parent_turn_id,
        child_session_id,
        child_subagente_id,
        child_role,
        child_goal[:200],
    )

def register(ctx):
    ctx.register_hook("subagente_start", log_subagente_start)
```

:::info
`subagente_start` is useful for delegation observability, but it is not a blocking policy hook. To block delegation before a child is built, use [`pre_herramienta_call`](#pre_herramienta_call) to block the `delegate_task` herramienta call.
:::

---

### `subagente_stop`

Fires **once per child agente** after `delegate_task` finishes. Whether you delegated a single task or a batch of three, this hook fires once for each child, serialised on the parent thread.

**Callback signature:**

```python
def my_callback(parent_session_id: str, child_role: str | None,
                child_summary: str | None, child_status: str,
                duration_ms: int, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `parent_session_id` | `str` | Session ID of the delegating parent agente |
| `child_role` | `str \| None` | Orchestrator role tag set on the child (`None` if the feature isn't enabled) |
| `child_summary` | `str \| None` | The final response the child returned to the parent |
| `child_status` | `str` | `"completed"`, `"failed"`, `"interrupted"`, or `"error"` |
| `duration_ms` | `int` | Wall-clock time spent running the child, in milliseconds |

**Fires:** In `herramientas/delegate_herramienta.py`, after `ThreadPoolExecutor.as_completed()` drains all child futures. Firing is marshalled to the parent thread so hook authors don't have to reason about concurrent callback execution.

**Return value:** Ignored.

**Use cases:** Logging orchestration activity, accumulating child durations for billing, writing post-delegation audit records.

**Example — log orchestrator activity:**

```python
import logging
logger = logging.getLogger(__name__)

def log_subagente(parent_session_id, child_role, child_status, duration_ms, **kwargs):
    logger.info(
        "SUBAGENT parent=%s role=%s status=%s duration_ms=%d",
        parent_session_id, child_role, child_status, duration_ms,
    )

def register(ctx):
    ctx.register_hook("subagente_stop", log_subagente)
```

:::info
With heavy delegation (e.g. orchestrator roles × 5 leaves × nested depth), `subagente_stop` fires many times per turn. Keep your callback fast; push expensive work to a background queue.
:::

---

### `pre_puerta de enlace_dispatch`

Fires **once per incoming `MessageEvent`** in the puerta de enlace, after the internal-event guard but **before** auth/pairing and agente dispatch. This is the interception point for puerta de enlace-level message-flow policies (listen-only windows, human handover, per-chat routing, etc.) that don't fit cleanly into any single platform adapter.

**Callback signature:**

```python
def my_callback(event, puerta de enlace, session_store, **kwargs):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `event` | `MessageEvent` | The normalized inbound message (has `.text`, `.source`, `.message_id`, `.internal`, etc.). |
| `puerta de enlace` | `Puerta de enlaceRunner` | The active puerta de enlace runner, so complementos can call `puerta de enlace.adapters[platform].send(...)` for side-channel replies (owner notifications, etc.). |
| `session_store` | `SessionStore` | For silent transcript ingestion via `session_store.append_to_transcript(...)`. |

**Fires:** In `puerta de enlace/run.py`, inside `Puerta de enlaceRunner._handle_message()`, immediately after `is_internal` is computed. **Internal events skip the hook entirely** (they are system-generated — background-process completions, etc. — and must not be gate-kept by user-facing policy).

**Return value:** `None` or a dict. The first recognized action dict wins; remaining complemento results are ignored. Exceptions in complemento callbacks are caught and logged; the puerta de enlace always falls through to normal dispatch on error.

| Return | Effect |
|--------|--------|
| `{"action": "skip", "reason": "..."}` | Drop the message — no agente reply, no pairing flow, no auth. Complemento is assumed to have handled it (e.g. silent-ingested into the transcript). |
| `{"action": "rewrite", "text": "new text"}` | Replace `event.text`, then continue normal dispatch with the modified event. Useful for collapsing buffered ambient messages into a single prompt. |
| `{"action": "allow"}` / `None` | Normal dispatch — runs the full auth / pairing / agente-loop chain. |

**Use cases:** Listen-only group chats (only respond when tagged; buffer ambient messages into contexto); human handover (silent-ingest customer messages while owner handles the chat manually); per-perfil rate limiting; policy-driven routing.

**Example — drop unauthorized DMs silently without triggering the pairing code:**

```python
def deny_unauthorized_dms(event, **kwargs):
    src = event.source
    if src.chat_type == "dm" and not _is_approved_user(src.user_id):
        return {"action": "skip", "reason": "unauthorized-dm"}
    return None

def register(ctx):
    ctx.register_hook("pre_puerta de enlace_dispatch", deny_unauthorized_dms)
```

**Example — rewrite an ambient-message buffer into a single prompt on mention:**

```python
_buffers = {}

def buffer_or_rewrite(event, **kwargs):
    key = (event.source.platform, event.source.chat_id)
    buf = _buffers.setdefault(key, [])
    if _bot_mentioned(event.text):
        combined = "\n".join(buf + [event.text])
        buf.clear()
        return {"action": "rewrite", "text": combined}
    buf.append(event.text)
    return {"action": "skip", "reason": "ambient-buffered"}

def register(ctx):
    ctx.register_hook("pre_puerta de enlace_dispatch", buffer_or_rewrite)
```

---

### `pre_approval_request`

Fires **immediately before** an approval request is shown to the user — covers every surface: interactive CLI, the Ink TUI, puerta de enlace platforms (Telegram, Discord, Slack, WhatsApp, Matrix, etc.), and ACP clients (VS Code, Zed, JetBrains).

This is the right place to wire a custom notifier — for example, a macOS menu-bar app that pops an allow/deny notification, or an audit log that records every approval request with contexto.

**Callback signature:**

```python
def my_callback(
    command: str,
    description: str,
    pattern_key: str,
    pattern_keys: list[str],
    session_key: str,
    surface: str,
    **kwargs,
):
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `command` | `str` | The shell command awaiting approval |
| `description` | `str` | Human-readable reason(s) the command is flagged (combined when multiple patterns match) |
| `pattern_key` | `str` | Primary pattern key that triggered the approval (e.g. `"rm_rf"`, `"sudo"`) |
| `pattern_keys` | `list[str]` | All pattern keys that matched |
| `session_key` | `str` | Session identifier, useful for scoping notifications per-chat |
| `surface` | `str` | `"cli"` for interactive CLI/TUI prompts, `"puerta de enlace"` for async platform approvals |

**Return value:** ignored. Hooks here are observer-only; they cannot veto or pre-answer the approval. Use [`pre_herramienta_call`](#pre_herramienta_call) to block a herramienta before it reaches the approval system.

**Use cases:** Desktop notifications, push alerts, audit logging, Slack webhooks, escalation routing, metrics.

**Example — desktop notification on macOS:**

```python
import subprocess

def notify_approval(command, description, session_key, **kwargs):
    title = "Hermes needs approval"
    body = f"{description}: {command[:80]}"
    subprocess.Popen([
        "osascript", "-e",
        f'display notification "{body}" with title "{title}"',
    ])

def register(ctx):
    ctx.register_hook("pre_approval_request", notify_approval)
```

---

### `post_approval_response`

Fires **after** the user responds to an approval prompt (or the prompt times out).

**Callback signature:**

```python
def my_callback(
    command: str,
    description: str,
    pattern_key: str,
    pattern_keys: list[str],
    session_key: str,
    surface: str,
    choice: str,
    **kwargs,
):
```

Same kwargs as `pre_approval_request`, plus:

| Parameter | Type | Description |
|-----------|------|-------------|
| `choice` | `str` | One of `"once"`, `"session"`, `"always"`, `"deny"`, or `"timeout"` |

**Return value:** ignored.

**Use cases:** Close the matching desktop notification, record the final decision in an audit log, update metrics, roll forward a rate limiter.

```python
def log_decision(command, choice, session_key, **kwargs):
    logger.info("approval %s: %s for session %s", choice, command[:60], session_key)

def register(ctx):
    ctx.register_hook("post_approval_response", log_decision)
```

---

### `transform_herramienta_result`

Fires **after** a herramienta returns and **before** the result is appended to the conversation. Lets a complemento rewrite ANY herramienta's result string — not just terminal output — before the modelo sees it.

**Callback signature:**

```python
def my_callback(
    herramienta_name: str,
    arguments: dict,
    result: str,
    task_id: str | None,
    **kwargs,
) -> str | None:
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `herramienta_name` | `str` | Herramienta that produced the result (`read_file`, `web_extract`, `delegate_task`, …). |
| `arguments` | `dict` | Arguments the modelo called the herramienta with. |
| `result` | `str` | The herramienta's raw result string, post-truncation and post-ANSI-strip. |
| `task_id` | `str \| None` | Task/session ID when running inside RL/benchmark environments. |

**Return value:** `str` to replace the result (the returned string is what the modelo sees), `None` to leave it unchanged.

**Use cases:** Redact organization-specific PII from `web_extract` output, wrap long JSON herramienta responses in a summary header, inject retrieval-augmented hints into `read_file` results, rewrite `delegate_task` subagente reports into a project-specific schema.

```python
import re
SECRET = re.compile(r"sk-[A-Za-z0-9]{32,}")

def redact_secrets(herramienta_name, result, **kwargs):
    if SECRET.search(result):
        return SECRET.sub("[REDACTED]", result)
    return None

def register(ctx):
    ctx.register_hook("transform_herramienta_result", redact_secrets)
```

Applies to every herramienta. For terminal-only rewriting see `transform_terminal_output` below — it's narrower and runs earlier in the pipeline (pre-truncation, pre-redaction).

---

### `transform_terminal_output`

Fires inside the `terminal` herramienta's foreground-output pipeline, **before** the default 50 KB truncation, ANSI strip, and secret redaction. Lets complementos rewrite the raw stdout/stderr of a shell command before any downstream processing touches it.

**Callback signature:**

```python
def my_callback(
    command: str,
    output: str,
    exit_code: int,
    cwd: str,
    task_id: str | None,
    **kwargs,
) -> str | None:
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `command` | `str` | The shell command that produced the output. |
| `output` | `str` | Raw combined stdout/stderr (may be very large — truncation happens after the hook). |
| `exit_code` | `int` | Process exit code. |
| `cwd` | `str` | Working directory the command ran in. |

**Return value:** `str` to replace the output, `None` to leave it unchanged.

**Use cases:** Inject summaries for commands that produce massive output (`du -ah`, `find`, `tree`), tag output with a project-specific marker so downstream hooks know how to handle it, strip timing noise that flaps between runs and defeats prompt caching.

```python
def summarize_find(command, output, **kwargs):
    if command.startswith("find ") and len(output) > 50_000:
        lines = output.count("\n")
        head = "\n".join(output.splitlines()[:40])
        return f"{head}\n\n[summary: {lines} paths total, showing first 40]"
    return None

def register(ctx):
    ctx.register_hook("transform_terminal_output", summarize_find)
```

Pairs well with `transform_herramienta_result` (which covers every other herramienta).

---

### `transform_llm_output`

Fires **once per turn** after the herramienta-calling loop completes and the modelo has produced a final response, **before** that response is delivered to the user (CLI, puerta de enlace, or programmatic caller). Lets a complemento rewrite the assistant's final text using classical-programming methods — no extra inference tokens burned on SOUL flavor text or a habilidad-driven transform.

**Callback signature:**

```python
def my_callback(
    response_text: str,
    session_id: str,
    modelo: str,
    platform: str,
    **kwargs,
) -> str | None:
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `response_text` | `str` | The assistant's final response text for this turn. |
| `session_id` | `str` | Session ID for this conversation (may be empty for one-shot runs). |
| `modelo` | `str` | Modelo name that produced the response (e.g. `anthropic/claude-sonnet-4.6`). |
| `platform` | `str` | Delivery platform (`cli`, `telegram`, `discord`, …; empty when unset). |

**Return value:** Non-empty `str` to replace the response text, `None` or empty string to leave it unchanged. **First non-empty string wins** when multiple complementos register — mirroring `transform_herramienta_result`.

**Use cases:** Apply a personality/vocabulary transform (pirate-speak, Spongebob), redact user-specific identifiers from the final text, append a project-specific signature footer, enforce a house style guide without burning tokens on SOUL instructions.

```python
import os, re

def spongebob(response_text, **kwargs):
    if os.environ.get("SPONGEBOB_MODE") != "on":
        return None  # pass through unchanged
    return re.sub(r"!", "!! Tartar sauce!", response_text)

def register(ctx):
    ctx.register_hook("transform_llm_output", spongebob)
```

The hook is guarded on a non-empty, non-interrupted response — it will not fire on stop-button interrupts or empty turns. Exceptions are logged as warnings and do not break agente execution.

---

## Shell Hooks

Declare shell-script hooks in your `cli-config.yaml` and Hermes will run them as subprocesses whenever the corresponding complemento-hook event fires — in both CLI and puerta de enlace sessions. No Python complemento authoring required.

Use shell hooks when you want a drop-in, single-file script (Bash, Python, anything with a shebang) to:

- **Block a herramienta call** — reject dangerous `terminal` commands, enforce per-directory policies, require approval for destructive `write_file` / `patch` operations.
- **Run after a herramienta call** — auto-format Python or TypeScript files that the agente just wrote, log API calls, trigger a CI workflow.
- **Inject contexto into the next LLM turn** — prepend `git status` output, the current weekday, or retrieved documents to the user message (see [`pre_llm_call`](#pre_llm_call)).
- **Observe lifecycle events** — write a log line when a subagente completes (`subagente_stop`) or a session starts (`on_session_start`).

Shell hooks are registered by calling `agente.shell_hooks.register_from_config(cfg)` at both CLI startup (`hermes_cli/main.py`) and puerta de enlace startup (`puerta de enlace/run.py`). They compose naturally with Python complemento hooks — both flow through the same dispatcher.

### Comparison at a glance

| Dimension | Shell hooks | [Complemento hooks](#complemento-hooks) | [Puerta de enlace hooks](#puerta de enlace-event-hooks) |
|-----------|-------------|-------------------------------|---------------------------------------|
| Declared in | `hooks:` block in `~/.hermes/config.yaml` | `register()` in a `complemento.yaml` complemento | `HOOK.yaml` + `handler.py` directory |
| Lives under | `~/.hermes/agente-hooks/` (by convention) | `~/.hermes/complementos/<name>/` | `~/.hermes/hooks/<name>/` |
| Language | Any (Bash, Python, Go binary, …) | Python only | Python only |
| Runs in | CLI + Puerta de enlace | CLI + Puerta de enlace | Puerta de enlace only |
| Events | `VALID_HOOKS` (incl. `subagente_stop`) | `VALID_HOOKS` | Puerta de enlace lifecycle (`puerta de enlace:startup`, `agente:*`, `command:*`) |
| Can block a herramienta call | Yes (`pre_herramienta_call`) | Yes (`pre_herramienta_call`) | No |
| Can inject LLM contexto | Yes (`pre_llm_call`) | Yes (`pre_llm_call`) | No |
| Consent | First-use prompt per `(event, command)` pair | Implicit (Python complemento trust) | Implicit (dir trust) |
| Inter-process isolation | Yes (subprocess) | No (in-process) | No (in-process) |

### Configuración schema

```yaml
hooks:
  <event_name>:                  # Must be in VALID_HOOKS
    - matcher: "<regex>"         # Optional; used for pre/post_herramienta_call only
      command: "<shell command>" # Required; runs via shlex.split, shell=False
      timeout: <seconds>         # Optional; default 60, capped at 300

hooks_auto_accept: false         # See "Consent modelo" below
```

Event names must be one of the [complemento hook events](#complemento-hooks); typos produce a "Did you mean X?" warning and are skipped. Unknown keys inside a single entry are ignored; missing `command` is a skip-with-warning. `timeout > 300` is clamped with a warning.

### JSON wire protocol

Each time the event fires, Hermes spawns a subprocess for every matching hook (matcher permitting), pipes a JSON payload to **stdin**, and reads **stdout** back as JSON.

**stdin — payload the script receives:**

```json
{
  "hook_event_name": "pre_herramienta_call",
  "herramienta_name":       "terminal",
  "herramienta_input":      {"command": "rm -rf /"},
  "session_id":      "sess_abc123",
  "cwd":             "/home/user/project",
  "extra":           {"task_id": "...", "herramienta_call_id": "..."}
}
```

`herramienta_name` and `herramienta_input` are `null` for non-herramienta events (`pre_llm_call`, `subagente_stop`, session lifecycle). The `extra` dict carries all event-specific kwargs (`user_message`, `conversation_history`, `child_role`, `duration_ms`, …). Unserialisable values are stringified rather than omitted.

**stdout — optional response:**

```jsonc
// Block a pre_herramienta_call (both shapes accepted; normalised internally):
{"decision": "block", "reason":  "Forbidden: rm -rf"}   // Claude-Code style
{"action":   "block", "message": "Forbidden: rm -rf"}   // Hermes-canonical

// Inject contexto for pre_llm_call:
{"contexto": "Today is Friday, 2026-04-17"}

// Silent no-op — any empty / non-matching output is fine:
```

Malformed JSON, non-zero exit codes, and timeouts log a warning but never abort the agente loop.

### Worked examples

#### 1. Auto-format Python files after every write

```yaml
# ~/.hermes/config.yaml
hooks:
  post_herramienta_call:
    - matcher: "write_file|patch"
      command: "~/.hermes/agente-hooks/auto-format.sh"
```

```bash
#!/usr/bin/env bash
# ~/.hermes/agente-hooks/auto-format.sh
payload="$(cat -)"
path=$(echo "$payload" | jq -r '.herramienta_input.path // empty')
[[ "$path" == *.py ]] && command -v black >/dev/null && black "$path" 2>/dev/null
printf '{}\n'
```

The agente's in-contexto view of the file is **not** re-read automatically — the reformat only affects the file on disk. Subsequent `read_file` calls pick up the formatted version.

#### 2. Block destructive `terminal` commands

```yaml
hooks:
  pre_herramienta_call:
    - matcher: "terminal"
      command: "~/.hermes/agente-hooks/block-rm-rf.sh"
      timeout: 5
```

```bash
#!/usr/bin/env bash
# ~/.hermes/agente-hooks/block-rm-rf.sh
payload="$(cat -)"
cmd=$(echo "$payload" | jq -r '.herramienta_input.command // empty')
if echo "$cmd" | grep -qE 'rm[[:space:]]+-rf?[[:space:]]+/'; then
  printf '{"decision": "block", "reason": "blocked: rm -rf / is not permitted"}\n'
else
  printf '{}\n'
fi
```

#### 3. Inject `git status` into every turn (Claude-Code `UserPromptSubmit` equivalent)

```yaml
hooks:
  pre_llm_call:
    - command: "~/.hermes/agente-hooks/inject-cwd-contexto.sh"
```

```bash
#!/usr/bin/env bash
# ~/.hermes/agente-hooks/inject-cwd-contexto.sh
cat - >/dev/null   # discard stdin payload
if status=$(git status --porcelain 2>/dev/null) && [[ -n "$status" ]]; then
  jq --null-input --arg s "$status" \
     '{contexto: ("Uncommitted changes in cwd:\n" + $s)}'
else
  printf '{}\n'
fi
```

Claude Code's `UserPromptSubmit` event is intentionally not a separate Hermes event — `pre_llm_call` fires at the same place and already supports contexto injection. Use it here.

#### 4. Log every subagente completion

```yaml
hooks:
  subagente_stop:
    - command: "~/.hermes/agente-hooks/log-orchestration.sh"
```

```bash
#!/usr/bin/env bash
# ~/.hermes/agente-hooks/log-orchestration.sh
log=~/.hermes/logs/orchestration.log
jq -c '{ts: now, parent: .session_id, extra: .extra}' < /dev/stdin >> "$log"
printf '{}\n'
```

### Consent modelo

Each unique `(event, command)` pair prompts the user for approval the first time Hermes sees it, then persists the decision to `~/.hermes/shell-hooks-allowlist.json`. Subsequent runs (CLI or puerta de enlace) skip the prompt.

Three escape hatches bypass the interactive prompt — any one is sufficient:

1. `--accept-hooks` flag on the CLI (e.g. `hermes --accept-hooks chat`)
2. `HERMES_ACCEPT_HOOKS=1` environment variable
3. `hooks_auto_accept: true` in `cli-config.yaml`

Non-TTY runs (puerta de enlace, cron, CI) need one of these three — otherwise any newly-added hook silently stays un-registered and logs a warning.

**Script edits are silently trusted.** The allowlist keys on the exact command string, not the script's hash, so editing the script on disk does not invalidate consent. `hermes hooks doctor` flags mtime drift so you can spot edits and decide whether to re-approve.

#### Manual allowlisting

Manual allowlisting is useful for non-TTY or service-account deployments where an operator cannot answer the first-use prompt interactively. The allowlist file is `~/.hermes/shell-hooks-allowlist.json`, and the expected format is an `approvals` array. Each approval records the hook `event` and the exact `command` string:

```json
{
  "approvals": [
    {
      "event": "post_llm_call",
      "command": "/home/hermes/.hermes/hooks/my-hook.py"
    }
  ]
}
```

The command string must match the configured hook command exactly. A path-keyed object with a `sha256` field is not the expected format and will not approve the hook. Verify manual entries with `hermes hooks list`.

### The `hermes hooks` CLI

| Command | What it does |
|---------|--------------|
| `hermes hooks list` | Dump configured hooks with matcher, timeout, and consent status |
| `hermes hooks test <event> [--for-herramienta X] [--payload-file F]` | Fire every matching hook against a synthetic payload and print the parsed response |
| `hermes hooks revoke <command>` | Remove every allowlist entry matching `<command>` (takes effect on next restart) |
| `hermes hooks doctor` | For every configured hook: check exec bit, allowlist status, mtime drift, JSON output validity, and rough execution time |

### Security

Shell hooks run with **your full user credentials** — same trust boundary as a cron entry or a shell alias. Treat the `hooks:` block in `config.yaml` as privileged configuración:

- Only reference scripts you wrote or fully reviewed.
- Keep scripts inside `~/.hermes/agente-hooks/` so the path is easy to audit.
- Re-run `hermes hooks doctor` after you pull a shared config to spot newly-added hooks before they register.
- If your config.yaml is version-controlled across a team, review PRs that change the `hooks:` section the same way you'd review CI config.

### Ordering and precedence

Both Python complemento hooks and shell hooks flow through the same `invoke_hook()` dispatcher. Python complementos are registered first (`discover_and_load()`), shell hooks second (`register_from_config()`), so Python `pre_herramienta_call` block decisions take precedence in tie cases. The first valid block wins — the aggregator returns as soon as any callback produces `{"action": "block", "message": str}` with a non-empty message.

---