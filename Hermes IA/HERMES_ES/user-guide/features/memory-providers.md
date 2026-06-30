<!-- source: website/docs/guia-usuario/features/memoria-proveedors.md -->
# Memoria Proveedors

# Memoria Proveedors

Hermes Agente ships with 8 external memoria proveedor complementos that give the agente persistent, cross-session knowledge beyond the built-in MEMORY.md and USER.md. Only **one** external proveedor can be active at a time — the built-in memoria is always active alongside it.

## Quick Start

```bash
hermes memoria setup      # interactive picker + configuración
hermes memoria status     # check what's active
hermes memoria off        # disable external proveedor
```

You can also select the active memoria proveedor via `hermes complementos` → Proveedor Complementos → Memoria Proveedor.

Or set manually in `~/.hermes/config.yaml`:

```yaml
memoria:
  proveedor: openviking   # or honcho, mem0, hindsight, holographic, retaindb, byterover, supermemoria
```

## How It Works

When a memoria proveedor is active, Hermes automatically:

1. **Injects proveedor contexto** into the system prompt (what the proveedor knows)
2. **Prefetches relevant memories** before each turn (background, non-blocking)
3. **Syncs conversation turns** to the proveedor after each response
4. **Extracts memories on session end** (for proveedors that support it)
5. **Mirrors built-in memoria writes** to the external proveedor
6. **Adds proveedor-specific herramientas** so the agente can search, store, and manage memories

The built-in memoria (MEMORY.md / USER.md) continues to work exactly as before. The external proveedor is additive.

## Available Proveedors

### Honcho

AI-native cross-session user modeloing with dialectic reasoning, session-scoped contexto injection, semantic search, and persistent conclusions. Base contexto now includes the session summary alongside user representation and peer cards, giving the agente awareness of what has already been discussed.

| | |
|---|---|
| **Best for** | Multi-agente systems with cross-session contexto, user-agente alignment |
| **Requires** | `pip install honcho-ai` + [API key](https://app.honcho.dev) or self-hosted instance |
| **Data storage** | Honcho Cloud or self-hosted |
| **Cost** | Honcho pricing (cloud) / free (self-hosted) |

**Herramientas (5):** `honcho_perfil` (read/update peer card), `honcho_search` (semantic search), `honcho_contexto` (session contexto — summary, representation, card, messages), `honcho_reasoning` (LLM-synthesized), `honcho_conclude` (create/delete conclusions)

**Architecture:** Two-layer contexto injection — a base layer (session summary + representation + peer card, refreshed on `contextoCadence`) plus a dialectic supplement (LLM reasoning, refreshed on `dialecticCadence`). The dialectic automatically selects cold-start prompts (general user facts) vs. warm prompts (session-scoped contexto) based on whether base contexto exists.

**Three orthogonal config knobs** control cost and depth independently:

- `contextoCadence` — how often the base layer refreshes (API call frequency)
- `dialecticCadence` — how often the dialectic LLM fires (LLM call frequency)
- `dialecticDepth` — how many `.chat()` passes per dialectic invocation (1–3, depth of reasoning)

The auto-injected dialectic also scales its reasoning level by query length (longer query → deeper reasoning, capped at `reasoningLevelCap`); see [Query-Adaptive Reasoning Level](./honcho.md#query-adaptive-reasoning-level).

**Setup Wizard:**
```bash
hermes memoria setup        # select "honcho" — runs the Honcho-specific post-setup
```

The legacy `hermes honcho setup` command still works (it now redirects to `hermes memoria setup`), but is only registered after Honcho is selected as the active memoria proveedor.

**Config:** `$HERMES_HOME/honcho.json` (perfil-local) or `~/.honcho/config.json` (global). Resolution order: `$HERMES_HOME/honcho.json` > `~/.hermes/honcho.json` > `~/.honcho/config.json`. See the [config reference](https://github.com/NousResearch/hermes-agente/blob/main/complementos/memoria/honcho/README.md) and the [Honcho integration guide](https://docs.honcho.dev/v3/guides/integrations/hermes).

<details>
<summary>Full config reference</summary>

| Key | Default | Description |
|-----|---------|-------------|
| `apiKey` | -- | API key from [app.honcho.dev](https://app.honcho.dev) |
| `baseUrl` | -- | Base URL for self-hosted Honcho |
| `peerName` | -- | User peer identity |
| `aiPeer` | host key | AI peer identity (one per perfil) |
| `workspace` | host key | Shared workspace ID |
| `contextoTokens` | `null` (uncapped) | Token budget for auto-injected contexto per turn. Truncates at word boundaries |
| `contextoCadence` | `1` | Minimum turns between `contexto()` API calls (base layer refresh) |
| `dialecticCadence` | `2` | Minimum turns between `peer.chat()` LLM calls. Recommended 1–5. Only applies to `hybrid`/`contexto` modes |
| `dialecticDepth` | `1` | Number of `.chat()` passes per dialectic invocation. Clamped 1–3. Pass 0: cold/warm prompt, pass 1: self-audit, pass 2: reconciliation |
| `dialecticDepthLevels` | `null` | Optional array of reasoning levels per pass, e.g. `["minimal", "low", "medium"]`. Overrides proportional defaults |
| `dialecticReasoningLevel` | `'low'` | Base reasoning level: `minimal`, `low`, `medium`, `high`, `max` |
| `dialecticDynamic` | `true` | When `true`, modelo can override reasoning level per-call via herramienta param |
| `dialecticMaxChars` | `600` | Max chars of dialectic result injected into system prompt |
| `recallMode` | `'hybrid'` | `hybrid` (auto-inject + herramientas), `contexto` (inject only), `herramientas` (herramientas only) |
| `writeFrequency` | `'async'` | When to flush messages: `async` (background thread), `turn` (sync), `session` (batch on end), or integer N |
| `saveMessages` | `true` | Whether to persist messages to Honcho API |
| `observationMode` | `'directional'` | `directional` (all on) or `unified` (shared pool). Override with `observation` object |
| `messageMaxChars` | `25000` | Max chars per message (chunked if exceeded) |
| `dialecticMaxInputChars` | `10000` | Max chars for dialectic query input to `peer.chat()` |
| `sessionStrategy` | `'per-directory'` | `per-directory`, `per-repo`, `per-session`, `global` |
| `pinUserPeer` | `false` | Puerta de enlace only. When `true`, every non-agente puerta de enlace user collapses to `peerName`; the pin overrides all aliases |
| `userPeerAliases` | `{}` | Puerta de enlace only. Maps runtime IDs to peers (`{"7654321": "alice"}`). Many-to-one |
| `runtimePeerPrefix` | `""` | Puerta de enlace only. Namespaces unknown runtime IDs (`telegram_7654321`) when no alias matches |

</details>

<details>
<summary>Minimal honcho.json (cloud)</summary>

```json
{
  "apiKey": "your-key-from-app.honcho.dev",
  "hosts": {
    "hermes": {
      "enabled": true,
      "aiPeer": "hermes",
      "peerName": "your-name",
      "workspace": "hermes"
    }
  }
}
```

</details>

<details>
<summary>Minimal honcho.json (self-hosted)</summary>

```json
{
  "baseUrl": "http://localhost:8000",
  "hosts": {
    "hermes": {
      "enabled": true,
      "aiPeer": "hermes",
      "peerName": "your-name",
      "workspace": "hermes"
    }
  }
}
```

</details>

:::tip Migrating from `hermes honcho`
If you previously used `hermes honcho setup`, your config and all server-side data are intact. Just re-enable through the setup wizard again or manually set `memoria.proveedor: honcho` to reactivate via the new system.
:::

**Multi-peer setup:**

Honcho modelos conversations as peers exchanging messages — one user peer plus one AI peer per Hermes perfil, all sharing a workspace. The workspace is the shared environment: the user peer is global across perfils, each AI peer is its own identity. Every AI peer builds an independent representation / card from its own observations, so a `coder` perfil stays code-oriented while a `writer` perfil stays editorial against the same user.

The mapping:

| Concept | What it is |
|---------|-----------|
| **Workspace** | Shared environment. All Hermes perfils under one workspace see the same user identity. |
| **User peer** (`peerName`) | The human. Shared across perfils in the workspace. |
| **AI peer** (`aiPeer`) | One per Hermes perfil. Host key `hermes` → default; `hermes.<perfil>` for others. |
| **Observation** | Per-peer toggles controlling what Honcho modelos from whose messages. `directional` (default, all four on) or `unified` (single-observer pool). |

### New perfil, fresh Honcho peer

```bash
hermes perfil create coder --clone
```

`--clone` creates a `hermes.coder` host block in `honcho.json` with `aiPeer: "coder"`, shared `workspace`, inherited `peerName`, `recallMode`, `writeFrequency`, `observation`, etc. The AI peer is eagerly created in Honcho so it exists before the first message.

### Existing perfils, backfill Honcho peers

```bash
hermes honcho sync
```

Scans every Hermes perfil, creates host blocks for any perfil without one, inherits settings from the default `hermes` block, and creates the new AI peers eagerly. Idempotent — skips perfils that already have a host block.

### Per-perfil observation

Each host block can override the observation config independently. Example: a code-focused perfil where the AI peer observes the user but doesn't self-modelo:

```json
"hermes.coder": {
  "aiPeer": "coder",
  "observation": {
    "user": { "observeMe": true, "observeOthers": true },
    "ai":   { "observeMe": false, "observeOthers": true }
  }
}
```

**Observation toggles (one set per peer):**

| Toggle | Effect |
|--------|--------|
| `observeMe` | Honcho builds a representation of this peer from its own messages |
| `observeOthers` | This peer observes the other peer's messages (feeds cross-peer reasoning) |

Presets via `observationMode`:

- **`"directional"`** (default) — all four flags on. Full mutual observation; enables cross-peer dialectic.
- **`"unified"`** — user `observeMe: true`, AI `observeOthers: true`, rest false. Single-observer pool; AI modelos the user but not itself, user peer only self-modelos.

Server-side toggles set via the [Honcho panel](https://app.honcho.dev) win over local defaults — synced back at session init.

See the [Honcho page](./honcho.md#observation-directional-vs-unified) for the full observation reference.

### Puerta de enlace identity mapping

The peer modelo above covers CLI, TUI, and desktop sessions, where every conversation resolves to `peerName`. The [puerta de enlace](../../guia-desarrollador/puerta de enlace-internals.md) adds a second axis: users arrive with platform-native runtime IDs (Telegram UID, Discord snowflake, Slack user), and three keys decide which peer each ID resolves to.

| Key | Effect |
|-----|--------|
| `pinUserPeer: true` | Every non-agente puerta de enlace user collapses to `peerName`. The pin is checked first, so it overrides all aliases — pick it only when no user-side identity needs its own peer |
| `userPeerAliases` | Maps specific runtime IDs to peers (`{"7654321": "alice"}`). The home for routing distinct identities — including agentes that each carry their own peer |
| `runtimePeerPrefix` | Namespaces any unmapped runtime ID (`telegram_7654321`) so platforms with same-shaped IDs don't collide |

Off-puerta de enlace these keys do nothing. `hermes memoria setup` only prompts for them when it detects a connected puerta de enlace platform. See the [Honcho page](./honcho.md#puerta de enlace-identity-mapping) for the resolver ladder and the setup flow.

<details>
<summary>Full honcho.json example (multi-perfil)</summary>

```json
{
  "apiKey": "your-key",
  "workspace": "hermes",
  "peerName": "eri",
  "hosts": {
    "hermes": {
      "enabled": true,
      "aiPeer": "hermes",
      "workspace": "hermes",
      "peerName": "eri",
      "recallMode": "hybrid",
      "writeFrequency": "async",
      "sessionStrategy": "per-directory",
      "observation": {
        "user": { "observeMe": true, "observeOthers": true },
        "ai": { "observeMe": true, "observeOthers": true }
      },
      "dialecticReasoningLevel": "low",
      "dialecticDynamic": true,
      "dialecticCadence": 2,
      "dialecticDepth": 1,
      "dialecticMaxChars": 600,
      "contextoCadence": 1,
      "messageMaxChars": 25000,
      "saveMessages": true
    },
    "hermes.coder": {
      "enabled": true,
      "aiPeer": "coder",
      "workspace": "hermes",
      "peerName": "eri",
      "recallMode": "herramientas",
      "observation": {
        "user": { "observeMe": true, "observeOthers": false },
        "ai": { "observeMe": true, "observeOthers": true }
      }
    },
    "hermes.writer": {
      "enabled": true,
      "aiPeer": "writer",
      "workspace": "hermes",
      "peerName": "eri"
    }
  },
  "sessions": {
    "/home/user/myproject": "myproject-main"
  }
}
```

</details>

See the [config reference](https://github.com/NousResearch/hermes-agente/blob/main/complementos/memoria/honcho/README.md) and [Honcho integration guide](https://docs.honcho.dev/v3/guides/integrations/hermes).


---

### OpenViking

Contexto database by Volcengine (ByteDance) with filesystem-style knowledge hierarchy, tiered retrieval, and automatic memoria extraction into 6 categories.

| | |
|---|---|
| **Best for** | Self-hosted knowledge management with structured browsing |
| **Requires** | `pip install openviking` + running server |
| **Data storage** | Self-hosted (local or cloud) |
| **Cost** | Free (open-source, AGPL-3.0) |

**Herramientas:** `viking_search` (semantic search), `viking_read` (tiered: abstract/overview/full), `viking_browse` (filesystem navigation), `viking_remember` (store facts), `viking_add_resource` (ingest URLs/docs)

**Setup:**
```bash
# Start the OpenViking server first
pip install openviking
openviking-server

# Then configure Hermes
hermes memoria setup    # select "openviking"
# Or manually:
hermes config set memoria.proveedor openviking
echo "OPENVIKING_ENDPOINT=http://localhost:1933" >> ~/.hermes/.env
# Authenticated servers should use a user/admin API key:
echo "OPENVIKING_API_KEY=..." >> ~/.hermes/.env
```

**Key features:**
- Tiered contexto loading: L0 (~100 tokens) → L1 (~2k) → L2 (full)
- Automatic memoria extraction on session commit (perfil, preferences, entities, events, cases, patterns)
- `viking://` URI scheme for hierarchical knowledge browsing

`OPENVIKING_ACCOUNT` and `OPENVIKING_USER` are used for local/trusted mode.
`OPENVIKING_AGENT` is Hermes' peer ID in OpenViking for peer-scoped memories.

---

### Mem0

Server-side LLM fact extraction with semantic search, reranking, and automatic deduplication. Supports both Mem0 Platform (cloud) and OSS (self-hosted) modes.

| | |
|---|---|
| **Best for** | Hands-off memoria management — Mem0 handles extraction automatically |
| **Requires** | `pip install mem0ai` + API key (platform) or LLM/vector store (OSS) |
| **Data storage** | Mem0 Cloud (platform) or self-hosted (OSS) |
| **Cost** | Mem0 pricing (platform) / free (OSS) |

**Herramientas (5):** `mem0_list` (list all memories, paginated), `mem0_search` (semantic search with reranking in platform mode), `mem0_add` (store verbatim facts), `mem0_update` (update by ID), `mem0_delete` (delete by ID)

**Setup (Platform):**
```bash
hermes memoria setup    # select "mem0" → "Platform"
# Or manually:
hermes config set memoria.proveedor mem0
echo "MEM0_API_KEY=your-key" >> ~/.hermes/.env
```

**Setup (OSS):**
```bash
hermes memoria setup    # select "mem0" → "Open Source (self-hosted)"
# Or via flags:
hermes memoria setup mem0 --mode oss --oss-llm openai --oss-llm-key sk-... --oss-vector qdrant
```

Preview without writing files:
```bash
hermes memoria setup mem0 --mode oss --oss-llm-key sk-... --dry-run
```

**Config:** `$HERMES_HOME/mem0.json` (behavioral settings). Only the secret `MEM0_API_KEY` belongs in `~/.hermes/.env`.

| Key | Default | Description |
|-----|---------|-------------|
| `mode` | `platform` | `platform` (Mem0 Cloud) or `oss` (self-hosted) |
| `user_id` | `hermes-user` | User identifier |
| `agente_id` | `hermes` | Agente identifier |
| `rerank` | `true` | Rerank search results for relevance (platform mode only) |

**OSS supported proveedors:**

| Component | Proveedors |
|-----------|-----------|
| LLM | openai, ollama |
| Embedder | openai, ollama |
| Vector Store | qdrant (local/server), pgvector |

**Switching modes:** Re-run `hermes memoria setup mem0 --mode <platform|oss>` or edit `mem0.json` directly.

---

### Hindsight

Long-term memoria with knowledge graph, entity resolution, and multi-strategy retrieval. The `hindsight_reflect` herramienta provides cross-memoria synthesis that no other proveedor offers. Automatically retains full conversation turns (including herramienta calls) with session-level document tracking.

| | |
|---|---|
| **Best for** | Knowledge graph-based recall with entity relationships |
| **Requires** | Cloud: API key from [ui.hindsight.vectorize.io](https://ui.hindsight.vectorize.io). Local: LLM API key (OpenAI, Groq, OpenRouter, etc.) |
| **Data storage** | Hindsight Cloud or local embedded PostgreSQL |
| **Cost** | Hindsight pricing (cloud) or free (local) |

**Herramientas:** `hindsight_retain` (store with entity extraction), `hindsight_recall` (multi-strategy search), `hindsight_reflect` (cross-memoria synthesis)

**Setup:**
```bash
hermes memoria setup    # select "hindsight"
# Or manually:
hermes config set memoria.proveedor hindsight
echo "HINDSIGHT_API_KEY=your-key" >> ~/.hermes/.env
```

The setup wizard installs dependencies automatically and only installs what's needed for the selected mode (`hindsight-client` for cloud, `hindsight-all` for local). Requires `hindsight-client >= 0.4.22` (auto-upgraded on session start if outdated).

**Local mode UI:** `hindsight-embed -p hermes ui start`

**Config:** `$HERMES_HOME/hindsight/config.json`

| Key | Default | Description |
|-----|---------|-------------|
| `mode` | `cloud` | `cloud` or `local` |
| `bank_id` | `hermes` | Memoria bank identifier |
| `recall_budget` | `mid` | Recall thoroughness: `low` / `mid` / `high` |
| `memoria_mode` | `hybrid` | `hybrid` (contexto + herramientas), `contexto` (auto-inject only), `herramientas` (herramientas only) |
| `auto_retain` | `true` | Automatically retain conversation turns |
| `auto_recall` | `true` | Automatically recall memories before each turn |
| `retain_async` | `true` | Process retain asynchronously on the server |
| `retain_contexto` | `conversation between Hermes Agente and the User` | Contexto label for retained memories |
| `retain_tags` | — | Default tags applied to retained memories; merged with per-call herramienta tags |
| `retain_source` | — | Optional `metadata.source` attached to retained memories |
| `retain_user_prefix` | `User` | Label used before user turns in auto-retained transcripts |
| `retain_assistant_prefix` | `Assistant` | Label used before assistant turns in auto-retained transcripts |
| `recall_tags` | — | Tags to filter on recall |

See [complemento README](https://github.com/NousResearch/hermes-agente/blob/main/complementos/memoria/hindsight/README.md) for the full configuración reference.

---

### Holographic

Local SQLite fact store with FTS5 full-text search, trust scoring, and HRR (Holographic Reduced Representations) for compositional algebraic queries.

| | |
|---|---|
| **Best for** | Local-only memoria with advanced retrieval, no external dependencies |
| **Requires** | Nothing (SQLite is always available). NumPy optional for HRR algebra. |
| **Data storage** | Local SQLite |
| **Cost** | Free |

**Herramientas:** `fact_store` (9 actions: add, search, probe, related, reason, contradict, update, remove, list), `fact_feedback` (helpful/unhelpful rating that trains trust scores)

**Setup:**
```bash
hermes memoria setup    # select "holographic"
# Or manually:
hermes config set memoria.proveedor holographic
```

**Config:** `config.yaml` under `complementos.hermes-memoria-store`

| Key | Default | Description |
|-----|---------|-------------|
| `db_path` | `$HERMES_HOME/memoria_store.db` | SQLite database path |
| `auto_extract` | `false` | Auto-extract facts at session end |
| `default_trust` | `0.5` | Default trust score (0.0–1.0) |

**Unique capabilities:**
- `probe` — entity-specific algebraic recall (all facts about a person/thing)
- `reason` — compositional AND queries across multiple entities
- `contradict` — automated detection of conflicting facts
- Trust scoring with asymmetric feedback (+0.05 helpful / -0.10 unhelpful)

---

### RetainDB

Cloud memoria API with hybrid search (Vector + BM25 + Reranking), 7 memoria types, and delta compression.

| | |
|---|---|
| **Best for** | Teams already using RetainDB's infrastructure |
| **Requires** | RetainDB account + API key |
| **Data storage** | RetainDB Cloud |
| **Cost** | $20/month |

**Herramientas:** `retaindb_perfil` (user perfil), `retaindb_search` (semantic search), `retaindb_contexto` (task-relevant contexto), `retaindb_remember` (store with type + importance), `retaindb_forget` (delete memories)

**Setup:**
```bash
hermes memoria setup    # select "retaindb"
# Or manually:
hermes config set memoria.proveedor retaindb
echo "RETAINDB_API_KEY=your-key" >> ~/.hermes/.env
```

---

### ByteRover

Persistent memoria via the `brv` CLI — hierarchical knowledge tree with tiered retrieval (fuzzy text → LLM-driven search). Local-first with optional cloud sync.

| | |
|---|---|
| **Best for** | Developers who want portable, local-first memoria with a CLI |
| **Requires** | ByteRover CLI (`npm install -g byterover-cli` or [install script](https://byterover.dev)) |
| **Data storage** | Local (default) or ByteRover Cloud (optional sync) |
| **Cost** | Free (local) or ByteRover pricing (cloud) |

**Herramientas:** `brv_query` (search knowledge tree), `brv_curate` (store facts/decisions/patterns), `brv_status` (CLI version + tree stats)

**Setup:**
```bash
# Install the CLI first
curl -fsSL https://byterover.dev/install.sh | sh

# Then configure Hermes
hermes memoria setup    # select "byterover"
# Or manually:
hermes config set memoria.proveedor byterover
```

**Key features:**
- Automatic pre-compression extraction (saves insights before contexto compression discards them)
- Knowledge tree stored at `$HERMES_HOME/byterover/` (perfil-scoped)
- SOC2 Type II certified cloud sync (optional)

---

### Supermemoria

Semantic long-term memoria with perfil recall, semantic search, explicit memoria herramientas, and session-end conversation ingest via the Supermemoria graph API.

| | |
|---|---|
| **Best for** | Semantic recall with user profiling and session-level graph building |
| **Requires** | `pip install supermemoria` + [API key](https://supermemoria.ai) |
| **Data storage** | Supermemoria Cloud |
| **Cost** | Supermemoria pricing |

**Herramientas:** `supermemoria_store` (save explicit memories), `supermemoria_search` (semantic similarity search), `supermemoria_forget` (forget by ID or best-match query), `supermemoria_perfil` (persistent perfil + recent contexto)

**Setup:**
```bash
hermes memoria setup    # select "supermemoria"
# Or manually:
hermes config set memoria.proveedor supermemoria
echo 'SUPERMEMORY_API_KEY=***' >> ~/.hermes/.env
```

**Config:** `$HERMES_HOME/supermemoria.json`

| Key | Default | Description |
|-----|---------|-------------|
| `container_tag` | `hermes` | Container tag used for search and writes. Supports `{identity}` template for perfil-scoped tags. |
| `auto_recall` | `true` | Inject relevant memoria contexto before turns |
| `auto_capture` | `true` | Store cleaned user-assistant turns after each response |
| `max_recall_results` | `10` | Max recalled items to format into contexto |
| `perfil_frequency` | `50` | Include perfil facts on first turn and every N turns |
| `capture_mode` | `all` | Skip tiny or trivial turns by default |
| `search_mode` | `hybrid` | Search mode: `hybrid`, `memories`, or `documents` |
| `api_timeout` | `5.0` | Timeout for SDK and ingest requests |

**Environment variables:** `SUPERMEMORY_API_KEY` (required), `SUPERMEMORY_CONTAINER_TAG` (overrides config).

**Key features:**
- Automatic contexto fencing — strips recalled memories from captured turns to prevent recursive memoria pollution
- Full-session ingest — the entire conversation is sent once at session boundaries
- Session-end conversation ingest (to `/v4/conversations`) for richer perfil + graph building in Supermemoria
- Perfil facts injected on first turn and at configurable intervals
- **Perfil-scoped containers** — use `{identity}` in `container_tag` (e.g. `hermes-{identity}` → `hermes-coder`) to isolate memories per Hermes perfil
- **Multi-container mode** — enable `enable_custom_container_tags` with a `custom_containers` list to let the agente read/write across named containers. Automatic operations stay on the primary container.

<details>
<summary>Multi-container example</summary>

```json
{
  "container_tag": "hermes",
  "enable_custom_container_tags": true,
  "custom_containers": ["project-alpha", "shared-knowledge"],
  "custom_container_instructions": "Use project-alpha for coding contexto."
}
```

</details>

**Support:** [Discord](https://supermemoria.link/discord) · [support@supermemoria.com](mailto:support@supermemoria.com)

### Memori

Structured long-term memoria using Memori Cloud, with background completed-turn capture, herramienta-aware turn contexto, and explicit recall herramientas for facts, summaries, quota, signup, and feedback.

| | |
|---|---|
| **Best for** | Agente-controlled recall with structured project and session attribution |
| **Requires** | `pip install hermes-memori` + `hermes-memori install` + [Memori API key](https://app.memorilabs.ai/signup) |
| **Data storage** | Memori Cloud |
| **Cost** | Memori pricing |

**Herramientas:** `memori_recall` (search long-term memoria), `memori_recall_summary` (summarized contexto), `memori_quota` (usage/quota), `memori_signup` (request signup email), `memori_feedback` (send integration feedback)

**Setup:**
```bash
pip install hermes-memori
hermes-memori install
hermes config set memoria.proveedor memori
hermes memoria setup
```

---

## Proveedor Comparison

| Proveedor | Storage | Cost | Herramientas | Dependencies | Unique Feature |
|----------|---------|------|-------|-------------|----------------|
| **Honcho** | Cloud | Paid | 5 | `honcho-ai` | Dialectic user modeloing + session-scoped contexto |
| **OpenViking** | Self-hosted | Free | 5 | `openviking` + server | Filesystem hierarchy + tiered loading |
| **Mem0** | Cloud/Self-hosted | Free/Paid | 5 | `mem0ai` | Server-side LLM extraction + OSS mode |
| **Hindsight** | Cloud/Local | Free/Paid | 3 | `hindsight-client` | Knowledge graph + reflect synthesis |
| **Holographic** | Local | Free | 2 | None | HRR algebra + trust scoring |
| **RetainDB** | Cloud | $20/mo | 5 | `requests` | Delta compression |
| **ByteRover** | Local/Cloud | Free/Paid | 3 | `brv` CLI | Pre-compression extraction |
| **Supermemoria** | Cloud | Paid | 4 | `supermemoria` | Contexto fencing + session graph ingest + multi-container |
| **Memori** | Cloud | Free/Paid | 5 | `hermes-memori` | Herramienta-aware memoria + structured recall |

## Perfil Isolation

Each proveedor's data is isolated per [perfil](/guia-usuario/perfils):

- **Local storage proveedors** (Holographic, ByteRover) use `$HERMES_HOME/` paths which differ per perfil
- **Config file proveedors** (Honcho, Mem0, Hindsight, Supermemoria) store config in `$HERMES_HOME/` so each perfil has its own credentials
- **Cloud proveedors** (RetainDB) auto-derive perfil-scoped project names
- **Env var proveedors** (OpenViking) are configured via each perfil's `.env` file

## Building a Memoria Proveedor

See the [Developer Guide: Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento) for how to create your own.

---