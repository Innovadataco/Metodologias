<!-- source: website/docs/guia-desarrollador/complemento-llm-access.md -->
# Complemento LLM Access

# Complemento LLM Access

`ctx.llm` is the supported way for a complemento to make an LLM call.
Chat completion, structured extraction, sync, async, with or without
images — same surface, same trust gate, same host-owned credentials.

Complementos reach for this when they need to do something that involves
the modelo but isn't part of the agente's conversation. A hook that
rewrites a herramienta error into something a non-engineer can read. A
puerta de enlace adapter that translates an inbound message before queuing
it. A slash command that summarises a long paste. A scheduled job
that scores yesterday's activity and writes one line to a status
board. A pre-filter that decides whether a message is worth waking
the agente up for at all.

These are jobs the agente shouldn't be in the loop on. They want one
LLM call, a typed answer, and to be done.

## The smallest possible call

```python
result = ctx.llm.complete(messages=[{"role": "user", "content": "ping"}])
return result.text
```

That's the whole API in one line. No keys, no proveedor config, no
SDK initialisation. The complemento runs against whatever proveedor and
modelo the user is currently using — when they switch proveedors, the
complemento follows them automatically.

## A more complete chat example

```python
result = ctx.llm.complete(
    messages=[
        {"role": "system", "content": "Rewrite errors as one short sentence a non-engineer can act on."},
        {"role": "user",   "content": traceback_text},
    ],
    max_tokens=64,
    purpose="hooks.error-rewrite",
)
return result.text
```

`purpose` is a free-form audit string — it shows up in `agente.log`
and in `result.audit` so operators can see which complemento made which
call. Optional but recommended for anything that fires often.

## Structured output

When the complemento needs a typed answer, switch to the structured lane:

```python
result = ctx.llm.complete_structured(
    instructions="Score this support reply for urgency (0–1) and pick a category.",
    input=[{"type": "text", "text": message_body}],
    json_schema=TRIAGE_SCHEMA,
    purpose="support.triage",
    temperature=0.0,
    max_tokens=128,
)

if result.parsed["urgency"] > 0.8:
    await dispatch_to_oncall(result.parsed["category"], message_body)
```

The host requests JSON output from the proveedor, parses it locally
as a fallback, validates against your schema if `jsonschema` is
installed, and hands back a Python object on `result.parsed`. If the
modelo couldn't produce valid JSON, `result.parsed` is `None` and
`result.text` carries the raw response.

## What this lane gives you

* **One call, four shapes.** `complete()` for chat,
  `complete_structured()` for typed JSON, `acomplete()` and
  `acomplete_structured()` for asyncio. Same arguments, same result
  objects.
* **Host-owned credentials.** OAuth tokens, refresh flows, the
  credential pool, per-task aux overrides — every credential
  concept Hermes already has applies. The complemento never sees a
  token; the host attributes the call back through `result.audit`.
* **Bounded.** Single sync or async call. No streaming, no herramienta
  loops, no conversation state to manage. State the input, get the
  result, return.
* **Fail-closed trust.** A complemento you've never configured cannot
  pick its own proveedor, modelo, agente, or stored credential. The
  default posture is "use what the user is using." Operators opt in
  to specific overrides, per complemento, in `config.yaml`.

## Quick start

Two complete complementos below — one chat, one structured. Both ship
inside a single `register(ctx)` function and need zero outside
configuración to run against whatever modelo the user has active.

### Chat completion — `/tldr`

```python
def register(ctx):
    ctx.register_command(
        name="tldr",
        handler=lambda raw: _tldr(ctx, raw),
        description="Summarise the supplied text in one paragraph.",
        args_hint="<text>",
    )


def _tldr(ctx, raw_args: str) -> str:
    text = raw_args.strip()
    if not text:
        return "Usage: /tldr <text to summarise>"
    result = ctx.llm.complete(
        messages=[
            {"role": "system",
             "content": "Summarise the user's text in one tight paragraph. No preamble."},
            {"role": "user", "content": text},
        ],
        max_tokens=256,
        temperature=0.3,
        purpose="tldr",
    )
    return result.text
```

`result.text` is the modelo's response; `result.usage` carries token
counts; `result.proveedor` and `result.modelo` carry attribution.

### Structured extraction — `/paste-to-tasks`

```python
def register(ctx):
    ctx.register_command(
        name="paste-to-tasks",
        handler=lambda raw: _paste_to_tasks(ctx, raw),
        description="Turn freeform meeting notes into structured tasks.",
        args_hint="<text>",
    )


_TASKS_SCHEMA = {
    "type": "object",
    "properties": {
        "tasks": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "owner":  {"type": "string"},
                    "action": {"type": "string"},
                    "due":    {"type": "string", "description": "ISO date or empty"},
                },
                "required": ["action"],
            },
        },
    },
    "required": ["tasks"],
}


def _paste_to_tasks(ctx, raw_args: str) -> str:
    if not raw_args.strip():
        return "Usage: /paste-to-tasks <meeting notes>"
    result = ctx.llm.complete_structured(
        instructions=(
            "Extract concrete action items from these meeting notes. "
            "One task per actionable line. If no owner is named, leave 'owner' blank."
        ),
        input=[{"type": "text", "text": raw_args}],
        json_schema=_TASKS_SCHEMA,
        schema_name="meeting.tasks",
        purpose="paste-to-tasks",
        temperature=0.0,
        max_tokens=512,
    )
    if result.parsed is None:
        return f"Couldn't parse a response. Raw output:\n{result.text}"
    lines = [f"- [{t.get('owner') or '?'}] {t['action']}" for t in result.parsed["tasks"]]
    return "\n".join(lines) or "(no tasks found)"
```

A third worked example, this time with image input, lives in the
[`hermes-example-complementos`](https://github.com/NousResearch/hermes-example-complementos/tree/main/complemento-llm-example)
repo (companion repo for reference complementos — not bundled with
hermes-agente itself). For the async surface (`acomplete()` /
`acomplete_structured()` with `asyncio.gather()`), see
[`complemento-llm-async-example`](https://github.com/NousResearch/hermes-example-complementos/tree/main/complemento-llm-async-example)
in the same repo.

## When to use which

| You want… | Reach for |
|---|---|
| A free-form text response (translation, summary, rewrite, generation) | `complete()` |
| A multi-turn prompt (system + few-shot examples + user) | `complete()` |
| A typed dict back, validated against a schema | `complete_structured()` |
| Image-or-text input with a typed dict back | `complete_structured()` |
| The same call from async code (puerta de enlace adapters, async hooks) | `acomplete()` / `acomplete_structured()` |

Everything else — proveedor selection, modelo resolution, auth, fallback,
timeout, vision routing — is the same across all four.

## API surface

`ctx.llm` is an instance of `agente.complemento_llm.ComplementoLlm`.

### `complete()`

```python
result = ctx.llm.complete(
    messages=[{"role": "user", "content": "Hi"}],
    proveedor=None,         # optional, gated — Hermes proveedor id (e.g. "openrouter")
    modelo=None,            # optional, gated — whatever string that proveedor expects
    temperature=None,
    max_tokens=None,
    timeout=None,          # seconds
    agente_id=None,         # optional, gated
    perfil=None,          # optional, gated — explicit auth-perfil name
    purpose="optional-audit-string",
)
# → ComplementoLlmCompleteResult(text, proveedor, modelo, agente_id, usage, audit)
```

Plain chat completion. `messages` is the standard OpenAI shape — a
list of `{"role": "...", "content": "..."}` dicts. Multi-turn
prompts (system + few-shot user/assistant pairs + final user) work
exactly as they would with the OpenAI SDK.

`proveedor=` and `modelo=` are independent and follow the same shape
as the host's main config (`modelo.proveedor` + `modelo.modelo`). Set
just `modelo=` to use the user's active proveedor with a different
modelo on it. Set both to switch proveedors entirely. Either argument
without operator opt-in raises `ComplementoLlmTrustError`.

### `complete_structured()`

```python
result = ctx.llm.complete_structured(
    instructions="What you want extracted.",
    input=[
        {"type": "text",  "text": "..."},
        {"type": "image", "data": b"...", "mime_type": "image/png"},
        {"type": "image", "url":  "https://..."},
    ],
    json_schema={...},     # optional — triggers parsed result + validation
    json_mode=False,       # set True without a schema to ask for JSON anyway
    schema_name=None,      # optional human-readable schema name
    system_prompt=None,
    proveedor=None,         # optional, gated
    modelo=None,            # optional, gated
    temperature=None,
    max_tokens=None,
    timeout=None,
    agente_id=None,
    perfil=None,
    purpose=None,
)
# → ComplementoLlmStructuredResult(text, proveedor, modelo, agente_id,
#                             usage, parsed, content_type, audit)
```

Inputs are typed text or image blocks (raw bytes get base64 encoded
as a `data:` URL automatically). When `json_schema` or
`json_mode=True` is supplied, the host requests JSON output via
`response_format`, parses it locally as a fallback, and validates
against your schema if `jsonschema` is installed.

* `result.content_type == "json"` — `result.parsed` is a Python
  object that matches your schema.
* `result.content_type == "text"` — parsing or validation failed;
  inspect `result.text` for the raw modelo response.

### Async

```python
result = await ctx.llm.acomplete(messages=...)
result = await ctx.llm.acomplete_structured(instructions=..., input=...)
```

Same arguments and result types as their sync counterparts. Use
these from puerta de enlace adapters, async hooks, or any complemento code
already running on an asyncio loop.

### Result attributes

```python
@dataclass
class ComplementoLlmCompleteResult:
    text: str                    # the assistant's response
    proveedor: str                # e.g. "openrouter", "anthropic"
    modelo: str                   # whatever the proveedor returned for this call
    agente_id: str                # whose modelo/auth was used
    usage: ComplementoLlmUsage        # tokens + cache + cost estimate
    audit: Dict[str, Any]        # complemento_id, purpose, perfil

@dataclass
class ComplementoLlmStructuredResult(ComplementoLlmCompleteResult):
    parsed: Optional[Any]        # JSON object when content_type == "json"
    content_type: str            # "json" or "text"
    # audit also carries schema_name when supplied
```

`usage` carries `input_tokens`, `output_tokens`, `total_tokens`,
`cache_read_tokens`, `cache_write_tokens`, and `cost_usd` when the
proveedor returns those fields.

## Trust gate

The default behaviour is fail-closed. With no `complementos.entries`
config block, a complemento can:

* run any of the four methods against the user's active proveedor
  and modelo,
* set request-shaping arguments (`temperature`, `max_tokens`,
  `timeout`, `system_prompt`, `purpose`, `messages`, `instructions`,
  `input`, `json_schema`),

…and that's it. `proveedor=`, `modelo=`, `agente_id=`, and `perfil=`
arguments raise `ComplementoLlmTrustError` until the operator opts in.

**Most complementos never need this section.** A complemento that just calls
`ctx.llm.complete(messages=...)` with no overrides runs against
whatever the user has active and works zero-config. The block below
is only relevant when a complemento specifically wants to pin to a
different modelo or proveedor than the user.

```yaml
complementos:
  entries:
    my-complemento:
      llm:
        # Allow this complemento to choose a different Hermes proveedor
        # (must be one Hermes already knows about — same names as
        # `hermes modelo` and config.yaml modelo.proveedor).
        allow_proveedor_override: true

        # Optionally restrict which proveedors. Use ["*"] for any.
        allowed_proveedors:
          - openrouter
          - anthropic

        # Allow this complemento to ask for a specific modelo.
        allow_modelo_override: true

        # Optionally restrict which modelos. Use ["*"] for any.
        # Modelos are matched literally against whatever string the
        # complemento sends — Hermes does not look anything up.
        allowed_modelos:
          - openai/gpt-4o-mini
          - anthropic/claude-3-5-haiku

        # Allow cross-agente calls (rare).
        allow_agente_id_override: false

        # Allow the complemento to request a specific stored auth perfil
        # (e.g. a different OAuth account on the same proveedor).
        allow_perfil_override: false
```

The complemento id is the manifest `name:` field for flat complementos, or the
path-derived key for nested complementos (`image_gen/openai`,
`memoria/honcho`, etc.).

### What the gate enforces

| Override        | Default | Config key                       |
| --------------- | ------- | -------------------------------- |
| `proveedor=`     | denied  | `allow_proveedor_override: true`  |
| ↳ allowlist     | —       | `allowed_proveedors: [...]`       |
| `modelo=`        | denied  | `allow_modelo_override: true`     |
| ↳ allowlist     | —       | `allowed_modelos: [...]`          |
| `agente_id=`     | denied  | `allow_agente_id_override: true`  |
| `perfil=`      | denied  | `allow_perfil_override: true`   |

Each override is independently gated. Granting `allow_modelo_override`
does **not** also grant `allow_proveedor_override` — a complemento trusted
to pick a modelo is still pinned to the user's active proveedor unless
it gets the proveedor gate as well.

### What the gate does NOT need to enforce

* Request-shaping arguments — `temperature`, `max_tokens`,
  `timeout`, `system_prompt`, `purpose`, `messages`, `instructions`,
  `input`, `json_schema`, `schema_name`, `json_mode` — are always
  allowed; they don't pick credentials or routes.
* The default deny posture means an unconfigured complemento can still do
  useful work — it just runs against the active proveedor and modelo.
  Operators only need to think about `complementos.entries` for complementos
  that want finer routing.

## What the host owns

A complete list of the things `ctx.llm` does for the complemento so you
don't have to:

* **Proveedor resolution.** Reads `modelo.proveedor` + `modelo.modelo`
  from the user's config (or the explicit overrides when trusted).
* **Auth.** Pulls API keys, OAuth tokens, or refresh tokens from
  `~/.hermes/auth.json` / env, including the credential pool when
  one is configured. The complemento never sees them.
* **Vision routing.** When image input is supplied and the user's
  active text modelo is text-only, the host falls back to the
  configured vision modelo automatically.
* **Fallback chain.** If the user's primary proveedor 5xxs or 429s,
  the request goes through Hermes' usual aggregator-aware fallback
  before it returns an error to the complemento.
* **Timeout.** Honours your `timeout=` argument, falling back to
  `auxiliary.<task>.timeout` config or the global aux default.
* **JSON shaping.** Sends `response_format` to the proveedor when
  you ask for JSON, then re-parses locally from a code-fenced
  response if the proveedor returned one.
* **Schema validation.** Validates against your `json_schema` when
  `jsonschema` is installed; logs a debug line and skips strict
  validation otherwise.
* **Audit log.** Each call writes one INFO line to `agente.log` with
  the complemento id, proveedor/modelo, purpose, and token totals.

## What the complemento owns

* **Request shape.** `messages` for chat, `instructions` + `input`
  for structured. The complemento builds the prompt; the host runs it.
* **Schema.** Whatever shape you want back. The host doesn't infer
  it for you.
* **Error handling.** `complete_structured()` raises `ValueError` on
  empty inputs and on schema-validation failure. `ComplementoLlmTrustError`
  fires when the trust gate denies an override. Anything else
  (proveedor 5xx, no credentials configured, timeout) raises whatever
  `auxiliary_client.call_llm()` raises.
* **Cost.** Every call runs against the user's paid proveedor. Don't
  loop on `complete()` for every puerta de enlace message without thinking
  about token spend.

## Where this fits in the complemento surface

Existing `ctx.*` methods extend an existing Hermes subsystem:

| `ctx.register_herramienta` | adds a herramienta the agente can call |
| `ctx.register_platform` | wires a new puerta de enlace adapter |
| `ctx.register_image_gen_proveedor` | replaces an image-gen backend |
| `ctx.register_memoria_proveedor` | replaces the memoria backend |
| `ctx.register_contexto_engine` | replaces the contexto compressor |
| `ctx.register_hook` | observes a lifecycle event |

`ctx.llm` is the first surface that lets a complemento run the same
modelo the user is talking to, *out of band*, without any of the
above. That's its only job. If your complemento needs to register a
herramienta the agente invokes, use `register_herramienta`. If it needs to react
to a lifecycle event, use `register_hook`. If it needs to make its
own modelo call — for any reason, structured or not — `ctx.llm`.

## Referencia

* Implementation: [`agente/complemento_llm.py`](https://github.com/NousResearch/hermes-agente/blob/main/agente/complemento_llm.py)
* Tests: [`tests/agente/test_complemento_llm.py`](https://github.com/NousResearch/hermes-agente/blob/main/tests/agente/test_complemento_llm.py)
* Reference complementos (companion repo):
  * [`complemento-llm-example`](https://github.com/NousResearch/hermes-example-complementos/tree/main/complemento-llm-example) — sync structured extraction with image input
  * [`complemento-llm-async-example`](https://github.com/NousResearch/hermes-example-complementos/tree/main/complemento-llm-async-example) — async with `asyncio.gather()`
* Auxiliary client (the engine under the hood): see
  [Proveedor Runtime](/guia-desarrollador/proveedor-runtime).

---