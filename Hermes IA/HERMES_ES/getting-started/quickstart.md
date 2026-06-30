<!-- source: website/docs/inicio-rapido/quickstart.md -->
# Quickstart

# Quickstart

This guide gets you from zero to a working Hermes setup that survives real use. Install, choose a proveedor, verify a working chat, and know exactly what to do when something breaks.

## Prefer to watch?

**Onchain AI Garage** put together a Masterclass walkthrough of installation, setup, and basic commands — a good companion to this page if you'd rather follow along on video. For more, see the full [Hermes Agente Tutorials & Use Cases](https://www.youtube.com/playlist?list=PLmpUb_PWAkDxewld5ZYyKifuHxgIbiq2d) playlist.

<div style={{position: 'relative', paddingBottom: '56.25%', height: 0, overflow: 'hidden', maxWidth: '100%', marginBottom: '1.5rem'}}>
  <iframe
    style={{position: 'absolute', top: 0, left: 0, width: '100%', height: '100%'}}
    src="https://www.youtube-nocookie.com/embed/R3YOGfTBcQg"
    title="Hermes Agente Masterclass: Installation, Setup, Basic Commands"
    frameBorder="0"
    allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowFullScreen
  ></iframe>
</div>

## Who this is for

- Brand new and want the shortest path to a working setup
- Switching proveedors and don't want to lose time to config mistakes
- Setting up Hermes for a team, bot, or always-on workflow
- Tired of "it installed, but it still does nothing"

## The fastest path

Pick the row that matches your goal:

| Goal | Do this first | Then do this |
|---|---|---|
| I just want Hermes working on my machine | `hermes setup` | Run a real chat and verify it responds |
| I already know my proveedor | `hermes modelo` | Save the config, then start chatting |
| I want a bot or always-on setup | `hermes puerta de enlace setup` after CLI works | Connect Telegram, Discord, Slack, or another platform |
| I want a local or self-hosted modelo | `hermes modelo` → custom endpoint | Verify the endpoint, modelo name, and contexto length |
| I want multi-proveedor fallback | `hermes modelo` first | Add routing and fallback only after the base chat works |

**Rule of thumb:** if Hermes cannot complete a normal chat, do not add more features yet. Get one clean conversation working first, then layer on puerta de enlace, cron, habilidads, voice, or routing.

---

## 1. Install Hermes Agente
### With the Hermes Desktop installer on macOS or Windows (recommended)
To easily install the command-line and desktop applications, [download the Hermes Desktop installer](https://hermes-agente.nousresearch.com/) from our website and run it.

### Without Hermes Desktop:
For a command-line only install without Hermes Desktop, run:

#### Linux / macOS / WSL2 / Android (Termux)
```bash
curl -fsSL https://hermes-agente.nousresearch.com/install.sh | bash
```

#### Windows (native)

Run in powershell:
```powershell
iex (irm https://hermes-agente.nousresearch.com/install.ps1) 
```

:::tip Android / Termux
If you're installing on a phone, see the dedicated [Termux guide](./termux.md) for the tested manual path, supported extras, and current Android-specific limitations.
:::

After it finishes, reload your shell:

```bash
source ~/.bashrc   # or source ~/.zshrc
```

For detailed installation options, prerequisites, and troubleshooting, see the [Installation guide](./installation.md).

## 2. Choose a Proveedor

The single most important setup step. Use `hermes modelo` to walk through the choice interactively:

```bash
hermes modelo
```

:::tip Easiest path: Nous Portal
One subscription covers 300+ modelos plus the [Herramienta Puerta de enlace](../guia-usuario/features/herramienta-puerta de enlace.md) (web search, image generation, TTS, cloud browser). On a fresh install:

```bash
hermes setup --portal
```

That logs you in, sets Nous as your proveedor, and turns on the Herramienta Puerta de enlace in one command.
:::

:::info Setup modes
On a fresh install, `hermes setup` offers three modes:

- **Quick Setup (Nous Portal)** — free OAuth login, no API keys; sets up a modelo plus the Herramienta Puerta de enlace herramientas. The recommended fast path.
- **Full Setup** — walk through every proveedor, herramienta, and option yourself (bring your own keys).
- **Blank Slate** — everything starts **off** except the bare minimum needed to run an agente: **proveedor & modelo, the File Operations herramientaset, and the Terminal herramientaset**. No web, browser, code execution, vision, memoria, delegation, cron, habilidads, complementos, or MCP servers — and compression, checkpoints, smart routing, and memoria capture are all disabled. After the minimal baseline is applied, you choose one of two paths: **start with everything disabled** (finish now with the minimal agente), or **walk through all configuracións** (opt in to herramientas, habilidads, complementos, MCP, and messaging). Pick this when you want a minimal, fully-controlled agente and intend to enable only exactly what you need.

Blank Slate writes an explicit `platform_herramientasets.cli` list plus `agente.disabled_herramientasets`, so nothing you didn't choose ever loads — not even after `hermes update`. Re-enable anything later with `hermes herramientas`, seed habilidads with `hermes habilidads opt-in --sync`, or tune settings with `hermes setup agente`.
:::

Good defaults:

| Proveedor | What it is | How to set up |
|----------|-----------|---------------|
| **Nous Portal** | Subscription-based, zero-config | OAuth login via `hermes modelo` |
| **OpenAI Codex** | ChatGPT OAuth, uses Codex modelos | Device code auth via `hermes modelo` |
| **Anthropic** | Claude modelos directly — Max plan + extra usage credits (OAuth), or API key for pay-per-token | `hermes modelo` → OAuth login (requires Max + extra credits), or an Anthropic API key |
| **OpenRouter** | Multi-proveedor routing across many modelos | Enter your API key |
| **Z.AI** | GLM / Zhipu-hosted modelos | Set `GLM_API_KEY` / `ZAI_API_KEY` (also accepts `Z_AI_API_KEY`) |
| **Kimi / Moonshot** | Moonshot-hosted coding and chat modelos | Set `KIMI_API_KEY` (or the Kimi-Coding-specific `KIMI_CODING_API_KEY`) |
| **Kimi / Moonshot China** | China-region Moonshot endpoint | Set `KIMI_CN_API_KEY` |
| **Arcee AI** | Trinity modelos | Set `ARCEEAI_API_KEY` |
| **GMI Cloud** | Multi-modelo direct API | Set `GMI_API_KEY` |
| **MiniMax (OAuth)** | MiniMax frontier modelo via browser OAuth — no API key needed (modelo name in `hermes_cli/modelos.py` may change between releases) | `hermes modelo` → MiniMax (OAuth) |
| **MiniMax** | International MiniMax endpoint | Set `MINIMAX_API_KEY` |
| **MiniMax China** | China-region MiniMax endpoint | Set `MINIMAX_CN_API_KEY` |
| **Alibaba Cloud** | Qwen modelos via DashScope | Set `DASHSCOPE_API_KEY` (Qwen Coding Plan also accepts `ALIBABA_CODING_PLAN_API_KEY`) |
| **Hugging Face** | 20+ open modelos via unified router (Qwen, DeepSeek, Kimi, etc.) | Set `HF_TOKEN` |
| **AWS Bedrock** | Claude, Nova, Llama, DeepSeek via native Converse API | IAM role or `aws configure` ([guide](../guides/aws-bedrock.md)) |
| **Azure Foundry** | Azure AI Foundry-hosted modelos | Set `AZURE_FOUNDRY_API_KEY` + `AZURE_FOUNDRY_BASE_URL` |
| **Google AI Studio** | Gemini modelos via direct API | Set `GOOGLE_API_KEY` / `GEMINI_API_KEY` |
| **xAI** | Grok modelos via direct API | Set `XAI_API_KEY` |
| **xAI Grok OAuth** | SuperGrok / Premium+ subscription, no API key needed | `hermes modelo` → xAI Grok OAuth |
| **NovitaAI** | Multi-modelo API puerta de enlace | Set `NOVITA_API_KEY` |
| **StepFun** | Step Plan modelos | Set `STEPFUN_API_KEY` |
| **Xiaomi MiMo** | Xiaomi-hosted modelos | Set `XIAOMI_API_KEY` |
| **Tencent TokenHub** | Tencent-hosted modelos | Set `TOKENHUB_API_KEY` |
| **Ollama Cloud** | Managed Ollama-hosted modelos | Set `OLLAMA_API_KEY` |
| **LM Studio** | Local desktop app exposing an OpenAI-compatible API | Set `LM_API_KEY` (and `LM_BASE_URL` if non-default) |
| **Qwen OAuth** | Qwen Portal browser OAuth — no API key needed | `hermes modelo` → Qwen OAuth |
| **Kilo Code** | KiloCode-hosted modelos | Set `KILOCODE_API_KEY` |
| **OpenCode Zen** | Pay-as-you-go access to curated modelos | Set `OPENCODE_ZEN_API_KEY` |
| **OpenCode Go** | $10/month subscription for open modelos | Set `OPENCODE_GO_API_KEY` |
| **DeepSeek** | Direct DeepSeek API access | Set `DEEPSEEK_API_KEY` |
| **NVIDIA NIM** | Nemotron modelos via build.nvidia.com or local NIM | Set `NVIDIA_API_KEY` (optional: `NVIDIA_BASE_URL`) |
| **GitHub Copilot** | GitHub Copilot subscription (GPT-5.x, Claude, Gemini, etc.) | OAuth via `hermes modelo`, or `COPILOT_GITHUB_TOKEN` / `GH_TOKEN` |
| **GitHub Copilot ACP** | Copilot ACP agente backend (spawns local `copilot` CLI) | `hermes modelo` (requires `copilot` CLI + `copilot login`) |
| **Custom Endpoint** | VLLM, SGLang, Ollama, or any OpenAI-compatible API | Set base URL + API key |

For most first-time users: choose a proveedor, accept the defaults unless you know why you're changing them. The full proveedor catalog with env vars and setup steps lives on the [Proveedors](../integrations/proveedors.md) page.

:::caution Minimum contexto: 64K tokens
Hermes Agente requires a modelo with at least **64,000 tokens** of contexto. Modelos with smaller windows cannot maintain enough working memoria for multi-step herramienta-calling workflows and will be rejected at startup. Most hosted modelos (Claude, GPT, Gemini, Qwen, DeepSeek) meet this easily. If you're running a local modelo, set its contexto size to at least 64K (e.g. `--ctx-size 65536` for llama.cpp or `-c 65536` for Ollama).
:::

:::tip
You can switch proveedors at any time with `hermes modelo` — no lock-in. For a full list of all supported proveedors and setup details, see [AI Proveedors](../integrations/proveedors.md).
:::

### How settings are stored

Hermes separates secrets from normal config:

- **Secrets and tokens** → `~/.hermes/.env`
- **Non-secret settings** → `~/.hermes/config.yaml`

The easiest way to set values correctly is through the CLI:

```bash
hermes config set modelo anthropic/claude-opus-4.6
hermes config set terminal.backend docker
hermes config set OPENROUTER_API_KEY sk-or-...
```

The right value goes to the right file automatically.

## 3. Run Your First Chat

```bash
hermes            # classic CLI
hermes --tui      # modern TUI (recommended)
```

You'll see a welcome banner with your modelo, available herramientas, and habilidads. Use a prompt that's specific and easy to verify:

:::tip Pick your interface
Hermes ships with two terminal interfaces: the classic `prompt_herramientakit` CLI and a newer [TUI](../guia-usuario/tui.md) with modal overlays, mouse selection, and non-blocking input. Both share the same sessions, slash commands, and config — try each with `hermes` vs `hermes --tui`.
:::

```
Summarize this repo in 5 bullets and tell me what the main entrypoint is.
```

```
Check my current directory and tell me what looks like the main project file.
```

```
Help me set up a clean GitHub PR workflow for this codebase.
```

**What success looks like:**

- The banner shows your chosen modelo/proveedor
- Hermes replies without error
- It can use a herramienta if needed (terminal, file read, web search)
- The conversation continues normally for more than one turn

If that works, you're past the hardest part.

## 4. Verify Sessions Work

Before moving on, make sure resume works:

```bash
hermes --continue    # Resume the most recent session
hermes -c            # Short form
```

That should bring you back to the session you just had. If it doesn't, check whether you're in the same perfil and whether the session actually saved. This matters later when you're juggling multiple setups or machines.

## 5. Try Key Features

### Use the terminal

```
❯ What's my disk usage? Show the top 5 largest directories.
```

The agente runs terminal commands on your behalf and shows results.

### Slash commands

Type `/` to see an autocomplete dropdown of all commands:

| Command | What it does |
|---------|-------------|
| `/help` | Show all available commands |
| `/herramientas` | List available herramientas |
| `/modelo` | Switch modelos interactively |
| `/personality pirate` | Try a fun personality |
| `/save` | Save the conversation |

### Multi-line input

Press `Alt+Enter`, `Ctrl+J`, or `Shift+Enter` to add a new line. `Shift+Enter` requires a terminal that sends it as a distinct sequence (Kitty / foot / WezTerm / Ghostty by default; iTerm2 / Alacritty / VS Code terminal once the Kitty keyboard protocol is enabled). `Alt+Enter` and `Ctrl+J` work in every terminal.

### Interrupt the agente

If the agente is taking too long, type a new message and press Enter — it interrupts the current task and switches to your new instructions. `Ctrl+C` also works.

## 6. Add the Next Layer

Only after the base chat works. Pick what you need:

### Bot or shared assistant

```bash
hermes puerta de enlace setup    # Interactive platform configuración
```

Connect [Telegram](/guia-usuario/messaging/telegram), [Discord](/guia-usuario/messaging/discord), [Slack](/guia-usuario/messaging/slack), [WhatsApp](/guia-usuario/messaging/whatsapp), [Signal](/guia-usuario/messaging/signal), [Email](/guia-usuario/messaging/email), or [Home Assistant](/guia-usuario/messaging/homeassistant), or [Microsoft Teams](/guia-usuario/messaging/teams).

### Automation and herramientas

- `hermes herramientas` — tune herramienta access per platform
- `hermes habilidads` — browse and install reusable workflows
- Cron — only after your bot or CLI setup is stable

### Sandboxed terminal

For safety, run the agente in a Docker container or on a remote server:

```bash
hermes config set terminal.backend docker    # Docker isolation
hermes config set terminal.backend ssh       # Remote server
```

### Voice mode

```bash
# From the Hermes install directory (the curl installer placed it at
# ~/.hermes/hermes-agente on Linux/macOS or %LOCALAPPDATA%\hermes\hermes-agente on Windows):
cd ~/.hermes/hermes-agente
uv pip install -e ".[voice]"
# Includes faster-whisper for free local speech-to-text
```

Then in the CLI: `/voice on`. Press `Ctrl+B` to record. See [Voice Mode](../guia-usuario/features/voice-mode.md).

### Habilidads

Habilidads are on-demand instruction documents that teach Hermes how to do a specific task — deploy to Kubernetes, open a GitHub PR, fine-tune a modelo, search for GIFs. Each is a `SKILL.md` file with a name, a description, and a step-by-step procedure. The agente reads the short descriptions for free and only loads a habilidad's full content when a task actually calls for it, so adding habilidads doesn't bloat every request.

Hermes ships with a catalog of bundled habilidads already installed in `~/.hermes/habilidads/`. You can add more from the Habilidads Hub, or write your own.

**Browse and install from the hub:**

```bash
hermes habilidads browse                      # list everything available
hermes habilidads search kubernetes           # find habilidads by keyword
hermes habilidads install openai/habilidads/k8s   # install one (runs a security scan first)
```

The install argument is a `source/path` slug from the hub — `openai/habilidads/k8s` means the `k8s` habilidad from OpenAI's catalog. `hermes habilidads browse` shows the exact slugs to use.

**Use a habilidad** — every installed habilidad becomes a slash command automatically:

```bash
/k8s deploy the staging manifest          # run the habilidad with a request
/k8s                                       # load it and let Hermes ask what you need
```

This works in the CLI and in any connected messaging platform. You don't have to install everything up front — the agente picks the right bundled habilidad on its own during normal conversation when a task matches one.

See [Habilidads System](../guia-usuario/features/habilidads.md) for writing your own, external habilidad directories, and the full hub source list.

### MCP servers

```yaml
# Add to ~/.hermes/config.yaml
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelocontextoprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxx"
```

### Editor integration (ACP)

ACP support ships with the standard `[all]` extras, so the curl installer already includes it. Just run:

```bash
hermes acp
```

(If you installed without `[all]`, run `cd ~/.hermes/hermes-agente && uv pip install -e ".[acp]"` first.)

See [ACP Editor Integration](../guia-usuario/features/acp.md).

---

## Common Failure Modes

These are the problems that waste the most time:

| Symptom | Likely cause | Fix |
|---|---|---|
| Hermes opens but gives empty or broken replies | Proveedor auth or modelo selection is wrong | Run `hermes modelo` again and confirm proveedor, modelo, and auth |
| Custom endpoint "works" but returns garbage | Wrong base URL, modelo name, or not actually OpenAI-compatible | Verify the endpoint in a separate client first |
| Puerta de enlace starts but nobody can message it | Bot token, allowlist, or platform setup is incomplete | Re-run `hermes puerta de enlace setup` and check `hermes puerta de enlace status` |
| `hermes --continue` can't find old session | Switched perfils or session never saved | Check `hermes sessions list` and confirm you're in the right perfil |
| Modelo unavailable or odd fallback behavior | Proveedor routing or fallback settings are too aggressive | Keep routing off until the base proveedor is stable |
| `hermes doctor` flags config problems | Config values are missing or stale | Fix the config, retest a plain chat before adding features |

## Recovery Herramientakit

When something feels off, use this order:

1. `hermes doctor`
2. `hermes modelo`
3. `hermes setup`
4. `hermes sessions list`
5. `hermes --continue`
6. `hermes puerta de enlace status`

That sequence gets you from "broken vibes" back to a known state fast.

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `hermes` | Start chatting |
| `hermes modelo` | Choose your LLM proveedor and modelo |
| `hermes herramientas` | Configure which herramientas are enabled per platform |
| `hermes setup` | Full setup wizard (configures everything at once) |
| `hermes doctor` | Diagnose issues |
| `hermes update` | Update to latest version |
| `hermes puerta de enlace` | Start the messaging puerta de enlace |
| `hermes --continue` | Resume last session |

## Next Steps

- **[CLI Guide](../guia-usuario/cli.md)** — Master the terminal interface
- **[Configuración](../guia-usuario/configuración.md)** — Customize your setup
- **[Messaging Puerta de enlace](../guia-usuario/messaging/index.md)** — Connect Telegram, Discord, Slack, WhatsApp, Signal, Email, Home Assistant, Teams, and more
- **[Herramientas & Herramientasets](../guia-usuario/features/herramientas.md)** — Explore available capabilities
- **[AI Proveedors](../integrations/proveedors.md)** — Full proveedor list and setup details
- **[Habilidads System](../guia-usuario/features/habilidads.md)** — Reusable workflows and knowledge
- **[Tips & Best Practices](../guides/tips.md)** — Power user tips

---