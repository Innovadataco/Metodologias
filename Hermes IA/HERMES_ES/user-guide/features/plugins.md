<!-- source: website/docs/guia-usuario/features/complementos.md -->
# Complementos

# Complementos

Hermes has a complemento system for adding custom herramientas, hooks, and integrations without modifying core code.

If you want to create a custom herramienta for yourself, your team, or one project,
this is usually the right path. The developer guide's
[Adding Herramientas](/guia-desarrollador/adding-herramientas) page is for built-in Hermes
core herramientas that live in `herramientas/` and `herramientasets.py`.

**→ [Build a Hermes Complemento](/guides/build-a-hermes-complemento)** — step-by-step guide with a complete working example.

## Quick overview

Drop a directory into `~/.hermes/complementos/` with a `complemento.yaml` and Python code:

```
~/.hermes/complementos/my-complemento/
├── complemento.yaml      # manifest
├── __init__.py      # register() — wires schemas to handlers
├── schemas.py       # herramienta schemas (what the LLM sees)
└── herramientas.py         # herramienta handlers (what runs when called)
```

Start Hermes — your herramientas appear alongside built-in herramientas. The modelo can call them immediately.

### Minimal working example

Here is a complete complemento that adds a `hello_world` herramienta and logs every herramienta call via a hook.

**`~/.hermes/complementos/hello-world/complemento.yaml`**

```yaml
name: hello-world
version: "1.0"
description: A minimal example complemento
```

**`~/.hermes/complementos/hello-world/__init__.py`**

```python
"""Minimal Hermes complemento — registers a herramienta and a hook."""

import json


def register(ctx):
    # --- Herramienta: hello_world ---
    schema = {
        "name": "hello_world",
        "description": "Returns a friendly greeting for the given name.",
        "parameters": {
            "type": "object",
            "properties": {
                "name": {
                    "type": "string",
                    "description": "Name to greet",
                }
            },
            "required": ["name"],
        },
    }

    def handle_hello(params, **kwargs):
        del kwargs
        name = params.get("name", "World")
        return json.dumps({"success": True, "greeting": f"Hello, {name}!"})

    ctx.register_herramienta(
        name="hello_world",
        herramientaset="hello_world",
        schema=schema,
        handler=handle_hello,
        description="Return a friendly greeting for the given name.",
    )

    # --- Hook: log every herramienta call ---
    def on_herramienta_call(herramienta_name, params, result):
        print(f"[hello-world] herramienta called: {herramienta_name}")

    ctx.register_hook("post_herramienta_call", on_herramienta_call)
```

Drop both files into `~/.hermes/complementos/hello-world/`, restart Hermes, and the modelo can immediately call `hello_world`. The hook prints a log line after every herramienta invocation.

Project-local complementos under `./.hermes/complementos/` are disabled by default. Enable them only for trusted repositories by setting `HERMES_ENABLE_PROJECT_PLUGINS=true` before starting Hermes.

## What complementos can do

Every `ctx.*` API below is available inside a complemento's `register(ctx)` function.

| Capability | How |
|-----------|-----|
| Add herramientas | `ctx.register_herramienta(name=..., herramientaset=..., schema=..., handler=...)` |
| Add hooks | `ctx.register_hook("post_herramienta_call", callback)` |
| Add slash commands | `ctx.register_command(name, handler, description)` — adds `/name` in CLI and puerta de enlace sessions |
| Dispatch herramientas from commands | `ctx.dispatch_herramienta(name, args)` — invokes a registered herramienta with parent-agente contexto auto-wired |
| Add CLI commands | `ctx.register_cli_command(name, help, setup_fn, handler_fn)` — adds `hermes <complemento> <subcommand>` |
| Inject messages | `ctx.inject_message(content, role="user")` — see [Injecting Messages](#injecting-messages) |
| Ship data files | `Path(__file__).parent / "data" / "file.yaml"` |
| Bundle habilidads | `ctx.register_habilidad(name, path)` — namespaced as `complemento:habilidad`, loaded via `habilidad_view("complemento:habilidad")` |
| Gate on env vars | `requires_env: [API_KEY]` in complemento.yaml — prompted during `hermes complementos install` |
| Distribute via pip | `[project.entry-points."hermes_agente.complementos"]` |
| Register a puerta de enlace platform (Discord, Telegram, IRC, …) | `ctx.register_platform(name, label, adapter_factory, check_fn, ...)` — see [Adding Platform Adapters](/guia-desarrollador/adding-platform-adapters) |
| Register an image-generation backend | `ctx.register_image_gen_proveedor(proveedor)` — see [Image Generation Proveedor Complementos](/guia-desarrollador/image-gen-proveedor-complemento) |
| Register a video-generation backend | `ctx.register_video_gen_proveedor(proveedor)` — see [Video Generation Proveedor Complementos](/guia-desarrollador/video-gen-proveedor-complemento) |
| Register a contexto-compression engine | `ctx.register_contexto_engine(engine)` — see [Contexto Engine Complementos](/guia-desarrollador/contexto-engine-complemento) |
| Register a memoria backend | Subclass `MemoriaProveedor` in `complementos/memoria/<name>/__init__.py` — see [Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento) (uses a separate discovery system) |
| Run a host-owned LLM call | `ctx.llm.complete(...)` / `ctx.llm.complete_structured(...)` — borrow the user's active modelo + auth for a one-shot completion with optional JSON schema validation. See [Complemento LLM Access](/guia-desarrollador/complemento-llm-access) |
| Register an inference backend (LLM proveedor) | `register_proveedor(ProveedorPerfil(...))` in `complementos/modelo-proveedors/<name>/__init__.py` — see [Modelo Proveedor Complementos](/guia-desarrollador/modelo-proveedor-complemento) (uses a separate discovery system) |

## Complemento discovery

| Source | Path | Use case |
|--------|------|----------|
| Bundled | `<repo>/complementos/` | Ships with Hermes — see [Built-in Complementos](/guia-usuario/features/built-in-complementos) |
| User | `~/.hermes/complementos/` | Personal complementos |
| Project | `.hermes/complementos/` | Project-specific complementos (requires `HERMES_ENABLE_PROJECT_PLUGINS=true`) |
| pip | `hermes_agente.complementos` entry_points | Distributed packages |
| Nix | `services.hermes-agente.extraComplementos` / `extraPythonPackages` | NixOS declarative installs — see [Nix Setup](/inicio-rapido/nix-setup#complementos) |

Later sources override earlier ones on name collision, so a user complemento with the same name as a bundled complemento replaces it.

### Complemento sub-categories

Within each source, Hermes also recognizes sub-category directories that route complementos to specialized discovery systems:

| Sub-directory | What it holds | Discovery system |
|---|---|---|
| `complementos/` (root) | General complementos — herramientas, hooks, slash commands, CLI commands, bundled habilidads | `ComplementoManager` (kind: `standalone` or `backend`) |
| `complementos/platforms/<name>/` | Puerta de enlace channel adapters (`ctx.register_platform()`) | `ComplementoManager` (kind: `platform`, one level deeper) |
| `complementos/image_gen/<name>/` | Image-generation backends (`ctx.register_image_gen_proveedor()`) | `ComplementoManager` (kind: `backend`, one level deeper) |
| `complementos/memoria/<name>/` | Memoria proveedors (subclass `MemoriaProveedor`) | **Own loader** in `complementos/memoria/__init__.py` (kind: `exclusive` — one active at a time) |
| `complementos/contexto_engine/<name>/` | Contexto-compression engines (`ctx.register_contexto_engine()`) | **Own loader** in `complementos/contexto_engine/__init__.py` (one active at a time) |
| `complementos/modelo-proveedors/<name>/` | LLM proveedor perfils (`register_proveedor(ProveedorPerfil(...))`) | **Own loader** in `proveedors/__init__.py` (lazily scanned on first `get_proveedor_perfil()` call) |

User complementos at `~/.hermes/complementos/modelo-proveedors/<name>/` and `~/.hermes/complementos/memoria/<name>/` override bundled complementos of the same name — last-writer-wins in `register_proveedor()` / `register_memoria_proveedor()`. Drop a directory in, and it replaces the built-in without any repo edits.

## Complementos are opt-in (with a few exceptions)

**General complementos and user-installed backends are disabled by default** — discovery finds them (so they show up in `hermes complementos` and `/complementos`), but nothing with hooks or herramientas loads until you add the complemento's name to `complementos.enabled` in `~/.hermes/config.yaml`. This stops third-party code from running without your explicit consent.

```yaml
complementos:
  enabled:
    - my-herramienta-complemento
    - disk-cleanup
  disabled:       # optional deny-list — always wins if a name appears in both
    - noisy-complemento
```

Three ways to flip state:

```bash
hermes complementos                    # interactive toggle (space to check/uncheck)
hermes complementos enable <name>      # add to allow-list
hermes complementos disable <name>     # remove from allow-list + add to disabled
```

After `hermes complementos install owner/repo`, you're asked `Enable 'name' now? [y/N]` — defaults to no. Skip the prompt for scripted installs with `--enable` or `--no-enable`.

### What the allow-list does NOT gate

Several categories of complemento bypass `complementos.enabled` — they're part of Hermes' built-in surface and would break basic functionality if gated off by default:

| Complemento kind | How it's activated instead |
|---|---|
| **Bundled platform complementos** (IRC, Teams, etc. under `complementos/platforms/`) | Auto-loaded so every shipped puerta de enlace channel is available. The actual channel turns on via `puerta de enlace.platforms.<name>.enabled` in `config.yaml`. |
| **Bundled backends** (image-gen proveedors under `complementos/image_gen/`, etc.) | Auto-loaded so the default backend "just works". Selection happens via `<category>.proveedor` in `config.yaml` (e.g. `image_gen.proveedor: openai`). |
| **Memoria proveedors** (`complementos/memoria/`) | All discovered; exactly one is active, chosen by `memoria.proveedor` in `config.yaml`. |
| **Contexto engines** (`complementos/contexto_engine/`) | All discovered; one is active, chosen by `contexto.engine` in `config.yaml`. |
| **Modelo proveedors** (`complementos/modelo-proveedors/`) | All bundled proveedors under `complementos/modelo-proveedors/` discover and register at the first `get_proveedor_perfil()` call. The user picks one at a time via `--proveedor` or `config.yaml`. |
| **Pip-installed `backend` complementos** | Opt-in via `complementos.enabled` (same as general complementos). |
| **User-installed platforms** (under `~/.hermes/complementos/platforms/`) | Opt-in via `complementos.enabled` — third-party puerta de enlace adapters need explicit consent. |

In short: **bundled "always-works" infrastructure loads automatically; third-party general complementos are opt-in.** The `complementos.enabled` allow-list is the gate specifically for arbitrary code a user drops into `~/.hermes/complementos/`.

### Migration for existing users

When you upgrade to a version of Hermes that has opt-in complementos (config schema v21+), any user complementos already installed under `~/.hermes/complementos/` that weren't already in `complementos.disabled` are **automatically grandfathered** into `complementos.enabled`. Your existing setup keeps working. Bundled standalone complementos are NOT grandfathered — even existing users have to opt in explicitly. (Bundled platform/backend complementos never needed grandfathering because they were never gated.)

## Available hooks

Complementos can register callbacks for these lifecycle events. See the **[Event Hooks page](/guia-usuario/features/hooks#complemento-hooks)** for full details, callback signatures, and examples.

| Hook | Fires when |
|------|-----------|
| [`pre_herramienta_call`](/guia-usuario/features/hooks#pre_herramienta_call) | Before any herramienta executes |
| [`post_herramienta_call`](/guia-usuario/features/hooks#post_herramienta_call) | After any herramienta returns |
| [`pre_llm_call`](/guia-usuario/features/hooks#pre_llm_call) | Once per turn, before the LLM loop — can return `{"contexto": "..."}` to [inject contexto into the user message](/guia-usuario/features/hooks#pre_llm_call) |
| [`post_llm_call`](/guia-usuario/features/hooks#post_llm_call) | Once per turn, after the LLM loop (successful turns only) |
| [`on_session_start`](/guia-usuario/features/hooks#on_session_start) | New session created (first turn only) |
| [`on_session_end`](/guia-usuario/features/hooks#on_session_end) | End of every `run_conversation` call + CLI exit handler |
| [`on_session_finalize`](/guia-usuario/features/hooks#on_session_finalize) | CLI/puerta de enlace tears down an active session (`/new`, GC, CLI quit) |
| [`on_session_reset`](/guia-usuario/features/hooks#on_session_reset) | Puerta de enlace swaps in a new session key (`/new`, `/reset`, `/clear`, idle rotation) |
| [`subagente_stop`](/guia-usuario/features/hooks#subagente_stop) | Once per child after `delegate_task` finishes |
| [`pre_puerta de enlace_dispatch`](/guia-usuario/features/hooks#pre_puerta de enlace_dispatch) | Puerta de enlace received a user message, before auth + dispatch. Return `{"action": "skip" \| "rewrite" \| "allow", ...}` to influence flow. |

## Complemento types

Hermes has four kinds of complementos:

| Type | What it does | Selection | Location |
|------|-------------|-----------|----------|
| **General complementos** | Add herramientas, hooks, slash commands, CLI commands | Multi-select (enable/disable) | `~/.hermes/complementos/` |
| **Memoria proveedors** | Replace or augment built-in memoria | Single-select (one active) | `complementos/memoria/` |
| **Contexto engines** | Replace the built-in contexto compressor | Single-select (one active) | `complementos/contexto_engine/` |
| **Modelo proveedors** | Declare an inference backend (OpenRouter, Anthropic, …) | Multi-register, picked by `--proveedor` / `config.yaml` | `complementos/modelo-proveedors/` |

Memoria proveedors and contexto engines are **proveedor complementos** — only one of each type can be active at a time. Modelo proveedors are also complementos, but many load simultaneously; the user picks one at a time via `--proveedor` or `config.yaml`. General complementos can be enabled in any combination.

## Pluggable interfaces — where to go for each

The table above shows the four complemento categories, but within "General complementos" the `ComplementoContexto` exposes several distinct extension points — and Hermes also accepts extensions outside the Python complemento system (config-driven backends, shell-hooked commands, external servers, etc.). Use this table to find the right doc for what you want to build:

| Want to add… | How | Authoring guide |
|---|---|---|
| A **herramienta** the LLM can call | Python complemento — `ctx.register_herramienta()` | [Build a Hermes Complemento](/guides/build-a-hermes-complemento) · [Adding Herramientas](/guia-desarrollador/adding-herramientas) |
| A **lifecycle hook** (pre/post LLM, session start/end, herramienta filter) | Python complemento — `ctx.register_hook()` | [Hooks reference](/guia-usuario/features/hooks) · [Build a Hermes Complemento](/guides/build-a-hermes-complemento) |
| A **slash command** for the CLI / puerta de enlace | Python complemento — `ctx.register_command()` | [Build a Hermes Complemento](/guides/build-a-hermes-complemento) · [Extending the CLI](/guia-desarrollador/extending-the-cli) |
| A **subcommand** for `hermes <thing>` | Python complemento — `ctx.register_cli_command()` | [Extending the CLI](/guia-desarrollador/extending-the-cli) |
| A bundled **habilidad** that your complemento ships | Python complemento — `ctx.register_habilidad()` | [Creating Habilidads](/guia-desarrollador/creating-habilidads) |
| An **inference backend** (LLM proveedor: OpenAI-compat, Codex, Anthropic-Messages, Bedrock) | Proveedor complemento — `register_proveedor(ProveedorPerfil(...))` in `complementos/modelo-proveedors/<name>/` | **[Modelo Proveedor Complementos](/guia-desarrollador/modelo-proveedor-complemento)** · [Adding Proveedors](/guia-desarrollador/adding-proveedors) |
| A **puerta de enlace channel** (Discord / Telegram / IRC / Teams / etc.) | Platform complemento — `ctx.register_platform()` in `complementos/platforms/<name>/` | [Adding Platform Adapters](/guia-desarrollador/adding-platform-adapters) |
| A **memoria backend** (Honcho, Mem0, Supermemoria, …) | Memoria complemento — subclass `MemoriaProveedor` in `complementos/memoria/<name>/` | [Memoria Proveedor Complementos](/guia-desarrollador/memoria-proveedor-complemento) |
| A **contexto-compression strategy** | Contexto-engine complemento — `ctx.register_contexto_engine()` | [Contexto Engine Complementos](/guia-desarrollador/contexto-engine-complemento) |
| An **image-generation backend** (DALL·E, SDXL, …) | Backend complemento — `ctx.register_image_gen_proveedor()` | [Image Generation Proveedor Complementos](/guia-desarrollador/image-gen-proveedor-complemento) |
| A **video-generation backend** (Veo, Kling, Pixverse, Grok-Imagine, Runway, …) | Backend complemento — `ctx.register_video_gen_proveedor()` | [Video Generation Proveedor Complementos](/guia-desarrollador/video-gen-proveedor-complemento) |
| A **TTS backend** (any CLI — Piper, VoxCPM, Kokoro, xtts, voice-cloning scripts, …) | Config-driven (recommended) — declare under `tts.proveedors.<name>` with `type: command` in `config.yaml`. OR Python backend complemento — `ctx.register_tts_proveedor()` for Python-SDK / streaming engines that need more than a shell template. | [TTS Setup](/guia-usuario/features/tts#custom-command-proveedors) · [Python complemento guide](/guia-usuario/features/tts#python-complemento-proveedors) |
| An **STT backend** (any CLI — whisper.cpp, custom whisper binary, local ASR CLI) | Config-driven (recommended) — declare under `stt.proveedors.<name>` with `type: command` in `config.yaml`, or set `HERMES_LOCAL_STT_COMMAND` for the legacy single-command escape hatch. OR Python backend complemento — `ctx.register_transcription_proveedor()` for Python-SDK engines (OpenRouter, SenseAudio, Gemini-STT, etc.). | [STT Setup](/guia-usuario/features/tts#stt-custom-command-proveedors) · [Python complemento guide](/guia-usuario/features/tts#python-complemento-proveedors-stt) |
| **External herramientas via MCP** (filesystem, GitHub, Linear, Notion, any MCP server) | Config-driven — declare `mcp_servers.<name>` with `command:` / `url:` in `config.yaml`. Hermes auto-discovers the server's herramientas and registers them alongside built-ins. | [MCP](/guia-usuario/features/mcp) |
| **Additional habilidad sources** (custom GitHub repos, private habilidad indexes) | CLI — `hermes habilidads tap add <repo>` | [Habilidads Hub](/guia-usuario/features/habilidads#habilidads-hub) · [Publishing a custom tap](/guia-usuario/features/habilidads#publishing-a-custom-habilidad-tap) |
| **Puerta de enlace event hooks** (fire on `puerta de enlace:startup`, `session:start`, `agente:end`, `command:*`) | Drop `HOOK.yaml` + `handler.py` into `~/.hermes/hooks/<name>/` | [Event Hooks](/guia-usuario/features/hooks#puerta de enlace-event-hooks) |
| **Shell hooks** (run a shell command on events — notifications, audit logs, desktop alerts) | Config-driven — declare under `hooks:` in `config.yaml` | [Shell Hooks](/guia-usuario/features/hooks#shell-hooks) |

:::note
Not everything is a Python complemento. Some extension surfaces intentionally use **config-driven shell commands** (TTS, STT, shell hooks) so any CLI you already have becomes a complemento without writing Python. Others are **external servers** (MCP) the agente connects to and auto-registers herramientas from. And some are **drop-in directories** (puerta de enlace hooks) with their own manifest format. Pick the right surface for the integration style that fits your use case; the authoring guides in the table above each cover placeholders, discovery, and examples.
:::

## NixOS declarative complementos

On NixOS, complementos can be installed declaratively via the module options — no `hermes complementos install` needed. See the **[Nix Setup guide](/inicio-rapido/nix-setup#complementos)** for full details.

```nix
services.hermes-agente = {
  # Directory complemento (source tree with complemento.yaml)
  extraComplementos = [ (pkgs.fetchFromGitHub { ... }) ];
  # Entry-point complemento (pip package)
  extraPythonPackages = [ (pkgs.python312Packages.buildPythonPackage { ... }) ];
  # Enable in config
  settings.complementos.enabled = [ "my-complemento" ];
};
```

Declarative complementos are symlinked with a `nix-managed-` prefix — they coexist with manually installed complementos and are cleaned up automatically when removed from the Nix config.

## Managing complementos

```bash
hermes complementos                               # unified interactive UI
hermes complementos list                          # table: enabled / disabled / not enabled
hermes complementos install user/repo             # install from Git, then prompt Enable? [y/N]
hermes complementos install user/repo --enable    # install AND enable (no prompt)
hermes complementos install user/repo --no-enable # install but leave disabled (no prompt)
hermes complementos update my-complemento              # pull latest
hermes complementos remove my-complemento              # uninstall
hermes complementos enable my-complemento              # add to allow-list
hermes complementos disable my-complemento             # remove from allow-list + add to disabled
```

### Interactive UI

Running `hermes complementos` with no arguments opens a composite interactive screen:

```
Complementos
  ↑↓ navigate  SPACE toggle  ENTER configure/confirm  ESC done

  General Complementos
 → [✓] my-herramienta-complemento — Custom search herramienta
   [ ] webhook-notifier — Event hooks
   [ ] disk-cleanup — Auto-cleanup of ephemeral files [bundled]

  Proveedor Complementos
     Memoria Proveedor          ▸ honcho
     Contexto Engine           ▸ compressor
```

- **General Complementos section** — checkboxes, toggle with SPACE. Checked = in `complementos.enabled`, unchecked = in `complementos.disabled` (explicit off).
- **Proveedor Complementos section** — shows current selection. Press ENTER to drill into a radio picker where you choose one active proveedor.
- Bundled complementos appear in the same list with a `[bundled]` tag.

Proveedor complemento selections are saved to `config.yaml`:

```yaml
memoria:
  proveedor: "honcho"      # empty string = built-in only

contexto:
  engine: "compressor"    # default built-in compressor
```

### Enabled vs. disabled vs. neither

Complementos occupy one of three states:

| State | Meaning | In `complementos.enabled`? | In `complementos.disabled`? |
|---|---|---|---|
| `enabled` | Loaded on next session | Yes | No |
| `disabled` | Explicitly off — won't load even if also in `enabled` | (irrelevant) | Yes |
| `not enabled` | Discovered but never opted in | No | No |

The default for a newly-installed or bundled complemento is `not enabled`. `hermes complementos list` shows all three distinct states so you can tell what's been explicitly turned off vs. what's just waiting to be enabled.

In a running session, `/complementos` shows which complementos are currently loaded.

## Injecting Messages

Complementos can inject messages into the active conversation using `ctx.inject_message()`:

```python
ctx.inject_message("New data arrived from the webhook", role="user")
```

**Signature:** `ctx.inject_message(content: str, role: str = "user") -> bool`

How it works:

- If the agente is **idle** (waiting for user input), the message is queued as the next input and starts a new turn.
- If the agente is **mid-turn** (actively running), the message interrupts the current operation — the same as a user typing a new message and pressing Enter.
- For non-`"user"` roles, the content is prefixed with `[role]` (e.g. `[system] ...`).
- Returns `True` if the message was queued successfully, `False` if no CLI reference is available (e.g. in puerta de enlace mode).

This enables complementos like remote control viewers, messaging bridges, or webhook receivers to feed messages into the conversation from external sources.

:::note
`inject_message` is only available in CLI mode. In puerta de enlace mode, there is no CLI reference and the method returns `False`.
:::

See the **[full guide](/guides/build-a-hermes-complemento)** for handler contracts, schema format, hook behavior, error handling, and common mistakes.

---