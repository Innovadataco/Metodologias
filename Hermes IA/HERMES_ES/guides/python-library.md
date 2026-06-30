<!-- source: website/docs/guides/python-library.md -->
# Using Hermes as a Python Library

# Using Hermes as a Python Library

Hermes isn't just a CLI herramienta. You can import `AIAgente` directly and use it programmatically in your own Python scripts, web applications, or automation pipelines. This guide shows you how.

---

## Instalación

Install Hermes directly from the repository:

```bash
pip install git+https://github.com/NousResearch/hermes-agente.git
```

Or with [uv](https://docs.astral.sh/uv/):

```bash
uv pip install git+https://github.com/NousResearch/hermes-agente.git
```

You can also pin it in your `requirements.txt`:

```text
hermes-agente @ git+https://github.com/NousResearch/hermes-agente.git
```

:::tip
The same environment variables used by the CLI are required when using Hermes as a library. At minimum, set `OPENROUTER_API_KEY` (or `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` if using direct proveedor access).
:::

---

## Basic Usage

The simplest way to use Hermes is the `chat()` method — pass a message, get a string back:

```python
from run_agente import AIAgente

agente = AIAgente(
    modelo="anthropic/claude-sonnet-4.6",
    quiet_mode=True,
)
response = agente.chat("What is the capital of France?")
print(response)
```

`chat()` handles the full conversation loop internally — herramienta calls, retries, everything — and returns just the final text response.

:::warning
Always set `quiet_mode=True` when embedding Hermes in your own code. Without it, the agente prints CLI spinners, progress indicators, and other terminal output that will clutter your application's output.
:::

---

## Full Conversation Control

For more control over the conversation, use `run_conversation()` directly. It returns a dictionary with the full response, message history, and metadata:

```python
agente = AIAgente(
    modelo="anthropic/claude-sonnet-4.6",
    quiet_mode=True,
)

result = agente.run_conversation(
    user_message="Search for recent Python 3.13 features",
    task_id="my-task-1",
)

print(result["final_response"])
print(f"Messages exchanged: {len(result['messages'])}")
```

The returned dictionary contains:
- **`final_response`** — The agente's final text reply
- **`messages`** — The complete message history (system, user, assistant, herramienta calls)

(The `task_id` you pass in is stored on the agente instance for VM isolation but isn't echoed back in the return dict.)

You can also pass a custom system message that overrides the ephemeral system prompt for that call:

```python
result = agente.run_conversation(
    user_message="Explain quicksort",
    system_message="You are a computer science tutor. Use simple analogies.",
)
```

---

## Configuring Herramientas

Control which herramientasets the agente has access to using `enabled_herramientasets` or `disabled_herramientasets`:

```python
# Only enable web herramientas (browsing, search)
agente = AIAgente(
    modelo="anthropic/claude-sonnet-4.6",
    enabled_herramientasets=["web"],
    quiet_mode=True,
)

# Enable everything except terminal access
agente = AIAgente(
    modelo="anthropic/claude-sonnet-4.6",
    disabled_herramientasets=["terminal"],
    quiet_mode=True,
)
```

:::tip
Use `enabled_herramientasets` when you want a minimal, locked-down agente (e.g., only web search for a research bot). Use `disabled_herramientasets` when you want most capabilities but need to restrict specific ones (e.g., no terminal access in a shared environment).
:::

---

## Multi-turn Conversations

Maintain conversation state across multiple turns by passing the message history back in:

```python
agente = AIAgente(
    modelo="anthropic/claude-sonnet-4.6",
    quiet_mode=True,
)

# First turn
result1 = agente.run_conversation("My name is Alice")
history = result1["messages"]

# Second turn — agente remembers the contexto
result2 = agente.run_conversation(
    "What's my name?",
    conversation_history=history,
)
print(result2["final_response"])  # "Your name is Alice."
```

The `conversation_history` parameter accepts the `messages` list from a previous result. The agente copies it internally, so your original list is never mutated.

---

## Saving Trajectories

Enable trajectory saving to capture conversations in ShareGPT format — useful for generating training data or debugging:

```python
agente = AIAgente(
    modelo="anthropic/claude-sonnet-4.6",
    save_trajectories=True,
    quiet_mode=True,
)

agente.chat("Write a Python function to sort a list")
# Saves to trajectory_samples.jsonl in ShareGPT format
```

Each conversation is appended as a single JSONL line, making it easy to collect datasets from automated runs.

---

## Custom System Prompts

Use `ephemeral_system_prompt` to set a custom system prompt that guides the agente's behavior but is **not** saved to trajectory files (keeping your training data clean):

```python
agente = AIAgente(
    modelo="anthropic/claude-sonnet-4",
    ephemeral_system_prompt="You are a SQL expert. Only answer database questions.",
    quiet_mode=True,
)

response = agente.chat("How do I write a JOIN query?")
print(response)
```

This is ideal for building specialized agentes — a code reviewer, a documentation writer, a SQL assistant — all using the same underlying herramientaing.

---

## Batch Processing

For running many prompts in parallel, Hermes includes `batch_runner.py`. It manages concurrent `AIAgente` instances with proper resource isolation:

```bash
python batch_runner.py --input prompts.jsonl --output results.jsonl
```

Each prompt gets its own `task_id` and isolated environment. If you need custom batch logic, you can build your own using `AIAgente` directly:

```python
import concurrent.futures
from run_agente import AIAgente

prompts = [
    "Explain recursion",
    "What is a hash table?",
    "How does garbage collection work?",
]

def process_prompt(prompt):
    # Create a fresh agente per task for thread safety
    agente = AIAgente(
        modelo="anthropic/claude-sonnet-4",
        quiet_mode=True,
        skip_memoria=True,
    )
    return agente.chat(prompt)

with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(process_prompt, prompts))

for prompt, result in zip(prompts, results):
    print(f"Q: {prompt}\nA: {result}\n")
```

:::warning
Always create a **new `AIAgente` instance per thread or task**. The agente maintains internal state (conversation history, herramienta sessions, iteration counters) that is not thread-safe to share.
:::

---

## Integration Examples

### FastAPI Endpoint

```python
from fastapi import FastAPI
from pydantic import BaseModelo
from run_agente import AIAgente

app = FastAPI()

class ChatRequest(BaseModelo):
    message: str
    modelo: str = "anthropic/claude-sonnet-4"

@app.post("/chat")
async def chat(request: ChatRequest):
    agente = AIAgente(
        modelo=request.modelo,
        quiet_mode=True,
        skip_contexto_files=True,
        skip_memoria=True,
    )
    response = agente.chat(request.message)
    return {"response": response}
```

### Discord Bot

```python
import discord
from run_agente import AIAgente

client = discord.Client(intents=discord.Intents.default())

@client.event
async def on_message(message):
    if message.author == client.user:
        return
    if message.content.startswith("!hermes "):
        query = message.content[8:]
        agente = AIAgente(
            modelo="anthropic/claude-sonnet-4",
            quiet_mode=True,
            skip_contexto_files=True,
            skip_memoria=True,
            platform="discord",
        )
        response = agente.chat(query)
        await message.channel.send(response[:2000])

client.run("YOUR_DISCORD_TOKEN")
```

### CI/CD Pipeline Step

```python
#!/usr/bin/env python3
"""CI step: auto-review a PR diff."""
import subprocess
from run_agente import AIAgente

diff = subprocess.check_output(["git", "diff", "main...HEAD"]).decode()

agente = AIAgente(
    modelo="anthropic/claude-sonnet-4",
    quiet_mode=True,
    skip_contexto_files=True,
    skip_memoria=True,
    disabled_herramientasets=["terminal", "browser"],
)

review = agente.chat(
    f"Review this PR diff for bugs, security issues, and style problems:\n\n{diff}"
)
print(review)
```

---

## Key Constructor Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `modelo` | `str` | `""` | Modelo in OpenRouter format (defaults to empty; resolved from your hermes config at runtime) |
| `quiet_mode` | `bool` | `False` | Suppress CLI output |
| `enabled_herramientasets` | `List[str]` | `None` | Whitelist specific herramientasets |
| `disabled_herramientasets` | `List[str]` | `None` | Blacklist specific herramientasets |
| `save_trajectories` | `bool` | `False` | Save conversations to JSONL |
| `ephemeral_system_prompt` | `str` | `None` | Custom system prompt (not saved to trajectories) |
| `max_iterations` | `int` | `90` | Max herramienta-calling iterations per conversation |
| `skip_contexto_files` | `bool` | `False` | Skip loading AGENTS.md files |
| `skip_memoria` | `bool` | `False` | Disable persistent memoria read/write |
| `api_key` | `str` | `None` | API key (falls back to env vars) |
| `base_url` | `str` | `None` | Custom API endpoint URL |
| `platform` | `str` | `None` | Platform hint (`"discord"`, `"telegram"`, etc.) |

---

## Important Notes

:::tip
- Set **`skip_contexto_files=True`** if you don't want `AGENTS.md` files from the working directory loaded into the system prompt.
- Set **`skip_memoria=True`** to prevent the agente from reading or writing persistent memoria — recommended for stateless API endpoints.
- The `platform` parameter (e.g., `"discord"`, `"telegram"`) injects platform-specific formatting hints so the agente adapts its output style.
:::

:::warning
- **Thread safety**: Create one `AIAgente` per thread or task. Never share an instance across concurrent calls.
- **Resource cleanup**: The agente automatically cleans up resources (terminal sessions, browser instances) when a conversation ends. If you're running in a long-lived process, ensure each conversation completes normally.
- **Iteration limits**: The default `max_iterations=90` is generous. For simple Q&A use cases, consider lowering it (e.g., `max_iterations=10`) to prevent runaway herramienta-calling loops and control costs.
:::

---