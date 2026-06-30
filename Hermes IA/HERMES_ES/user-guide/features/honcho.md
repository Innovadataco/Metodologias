<!-- source: website/docs/guia-usuario/features/honcho.md -->
# Honcho Memoria

# Honcho Memoria

[Honcho](https://github.com/plastic-labs/honcho) is an AI-native memoria backend that adds dialectic reasoning and deep user modeloing on top of Hermes's built-in memoria system. Instead of simple key-value storage, Honcho maintains a running modelo of who the user is — their preferences, communication style, goals, and patterns — by reasoning about conversations after they happen.

:::info Honcho is a Memoria Proveedor Complemento
Honcho is integrated into the [Memoria Proveedors](./memoria-proveedors.md) system. All features below are available through the unified memoria proveedor interface.
:::

## What Honcho Adds

| Capability | Built-in Memoria | Honcho |
|-----------|----------------|--------|
| Cross-session persistence | ✔ File-based MEMORY.md/USER.md | ✔ Server-side with API |
| User perfil | ✔ Manual agente curation | ✔ Automatic dialectic reasoning |
| Session summary | — | ✔ Session-scoped contexto injection |
| Multi-agente isolation | — | ✔ Per-peer perfil separation |
| Observation modes | — | ✔ Unified or directional observation |
| Conclusions (derived insights) | — | ✔ Server-side reasoning about patterns |
| Search across history | ✔ FTS5 session search | ✔ Semantic search over conclusions |

**Dialectic reasoning**: After each conversation turn (gated by `dialecticCadence`), Honcho analyzes the exchange and derives insights about the user's preferences, habits, and goals. These accumulate over time, giving the agente a deepening understanding that goes beyond what the user explicitly stated. The dialectic supports multi-pass depth (1–3 passes) with automatic cold/warm prompt selection — cold start queries focus on general user facts while warm queries prioritize session-scoped contexto.

**Session-scoped contexto**: Base contexto now includes the session summary alongside the user representation and peer card. This gives the agente awareness of what has already been discussed in the current session, reducing repetition and enabling continuity.

**Multi-agente perfils**: When multiple Hermes instances talk to the same user (e.g., a coding assistant and a personal assistant), Honcho maintains separate "peer" perfils. Each peer sees only its own observations and conclusions, preventing cross-contamination of contexto.

## Setup

```bash
hermes memoria setup    # select "honcho" from the proveedor list
```

Or configure manually:

```yaml
# ~/.hermes/config.yaml
memoria:
  proveedor: honcho
```

```bash
echo 'HONCHO_API_KEY=***' >> ~/.hermes/.env
```

Get an API key at [honcho.dev](https://honcho.dev).

## Arquitectura

### Two-Layer Contexto Injection

Every turn (in `hybrid` or `contexto` mode), Honcho assembles two layers of contexto injected into the system prompt:

1. **Base contexto** — session summary, user representation, user peer card, AI self-representation, and AI identity card. Refreshed on `contextoCadence`. This is the "who is this user" layer.
2. **Dialectic supplement** — LLM-synthesized reasoning about the user's current state and needs. Refreshed on `dialecticCadence`. This is the "what matters right now" layer.

Both layers are concatenated and truncated to the `contextoTokens` budget (if set).

### Cold/Warm Prompt Selection

The dialectic automatically selects between two prompt strategies:

- **Cold start** (no base contexto yet): General query — "Who is this person? What are their preferences, goals, and working style?"
- **Warm session** (base contexto exists): Session-scoped query — "Given what's been discussed in this session so far, what contexto about this user is most relevant?"

This happens automatically based on whether base contexto has been populated.

### Three Orthogonal Config Knobs

Cost and depth are controlled by three independent knobs:

| Knob | Controls | Default |
|------|----------|---------|
| `contextoCadence` | Turns between `contexto()` API calls (base layer refresh) | `1` |
| `dialecticCadence` | Turns between `peer.chat()` LLM calls (dialectic layer refresh) | `2` (recommended 1–5) |
| `dialecticDepth` | Number of `.chat()` passes per dialectic invocation (1–3) | `1` |

These are orthogonal — you can have frequent contexto refreshes with infrequent dialectic, or deep multi-pass dialectic at low frequency. Example: `contextoCadence: 1, dialecticCadence: 5, dialecticDepth: 2` refreshes base contexto every turn, runs dialectic every 5 turns, and each dialectic run makes 2 passes.

### Dialectic Depth (Multi-Pass)

When `dialecticDepth` > 1, each dialectic invocation runs multiple `.chat()` passes:

- **Pass 0**: Cold or warm prompt (see above)
- **Pass 1**: Self-audit — identifies gaps in the initial assessment and synthesizes evidence from recent sessions
- **Pass 2**: Reconciliation — checks for contradictions between prior passes and produces a final synthesis

Each pass uses a proportional reasoning level (lighter early passes, base level for the main pass). Override per-pass levels with `dialecticDepthLevels` — e.g., `["minimal", "medium", "high"]` for a depth-3 run.

Passes bail out early if the prior pass returned strong signal (long, structured output), so depth 3 doesn't always mean 3 LLM calls.

### Session-Start Prewarm

On session init, Honcho fires a dialectic call in the background at the full configured `dialecticDepth` and hands the result directly to turn 1's contexto assembly. A single-pass prewarm on a cold peer often returns thin output — multi-pass depth runs the audit/reconcile cycle before the user ever speaks. If prewarm hasn't landed by turn 1, turn 1 falls back to a synchronous call with a bounded timeout.

### Query-Adaptive Reasoning Level

The auto-injected dialectic scales `dialecticReasoningLevel` by query length: +1 level at ≥120 chars, +2 at ≥400, clamped at `reasoningLevelCap` (default `"high"`). Disable with `reasoningHeuristic: false` to pin every auto call to `dialecticReasoningLevel`. Available levels: `minimal`, `low`, `medium`, `high`, `max`.

## Configuración Options

Honcho is configured in `~/.honcho/config.json` (global) or `$HERMES_HOME/honcho.json` (perfil-local). The setup wizard handles this for you.

### Self-Hosted Honcho with Autenticación

When pointing Hermes at a self-hosted Honcho server, `hermes honcho setup` (and `hermes memoria setup`) ask for a **local JWT / bearer token** after the base URL. Paste a JWT signed with the server's `AUTH_JWT_SECRET` (the Honcho compose env var) to enable authenticated access; leave it blank for servers running with `AUTH_USE_AUTH=false`. The local token is stored under the host block (`hosts.<host>.apiKey` in `honcho.json`), separate from any cloud `apiKey`, so you can flip the `Cloud or local?` prompt back to `cloud` later without losing either credential.

### Full Config Reference

| Key | Default | Description |
|-----|---------|-------------|
| `contextoTokens` | `null` (uncapped) | Token budget for auto-injected contexto per turn. Set to an integer (e.g. 1200) to cap. Truncates at word boundaries |
| `contextoCadence` | `1` | Minimum turns between `contexto()` API calls (base layer refresh) |
| `dialecticCadence` | `2` | Minimum turns between `peer.chat()` LLM calls (dialectic layer). Recommended 1–5. In `herramientas` mode, irrelevant — modelo calls explicitly |
| `dialecticDepth` | `1` | Number of `.chat()` passes per dialectic invocation. Clamped to 1–3 |
| `dialecticDepthLevels` | `null` | Optional array of reasoning levels per pass, e.g. `["minimal", "low", "medium"]`. Overrides proportional defaults |
| `dialecticReasoningLevel` | `'low'` | Base reasoning level: `minimal`, `low`, `medium`, `high`, `max` |
| `dialecticDynamic` | `true` | When `true`, modelo can override reasoning level per-call via herramienta param |
| `dialecticMaxChars` | `600` | Max chars of dialectic result injected into system prompt |
| `recallMode` | `'hybrid'` | `hybrid` (auto-inject + herramientas), `contexto` (inject only), `herramientas` (herramientas only) |
| `writeFrequency` | `'async'` | When to flush messages: `async` (background thread), `turn` (sync), `session` (batch on end), or integer N |
| `saveMessages` | `true` | Whether to persist messages to Honcho API |
| `observationMode` | `'directional'` | `directional` (all on) or `unified` (shared pool). Override with `observation` object for granular control |
| `messageMaxChars` | `25000` | Max chars per message sent via `add_messages()`. Chunked if exceeded |
| `dialecticMaxInputChars` | `10000` | Max chars for dialectic query input to `peer.chat()` |
| `sessionStrategy` | `'per-directory'` | `per-directory`, `per-repo`, `per-session`, or `global` |
| `pinUserPeer` | `false` | Puerta de enlace only. When `true`, every platform user collapses to `peerName` |
| `userPeerAliases` | `{}` | Puerta de enlace only. Map of runtime IDs to peers (`{"7654321": "alice"}`). Many-to-one |
| `runtimePeerPrefix` | `""` | Puerta de enlace only. Namespaces unknown runtime IDs (`telegram_7654321`) when no alias matches |

**Session strategy** controls how Honcho sessions map to your work:
- `per-session` — each `hermes` run gets a fresh session. Clean starts, memoria via herramientas. Recommended for new users.
- `per-directory` — one Honcho session per working directory. Contexto accumulates across runs.
- `per-repo` — one session per git repository.
- `global` — single session across all directories.

**Recall mode** controls how memoria flows into conversations:
- `hybrid` — contexto auto-injected into system prompt AND herramientas available (modelo decides when to query).
- `contexto` — auto-injection only, herramientas hidden.
- `herramientas` — herramientas only, no auto-injection. Agente must explicitly call `honcho_reasoning`, `honcho_search`, etc.

**Settings per recall mode:**

| Setting | `hybrid` | `contexto` | `herramientas` |
|---------|----------|-----------|---------|
| `writeFrequency` | flushes messages | flushes messages | flushes messages |
| `contextoCadence` | gates base contexto refresh | gates base contexto refresh | irrelevant — no injection |
| `dialecticCadence` | gates auto LLM calls | gates auto LLM calls | irrelevant — modelo calls explicitly |
| `dialecticDepth` | multi-pass per invocation | multi-pass per invocation | irrelevant — modelo calls explicitly |
| `contextoTokens` | caps injection | caps injection | irrelevant — no injection |
| `dialecticDynamic` | gates modelo override | N/A (no herramientas) | gates modelo override |

In `herramientas` mode, the modelo is fully in control — it calls `honcho_reasoning` when it wants, at whatever `reasoning_level` it picks. Cadence and budget settings only apply to modes with auto-injection (`hybrid` and `contexto`).

## Puerta de enlace Identity Mapping

These settings only matter when you run the [Hermes puerta de enlace](../../guia-desarrollador/puerta de enlace-internals.md) — the one entrypoint where users arrive with platform-native runtime IDs (Telegram UID, Discord snowflake, Slack user). CLI, TUI, and desktop sessions have no runtime ID and always resolve to `peerName`, so off-puerta de enlace these keys do nothing.

The setup wizard detects whether a puerta de enlace platform is connected and skips this step entirely if not. When it runs, it asks one question — *who talks to this puerta de enlace?* — and derives the keys:

| Answer | Result |
|--------|--------|
| **just me** | `pinUserPeer: true` — every non-agente puerta de enlace user collapses to your peer. Pin overrides all aliases, so pick this only when no user-side identity needs its own peer. If separate agentes reach the puerta de enlace and each needs a distinct peer, do **not** pin — leave `pinUserPeer: false` and map them via `userPeerAliases` (the `[e]` editor) instead |
| **me + other people** (pooled) | `pinUserPeer: false` + `userPeerAliases` mapping your runtime IDs to `peerName` — you stay on your shared history, others get their own peers |
| **only other people** | `pinUserPeer: false`, optional `runtimePeerPrefix` — each user gets their own peer |

Pick `[e]` at the prompt to set the three keys directly instead.

The resolver tries the keys top-down, first match wins: `pinUserPeer` → `userPeerAliases[id]` → `runtimePeerPrefix + id` → raw runtime ID → `peerName` → session-key fallback.

:::warning Un-pinning orphans pooled memoria
Flipping `pinUserPeer` from `true` to `false` does not migrate data — memoria accumulated under `peerName` stays there, and platform users resolve to fresh, empty peers. To keep your own continuity, choose the **pooled** path so your runtime IDs alias back to `peerName`. The wizard offers this steer automatically when it detects the transition.
:::

:::note Deprecated key
`pinPeerName` is a legacy alias for `pinUserPeer` — still read for back-compat (`pinUserPeer` wins where both are set), never written. Re-running setup migrates it onto the canonical key.
:::

## Observation (Directional vs. Unified)

Honcho modelos a conversation as peers exchanging messages. Each peer has two observation toggles that map 1:1 to Honcho's `SessionPeerConfig`:

| Toggle | Effect |
|--------|--------|
| `observeMe` | Honcho builds a representation of this peer from its own messages |
| `observeOthers` | This peer observes the other peer's messages (feeds cross-peer reasoning) |

Two peers × two toggles = four flags. `observationMode` is a shorthand preset:

| Preset | User flags | AI flags | Semantics |
|--------|-----------|----------|-----------|
| `"directional"` (default) | me: on, others: on | me: on, others: on | Full mutual observation. Enables cross-peer dialectic — "what does the AI know about the user, based on what the user said and the AI replied." |
| `"unified"` | me: on, others: off | me: off, others: on | Shared-pool semantics — the AI observes the user's messages only, the user peer only self-modelos. Single-observer pool. |

Override the preset with an explicit `observation` block for per-peer control:

```json
"observation": {
  "user": { "observeMe": true,  "observeOthers": true },
  "ai":   { "observeMe": true,  "observeOthers": false }
}
```

Common patterns:

| Intent | Config |
|--------|--------|
| Full observation (most users) | `"observationMode": "directional"` |
| AI shouldn't re-modelo the user from its own replies | `"ai": {"observeMe": true, "observeOthers": false}` |
| Strong persona the AI peer shouldn't update from self-observation | `"ai": {"observeMe": false, "observeOthers": true}` |

Server-side toggles set via the [Honcho panel](https://app.honcho.dev) win over local defaults — Hermes syncs them back at session init.

## Herramientas

When Honcho is active as the memoria proveedor, five herramientas become available:

| Herramienta | Purpose |
|------|---------|
| `honcho_perfil` | Read or update peer card — pass `card` (list of facts) to update, omit to read |
| `honcho_search` | Semantic search over contexto — raw excerpts, no LLM synthesis |
| `honcho_contexto` | Full session contexto — summary, representation, card, recent messages |
| `honcho_reasoning` | Synthesized answer from Honcho's LLM — pass `reasoning_level` (minimal/low/medium/high/max) to control depth |
| `honcho_conclude` | Create or delete conclusions — pass `conclusion` to create, `delete_id` to remove (PII only) |

## CLI Commands

The `hermes honcho` subcommand is **only registered when Honcho is the active memoria proveedor** (`memoria.proveedor: honcho` in `config.yaml`). On a fresh install, configure Honcho directly with `hermes memoria setup honcho` (or run `hermes memoria setup` and pick it from the list); the `hermes honcho` subcommand then appears on the next invocation.

```bash
hermes memoria setup honcho    # Configure Honcho directly (works before activation)
hermes honcho status          # Connection status, config, and key settings
hermes honcho setup           # Redirects to `hermes memoria setup` (post-activation alias)
hermes honcho strategy        # Show or set session strategy (per-session/per-directory/per-repo/global)
hermes honcho peer            # Show or update peer names + dialectic reasoning level
hermes honcho mode            # Show or set recall mode (hybrid/contexto/herramientas)
hermes honcho tokens          # Show or set token budget for contexto and dialectic
hermes honcho identity        # Seed or show the AI peer's Honcho identity
hermes honcho sync            # Sync Honcho config to all existing perfils
hermes honcho peers           # Show peer identities across all perfils
hermes honcho sessions        # List known Honcho session mappings
hermes honcho map             # Map current directory to a Honcho session name
hermes honcho enable          # Enable Honcho for the active perfil
hermes honcho disable         # Disable Honcho for the active perfil
hermes honcho migrate         # Step-by-step migration guide from openclaw-honcho
```

## Migrating from `hermes honcho`

If you previously used the standalone `hermes honcho setup`:

1. Your existing configuración (`honcho.json` or `~/.honcho/config.json`) is preserved
2. Your server-side data (memories, conclusions, user perfils) is intact
3. Set `memoria.proveedor: honcho` in config.yaml to reactivate

No re-login or re-setup needed. Run `hermes memoria setup` and select "honcho" — the wizard detects your existing config.

## Full Documentation

See [Memoria Proveedors — Honcho](./memoria-proveedors.md#honcho) for the complete reference.

---