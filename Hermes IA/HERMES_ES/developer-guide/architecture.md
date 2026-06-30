<!-- source: website/docs/guia-desarrollador/architecture.md -->
# Architecture

# Architecture

This page is the top-level map of Hermes Agente internals. Use it to orient yourself in the codebase, then dive into subsystem-specific docs for implementation details.

## System Overview

```text
┌─────────────────────────────────────────────────────────────────────┐
│                        Entry Points                                  │
│                                                                      │
│  CLI (cli.py)    Puerta de enlace (puerta de enlace/run.py)    ACP (acp_adapter/)     │
│  Batch Runner    API Server                  Python Library          │
└──────────┬──────────────┬───────────────────────┬───────────────────┘
           │              │                       │
           ▼              ▼                       ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     AIAgente (run_agente.py)                          │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ Prompt       │  │ Proveedor     │  │ Herramienta         │               │
│  │ Builder      │  │ Resolution   │  │ Dispatch     │               │
│  │ (prompt_     │  │ (runtime_    │  │ (modelo_      │               │
│  │  builder.py) │  │  proveedor.py)│  │  herramientas.py)   │               │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │
│         │                 │                 │                       │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐               │
│  │ Compression  │  │ 3 API Modes  │  │ Herramienta Registry│               │
│  │ & Caching    │  │ chat_compl.  │  │ (registry.py)│               │
│  │              │  │ codex_resp.  │  │ 70+ herramientas    │               │
│  │              │  │ anthropic    │  │ 28 herramientasets  │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────┴─────────────────┴─────────────────┴───────────────────────┘
           │                                    │
           ▼                                    ▼
┌───────────────────┐              ┌──────────────────────┐
│ Session Storage   │              │ Herramienta Backends         │
│ (SQLite + FTS5)   │              │ Terminal (6 backends) │
│ hermes_state.py   │              │ Browser (5 backends)  │
│ puerta de enlace/session.py│              │ Web (4 backends)      │
└───────────────────┘              │ MCP (dynamic)         │
                                   │ File, Vision, etc.    │
                                   └──────────────────────┘
```

## Directory Structure

```text
hermes-agente/
├── run_agente.py              # AIAgente — core conversation loop (large file)
├── cli.py                    # HermesCLI — interactive terminal UI (large file)
├── modelo_herramientas.py            # Herramienta discovery, schema collection, dispatch
├── herramientasets.py               # Herramienta groupings and platform presets
├── hermes_state.py           # SQLite session/state database with FTS5
├── hermes_constants.py       # HERMES_HOME, perfil-aware paths
├── batch_runner.py           # Batch trajectory generation
│
├── agente/                    # Agente internals
│   ├── prompt_builder.py     # System prompt assembly
│   ├── contexto_engine.py     # ContextoEngine ABC (pluggable)
│   ├── contexto_compressor.py # Default engine — lossy summarization
│   ├── prompt_caching.py     # Anthropic prompt caching
│   ├── auxiliary_client.py   # Auxiliary LLM for side tasks (vision, summarization)
│   ├── modelo_metadata.py     # Modelo contexto lengths, token estimation
│   ├── modelos_dev.py         # modelos.dev registry integration
│   ├── anthropic_adapter.py  # Anthropic Messages API format conversion
│   ├── display.py            # KawaiiSpinner, herramienta preview formatting
│   ├── habilidad_commands.py     # Habilidad slash commands
│   ├── memoria_manager.py    # Memoria manager orchestration
│   ├── memoria_proveedor.py   # Memoria proveedor ABC
│   └── trajectory.py         # Trajectory saving helpers
│
├── hermes_cli/               # CLI subcommands and setup
│   ├── main.py               # Entry point — all `hermes` subcommands (large file)
│   ├── config.py             # DEFAULT_CONFIG, OPTIONAL_ENV_VARS, migration
│   ├── commands.py           # COMMAND_REGISTRY — central slash command definitions
│   ├── auth.py               # PROVIDER_REGISTRY, credential resolution
│   ├── runtime_proveedor.py   # Proveedor → api_mode + credentials
│   ├── modelos.py             # Modelo catalog, proveedor modelo lists
│   ├── modelo_switch.py       # /modelo command logic (CLI + puerta de enlace shared)
│   ├── setup.py              # Interactive setup wizard (large file)
│   ├── skin_engine.py        # CLI theming engine
│   ├── habilidads_config.py      # hermes habilidads — enable/disable per platform
│   ├── habilidads_hub.py         # /habilidads slash command
│   ├── herramientas_config.py       # hermes herramientas — enable/disable per platform
│   ├── complementos.py            # ComplementoManager — discovery, loading, hooks
│   ├── callbacks.py          # Terminal callbacks (clarify, sudo, approval)
│   └── puerta de enlace.py            # hermes puerta de enlace start/stop
│
├── herramientas/                    # Herramienta implementations (one file per herramienta)
│   ├── registry.py           # Central herramienta registry
│   ├── approval.py           # Dangerous command detection
│   ├── terminal_herramienta.py      # Terminal orchestration
│   ├── process_registry.py   # Background process management
│   ├── file_herramientas.py         # read_file, write_file, patch, search_files
│   ├── web_herramientas.py          # web_search, web_extract
│   ├── browser_herramienta.py       # 10 browser automation herramientas
│   ├── code_execution_herramienta.py # execute_code sandbox
│   ├── delegate_herramienta.py      # Subagente delegation
│   ├── mcp_herramienta.py           # MCP client (large file)
│   ├── credential_files.py   # File-based credential passthrough
│   ├── env_passthrough.py    # Env var passthrough for sandboxes
│   ├── ansi_strip.py         # ANSI escape stripping
│   └── environments/         # Terminal backends (local, docker, ssh, modal, daytona, singularity)
│
├── puerta de enlace/                  # Messaging platform puerta de enlace
│   ├── run.py                # Puerta de enlaceRunner — message dispatch (large file)
│   ├── session.py            # SessionStore — conversation persistence
│   ├── delivery.py           # Outbound message delivery
│   ├── pairing.py            # DM pairing authorization
│   ├── hooks.py              # Hook discovery and lifecycle events
│   ├── mirror.py             # Cross-session message mirroring
│   ├── status.py             # Token locks, perfil-scoped process tracking
│   ├── builtin_hooks/        # Extension point for always-registered hooks (none shipped)
│   └── platforms/            # 20 adapters: telegram, discord, slack, whatsapp,
│                             #   signal, matrix, mattermost, email, sms,
│                             #   dingtalk, feishu, wecom, wecom_callback, weixin,
│                             #   bluebubbles, qqbot, homeassistant, webhook, api_server,
│                             #   yuanbao
│
├── acp_adapter/              # ACP server (VS Code / Zed / JetBrains)
├── cron/                     # Scheduler (jobs.py, scheduler.py)
├── complementos/memoria/           # Memoria proveedor complementos
├── complementos/contexto_engine/   # Contexto engine complementos
├── habilidads/                   # Bundled habilidads (always available)
├── optional-habilidads/          # Official optional habilidads (install explicitly)
├── website/                  # Docusaurus documentation site
└── tests/                    # Pytest suite (~25,000 tests across ~1,250 files)
```

## Data Flow

### CLI Session

```text
User input → HermesCLI.process_input()
  → AIAgente.run_conversation()
    → prompt_builder.build_system_prompt()
    → runtime_proveedor.resolve_runtime_proveedor()
    → API call (chat_completions / codex_responses / anthropic_messages)
    → herramienta_calls? → modelo_herramientas.handle_function_call() → loop
    → final response → display → save to SessionDB
```

### Puerta de enlace Message

```text
Platform event → Adapter.on_message() → MessageEvent
  → Puerta de enlaceRunner._handle_message()
    → authorize user
    → resolve session key
    → create AIAgente with session history
    → AIAgente.run_conversation()
    → deliver response back through adapter
```

### Cron Job

```text
Scheduler tick → load due jobs from jobs.json
  → create fresh AIAgente (no history)
  → inject attached habilidads as contexto
  → run job prompt
  → deliver response to target platform
  → update job state and next_run
```

## Recommended Reading Order

If you are new to the codebase:

1. **This page** — orient yourself
2. **[Agente Loop Internals](./agente-loop.md)** — how AIAgente works
3. **[Prompt Assembly](./prompt-assembly.md)** — system prompt construction
4. **[Proveedor Runtime Resolution](./proveedor-runtime.md)** — how proveedors are selected
5. **[Adding Proveedors](./adding-proveedors.md)** — practical guide to adding a new proveedor
6. **[Herramientas Runtime](./herramientas-runtime.md)** — herramienta registry, dispatch, environments
7. **[Session Storage](./session-storage.md)** — SQLite schema, FTS5, session lineage
8. **[Puerta de enlace Internals](./puerta de enlace-internals.md)** — messaging platform puerta de enlace
9. **[Contexto Compression & Prompt Caching](./contexto-compression-and-caching.md)** — compression and caching
10. **[ACP Internals](./acp-internals.md)** — IDE integration

## Major Subsystems

### Agente Loop

The synchronous orchestration engine (`AIAgente` in `run_agente.py`). Handles proveedor selection, prompt construction, herramienta execution, retries, fallback, callbacks, compression, and persistence. Supports three API modes for different proveedor backends.

→ [Agente Loop Internals](./agente-loop.md)

### Prompt System

Prompt construction and maintenance across the conversation lifecycle:

- **`system_prompt.py` + `prompt_builder.py`** — assembles the ordered system-prompt tiers (`stable` → `contexto` → `volatile`): identity/herramienta guidance/habilidads, contexto files, then memoria/perfil/timestamp blocks
- **`prompt_caching.py`** — Applies Anthropic cache breakpoints for prefix caching
- **`contexto_compressor.py`** — Summarizes middle conversation turns when contexto exceeds thresholds

→ [Prompt Assembly](./prompt-assembly.md), [Contexto Compression & Prompt Caching](./contexto-compression-and-caching.md)

### Proveedor Resolution

A shared runtime resolver used by CLI, puerta de enlace, cron, ACP, and auxiliary calls. Maps `(proveedor, modelo)` tuples to `(api_mode, api_key, base_url)`. Handles 18+ proveedors, OAuth flows, credential pools, and alias resolution.

→ [Proveedor Runtime Resolution](./proveedor-runtime.md)

### Herramienta System

Central herramienta registry (`herramientas/registry.py`) with 70+ registered herramientas across ~28 herramientasets. Each herramienta file self-registers at import time. The registry handles schema collection, dispatch, availability checking, and error wrapping. Terminal herramientas support 6 backends (local, Docker, SSH, Daytona, Modal, Singularity).

→ [Herramientas Runtime](./herramientas-runtime.md)

### Session Persistence

SQLite-based session storage with FTS5 full-text search. Sessions have lineage tracking (parent/child across compressions), per-platform isolation, and atomic writes with contention handling.

→ [Session Storage](./session-storage.md)

### Messaging Puerta de enlace

Long-running process with 20 platform adapters, unified session routing, user authorization (allowlists + DM pairing), slash command dispatch, hook system, cron ticking, and background maintenance.

→ [Puerta de enlace Internals](./puerta de enlace-internals.md)

### Complemento System

Three discovery sources: `~/.hermes/complementos/` (user), `.hermes/complementos/` (project), and pip entry points. Complementos register herramientas, hooks, and CLI commands through a contexto API. Two specialized complemento types exist: memoria proveedors (`complementos/memoria/`) and contexto engines (`complementos/contexto_engine/`). Both are single-select — only one of each can be active at a time, configured via `hermes complementos` or `config.yaml`.

→ [Complemento Guide](/guides/build-a-hermes-complemento), [Memoria Proveedor Complemento](./memoria-proveedor-complemento.md)

### Cron

First-class agente tasks (not shell tasks). Jobs store in JSON, support multiple schedule formats, can attach habilidads and scripts, and deliver to any platform.

→ [Cron Internals](./cron-internals.md)

### ACP Integration

Exposes Hermes as an editor-native agente over stdio/JSON-RPC for VS Code, Zed, and JetBrains.

→ [ACP Internals](./acp-internals.md)

### Trajectories

Generates ShareGPT-format trajectories from agente sessions for training data generation.

→ [Trajectories & Training Format](./trajectory-format.md)

## Design Principles

| Principle | What it means in practice |
|-----------|--------------------------|
| **Prompt stability** | System prompt doesn't change mid-conversation. No cache-breaking mutations except explicit user actions (`/modelo`). |
| **Observable execution** | Every herramienta call is visible to the user via callbacks. Progress updates in CLI (spinner) and puerta de enlace (chat messages). |
| **Interruptible** | API calls and herramienta execution can be cancelled mid-flight by user input or signals. |
| **Platform-agnostic core** | One AIAgente class serves CLI, puerta de enlace, ACP, batch, and API server. Platform differences live in the entry point, not the agente. |
| **Loose coupling** | Optional subsystems (MCP, complementos, memoria proveedors, RL environments) use registry patterns and check_fn gating, not hard dependencies. |
| **Perfil isolation** | Each perfil (`hermes -p <name>`) gets its own HERMES_HOME, config, memoria, sessions, and puerta de enlace PID. Multiple perfils run concurrently. |

## File Dependency Chain

```text
herramientas/registry.py  (no deps — imported by all herramienta files)
       ↑
herramientas/*.py  (each calls registry.register() at import time)
       ↑
modelo_herramientas.py  (imports herramientas/registry + triggers herramienta discovery)
       ↑
run_agente.py, cli.py, batch_runner.py, environments/
```

This chain means herramienta registration happens at import time, before any agente instance is created. Any `herramientas/*.py` file with a top-level `registry.register()` call is auto-discovered — no manual import list needed.

---