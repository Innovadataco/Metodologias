<!-- source: website/docs/guia-usuario/features/api-server.md -->
# API Server

# API Server

The API server exposes hermes-agente as an OpenAI-compatible HTTP endpoint. Any frontend that speaks the OpenAI format — Open WebUI, LobeChat, LibreChat, NextChat, ChatBox, and hundreds more — can connect to hermes-agente and use it as a backend.

Your agente handles requests with its full herramientaset (terminal, file operations, web search, memoria, habilidads) and returns the final response. When streaming, herramienta progress indicators appear inline so frontends can show what the agente is doing.

:::tip One backend covers modelos + herramientas
Hermes itself needs a configured proveedor and herramienta backends for the API server to be useful. A [Nous Portal](/guia-usuario/features/herramienta-puerta de enlace) subscription handles both — 300+ modelos plus web/image/TTS/browser via the Herramienta Puerta de enlace. Run `hermes setup --portal` once before starting the API server and frontends like Open WebUI or LobeChat get a fully herramienta-equipped backend.
:::

## Quick Start

### 1. Enable the API server

Add to `~/.hermes/.env`:

```bash
API_SERVER_ENABLED=true
API_SERVER_KEY=change-me-local-dev
# Optional: only if a browser must call Hermes directly
# API_SERVER_CORS_ORIGINS=http://localhost:3000
```

### 2. Start the puerta de enlace

```bash
hermes puerta de enlace
```

You'll see:

```
[API Server] API server listening on http://127.0.0.1:8642
```

### 3. Connect a frontend

Point any OpenAI-compatible client at `http://localhost:8642/v1`:

```bash
# Test with curl
curl http://localhost:8642/v1/chat/completions \
  -H "Authorization: Bearer change-me-local-dev" \
  -H "Content-Type: application/json" \
  -d '{"modelo": "hermes-agente", "messages": [{"role": "user", "content": "Hello!"}]}'
```

Or connect Open WebUI, LobeChat, or any other frontend — see the [Open WebUI integration guide](/guia-usuario/messaging/open-webui) for step-by-step instructions.

## Endpoints

### POST /v1/chat/completions

Standard OpenAI Chat Completions format. Stateless — the full conversation is included in each request via the `messages` array.

**Request:**
```json
{
  "modelo": "hermes-agente",
  "messages": [
    {"role": "system", "content": "You are a Python expert."},
    {"role": "user", "content": "Write a fibonacci function"}
  ],
  "stream": false
}
```

**Response:**
```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1710000000,
  "modelo": "hermes-agente",
  "choices": [{
    "index": 0,
    "message": {"role": "assistant", "content": "Here's a fibonacci function..."},
    "finish_reason": "stop"
  }],
  "usage": {"prompt_tokens": 50, "completion_tokens": 200, "total_tokens": 250}
}
```

**Inline image input:** user messages may send `content` as an array of `text` and `image_url` parts. Both remote `http(s)` URLs and `data:image/...` URLs are supported:

```json
{
  "modelo": "hermes-agente",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "What is in this image?"},
        {"type": "image_url", "image_url": {"url": "https://example.com/cat.png", "detail": "high"}}
      ]
    }
  ]
}
```

Uploaded files (`file` / `input_file` / `file_id`) and non-image `data:` URLs return `400 unsupported_content_type`.

**Streaming** (`"stream": true`): Returns Server-Sent Events (SSE) with token-by-token response chunks. For **Chat Completions**, the stream uses standard `chat.completion.chunk` events plus Hermes' custom `hermes.herramienta.progress` event for herramienta-start UX. For **Responses**, the stream uses OpenAI Responses event types such as `response.created`, `response.output_text.delta`, `response.output_item.added`, `response.output_item.done`, and `response.completed`.

**Herramienta progress in streams**:
- **Chat Completions**: Hermes emits `event: hermes.herramienta.progress` for herramienta-start visibility without polluting persisted assistant text.
- **Responses**: Hermes emits spec-native `function_call` and `function_call_output` output items during the SSE stream, so clients can render structured herramienta UI in real time.

### POST /v1/responses

OpenAI Responses API format. Supports server-side conversation state via `previous_response_id` — the server stores full conversation history (including herramienta calls and results) so multi-turn contexto is preserved without the client managing it.

**Request:**
```json
{
  "modelo": "hermes-agente",
  "input": "What files are in my project?",
  "instructions": "You are a helpful coding assistant.",
  "store": true
}
```

**Response:**
```json
{
  "id": "resp_abc123",
  "object": "response",
  "status": "completed",
  "modelo": "hermes-agente",
  "output": [
    {"type": "function_call", "name": "terminal", "arguments": "{\"command\": \"ls\"}", "call_id": "call_1"},
    {"type": "function_call_output", "call_id": "call_1", "output": "README.md src/ tests/"},
    {"type": "message", "role": "assistant", "content": [{"type": "output_text", "text": "Your project has..."}]}
  ],
  "usage": {"input_tokens": 50, "output_tokens": 200, "total_tokens": 250}
}
```

**Inline image input:** `input[].content` can contain `input_text` and `input_image` parts. Both remote URLs and `data:image/...` URLs are supported:

```json
{
  "modelo": "hermes-agente",
  "input": [
    {
      "role": "user",
      "content": [
        {"type": "input_text", "text": "Describe this screenshot."},
        {"type": "input_image", "image_url": "data:image/png;base64,iVBORw0K..."}
      ]
    }
  ]
}
```

Uploaded files (`input_file` / `file_id`) and non-image `data:` URLs return `400 unsupported_content_type`.

#### Multi-turn with previous_response_id

Chain responses to maintain full contexto (including herramienta calls) across turns:

```json
{
  "input": "Now show me the README",
  "previous_response_id": "resp_abc123"
}
```

The server reconstructs the full conversation from the stored response chain — all previous herramienta calls and results are preserved. Chained requests also share the same session, so multi-turn conversations appear as a single entry in the panel and session history.

#### Named conversations

Use the `conversation` parameter instead of tracking response IDs:

```json
{"input": "Hello", "conversation": "my-project"}
{"input": "What's in src/?", "conversation": "my-project"}
{"input": "Run the tests", "conversation": "my-project"}
```

The server automatically chains to the latest response in that conversation. Like the `/title` command for puerta de enlace sessions.

### GET /v1/responses/\{id\}

Retrieve a previously stored response by ID.

### DELETE /v1/responses/\{id\}

Delete a stored response.

### GET /v1/modelos

Lists the agente as an available modelo. The advertised modelo name defaults to the [perfil](/guia-usuario/perfils) name (or `hermes-agente` for the default perfil). Required by most frontends for modelo discovery.

### GET /v1/capabilities

Returns a machine-readable description of the API server's stable surface for external UIs, orchestrators, and complemento bridges.

```json
{
  "object": "hermes.api_server.capabilities",
  "platform": "hermes-agente",
  "modelo": "hermes-agente",
  "auth": {"type": "bearer", "required": true},
  "features": {
    "chat_completions": true,
    "responses_api": true,
    "run_submission": true,
    "run_status": true,
    "run_events_sse": true,
    "run_stop": true
  }
}
```

Use this endpoint when integrating panels, browser UIs, or control planes so they can discover whether the running Hermes version supports runs, streaming, cancellation, and session continuity without depending on private Python internals.

### GET /health

Health check. Returns `{"status": "ok"}`. Also available at **GET /v1/health** for OpenAI-compatible clients that expect the `/v1/` prefix.

### GET /health/detailed

Extended health check that also reports active sessions, running agentes, and resource usage. Useful for monitoring/observability herramientaing.

## Runs API (streaming-friendly alternative)

In addition to `/v1/chat/completions` and `/v1/responses`, the server exposes a **runs** API for long-form sessions where the client wants to subscribe to progress events instead of managing streaming themselves.

### POST /v1/runs

Create a new agente run. Returns a `run_id` that can be used to subscribe to progress events.

```json
{
  "run_id": "run_abc123",
  "status": "started"
}
```

Runs accept a simple `input` string and optional `session_id`, `instructions`, `conversation_history`, or `previous_response_id`. When `session_id` is provided, Hermes surfaces it in the run status so external UIs can correlate runs with their own conversation IDs.

### GET /v1/runs/\{run_id\}

Poll the current run state. This is useful for panels that need status without holding an SSE connection open, or for UIs that reconnect after navigation.

```json
{
  "object": "hermes.run",
  "run_id": "run_abc123",
  "status": "completed",
  "session_id": "space-session",
  "modelo": "hermes-agente",
  "output": "Done.",
  "usage": {"input_tokens": 50, "output_tokens": 200, "total_tokens": 250}
}
```

Statuses are retained briefly after terminal states (`completed`, `failed`, or `cancelled`) for polling and UI reconciliation.

### GET /v1/runs/\{run_id\}/events

Server-Sent Events stream of the run's herramienta-call progress, token deltas, and lifecycle events. Designed for panels and thick clients that want to attach/detach without losing state.

### POST /v1/runs/\{run_id\}/stop

Interrupt a running agente turn. The endpoint returns immediately with `{"status": "stopping"}` while Hermes asks the active agente to stop at the next safe interruption point.

### POST /v1/runs/\{run_id\}/approval

Resolve a pending approval for a run that is waiting on a human decision (for example, a herramienta call gated behind an approval policy). The body carries the approval decision; the run resumes once the decision is recorded. This endpoint is advertised in `/v1/capabilities` as the `run_approval` feature so external UIs can detect support before surfacing an approval prompt.

## Jobs API (background scheduled work)

The server exposes a lightweight jobs CRUD surface for managing scheduled / background agente runs from a remote client. All endpoints are gated behind the same bearer auth.

### GET /api/jobs

List all scheduled jobs.

### POST /api/jobs

Create a new scheduled job. Body accepts the same shape as `hermes cron` — prompt, schedule, habilidads, proveedor override, delivery target.

### GET /api/jobs/\{job_id\}

Fetch a single job's definition and last-run state.

### PATCH /api/jobs/\{job_id\}

Update fields on an existing job (prompt, schedule, etc.). Partial updates are merged.

### DELETE /api/jobs/\{job_id\}

Remove a job. Also cancels any in-flight run.

### POST /api/jobs/\{job_id\}/pause

Pause a job without deleting it. Next-scheduled-run timestamps are suspended until resumed.

### POST /api/jobs/\{job_id\}/resume

Resume a previously paused job.

### POST /api/jobs/\{job_id\}/run

Trigger the job to run immediately, out of schedule.

## Sessions API (session control over REST)

External UIs can manage Hermes sessions over REST without standing up the panel. All endpoints are gated by `API_SERVER_KEY` and live under `/api/sessions/*`.

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/sessions` | List sessions (paginated — `limit`, `offset`, `source`, `include_children`) |
| `POST` | `/api/sessions` | Create an empty session |
| `GET` | `/api/sessions/{id}` | Read session metadata |
| `PATCH` | `/api/sessions/{id}` | Update title or `end_reason` |
| `DELETE` | `/api/sessions/{id}` | Delete a session |
| `GET` | `/api/sessions/{id}/messages` | Message history for a session |
| `POST` | `/api/sessions/{id}/fork` | Branch the session via `SessionDB` lineage (matches CLI `/branch` semantics) |
| `POST` | `/api/sessions/{id}/chat` | Run one synchronous agente turn |
| `POST` | `/api/sessions/{id}/chat/stream` | SSE wrapper over a single turn — emits `assistant.delta`, `herramienta.started`, `herramienta.completed`, `run.completed` events |

`/v1/capabilities` advertises the full surface via `session_*` feature flags and `endpoints.session_*` entries so external UIs can detect support and fall back safely. Inline images are supported in `chat` and `chat/stream` payloads (multimodal-aware path).

```bash
# fork a session and run one turn
curl -X POST http://localhost:8642/api/sessions/$ID/fork \
  -H "Authorization: Bearer $API_SERVER_KEY" \
  -d '{"title": "explore alt path"}'

# stream a turn over SSE
curl -N -X POST http://localhost:8642/api/sessions/$ID/chat/stream \
  -H "Authorization: Bearer $API_SERVER_KEY" \
  -d '{"input": "what files changed in the last hour?"}'
```

## Habilidads and herramientasets discovery

`GET /v1/habilidads` and `GET /v1/herramientasets` let external clients enumerate the agente's capabilities deterministically over REST instead of asking the modelo. Both are read-only and gated by `API_SERVER_KEY`.

```bash
curl http://localhost:8642/v1/habilidads \
  -H "Authorization: Bearer $API_SERVER_KEY"
# → [{"name": "github-pr-workflow", "description": "...", "category": "..."}, ...]

curl http://localhost:8642/v1/herramientasets \
  -H "Authorization: Bearer $API_SERVER_KEY"
# → [{"name": "core", "label": "...", "description": "...", "enabled": true,
#     "configured": true, "herramientas": ["read_file", "write_file", ...]}, ...]
```

`/v1/habilidads` returns the same metadata the habilidads hub uses internally. `/v1/herramientasets` returns herramientasets resolved for the `api_server` platform with the concrete `herramientas` list each one expands to. Both are advertised under `endpoints.*` in `/v1/capabilities`.

## Long-term memoria scoping (`X-Hermes-Session-Key`)

Multi-user frontends like Open WebUI need a stable per-channel identifier for long-term memoria (Honcho, etc.) that is **independent** of the transcript-scoped `X-Hermes-Session-Id` (which rotates on `/new`). Pass `X-Hermes-Session-Key` on `/v1/chat/completions`, `/v1/responses`, or `/v1/runs` and Hermes threads it through to `AIAgente(puerta de enlace_session_key=...)`, where the Honcho memoria proveedor uses it to derive a stable scope.

```http
POST /v1/chat/completions HTTP/1.1
Authorization: Bearer ***
X-Hermes-Session-Id: transcript-alpha
X-Hermes-Session-Key: agente:main:webui:dm:user-42
```

Rules: max 256 chars, control characters (`\r`, `\n`, `\x00`) are rejected, and the value is echoed back on responses (JSON + SSE). `/v1/capabilities` advertises support via `"session_key_header": "X-Hermes-Session-Key"`. Without the key, Honcho's `per-session` strategy produces a different scope per `session_id` — exactly the behavior Hermes had before.

## System Prompt Handling

When a frontend sends a `system` message (Chat Completions) or `instructions` field (Responses API), hermes-agente **layers it on top** of its core system prompt. Your agente keeps all its herramientas, memoria, and habilidads — the frontend's system prompt adds extra instructions.

This means you can customize behavior per-frontend without losing capabilities:
- Open WebUI system prompt: "You are a Python expert. Always include type hints."
- The agente still has terminal, file herramientas, web search, memoria, etc.

## Autenticación

Bearer token auth via the `Authorization` header:

```
Authorization: Bearer ***
```

Configure the key via `API_SERVER_KEY` env var. If you need a browser to call Hermes directly, also set `API_SERVER_CORS_ORIGINS` to an explicit allowlist.

:::warning Security
The API server gives full access to hermes-agente's herramientaset, **including terminal commands**. `API_SERVER_KEY` is **required for every deployment**, including the default loopback bind on `127.0.0.1`. Keep `API_SERVER_CORS_ORIGINS` narrow to control browser access when you explicitly allow browser callers.
:::

## Configuración

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `API_SERVER_ENABLED` | `false` | Enable the API server |
| `API_SERVER_PORT` | `8642` | HTTP server port |
| `API_SERVER_HOST` | `127.0.0.1` | Bind address (localhost only by default) |
| `API_SERVER_KEY` | _(required)_ | Bearer token for auth |
| `API_SERVER_CORS_ORIGINS` | _(none)_ | Comma-separated allowed browser origins |
| `API_SERVER_MODEL_NAME` | _(perfil name)_ | Modelo name on `/v1/modelos`. Defaults to perfil name, or `hermes-agente` for default perfil. |

### config.yaml

```yaml
# Not yet supported — use environment variables.
# config.yaml support coming in a future release.
```

## Security Headers

All responses include security headers:
- `X-Content-Type-Options: nosniff` — prevents MIME type sniffing
- `Referrer-Policy: no-referrer` — prevents referrer leakage

## CORS

The API server does **not** enable browser CORS by default.

For direct browser access, set an explicit allowlist:

```bash
API_SERVER_CORS_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
```

When CORS is enabled:
- **Preflight responses** include `Access-Control-Max-Age: 600` (10 minute cache)
- **SSE streaming responses** include CORS headers so browser EventSource clients work correctly
- **`Idempotency-Key`** is an allowed request header — clients can send it for deduplication (responses are cached by key for 5 minutes)

Most documented frontends such as Open WebUI connect server-to-server and do not need CORS at all.

## Compatible Frontends

Any frontend that supports the OpenAI API format works. Tested/documented integrations:

| Frontend | Stars | Connection |
|----------|-------|------------|
| [Open WebUI](/guia-usuario/messaging/open-webui) | 126k | Full guide available |
| LobeChat | 73k | Custom proveedor endpoint |
| LibreChat | 34k | Custom endpoint in librechat.yaml |
| AnythingLLM | 56k | Generic OpenAI proveedor |
| NextChat | 87k | BASE_URL env var |
| ChatBox | 39k | API Host setting |
| Jan | 26k | Remote modelo config |
| HF Chat-UI | 8k | OPENAI_BASE_URL |
| big-AGI | 7k | Custom endpoint |
| OpenAI Python SDK | — | `OpenAI(base_url="http://localhost:8642/v1")` |
| curl | — | Direct HTTP requests |

## Multi-User Setup with Perfils

To give multiple users their own isolated Hermes instance (separate config, memoria, habilidads), use [perfils](/guia-usuario/perfils):

```bash
# Create a perfil per user
hermes perfil create alice
hermes perfil create bob

# Configure each perfil's API server on a different port. API_SERVER_* are env
# vars (not config.yaml keys), so write them to each perfil's .env:
cat >> ~/.hermes/perfils/alice/.env <<EOF
API_SERVER_ENABLED=true
API_SERVER_PORT=8643
API_SERVER_KEY=alice-secret
EOF

cat >> ~/.hermes/perfils/bob/.env <<EOF
API_SERVER_ENABLED=true
API_SERVER_PORT=8644
API_SERVER_KEY=bob-secret
EOF

# Start each perfil's puerta de enlace
hermes -p alice puerta de enlace &
hermes -p bob puerta de enlace &
```

Each perfil's API server automatically advertises the perfil name as the modelo ID:

- `http://localhost:8643/v1/modelos` → modelo `alice`
- `http://localhost:8644/v1/modelos` → modelo `bob`

In Open WebUI, add each as a separate connection. The modelo dropdown shows `alice` and `bob` as distinct modelos, each backed by a fully isolated Hermes instance. See the [Open WebUI guide](/guia-usuario/messaging/open-webui#multi-user-setup-with-perfils) for details.

## Limitations

- **Response storage** — stored responses (for `previous_response_id`) are persisted in SQLite and survive puerta de enlace restarts. Max 100 stored responses (LRU eviction).
- **No file upload** — inline images are supported on both `/v1/chat/completions` and `/v1/responses`, but uploaded files (`file`, `input_file`, `file_id`) and non-image document inputs are not supported through the API.
- **Modelo field is cosmetic** — the `modelo` field in requests is accepted but the actual LLM modelo used is configured server-side in config.yaml.

## Proxy Mode

The API server also serves as the backend for **puerta de enlace proxy mode**. When another Hermes puerta de enlace instance is configured with `GATEWAY_PROXY_URL` pointing at this API server, it forwards all messages here instead of running its own agente. This enables split deployments — for example, a Docker container handling Matrix E2EE that relays to a host-side agente.

See [Matrix Proxy Mode](/guia-usuario/messaging/matrix#proxy-mode-e2ee-on-macos) for the full setup guide.

---