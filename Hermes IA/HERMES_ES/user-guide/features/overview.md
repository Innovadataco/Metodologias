<!-- source: website/docs/guia-usuario/features/overview.md -->
# Features Overview

# Features Overview

Hermes Agente includes a rich set of capabilities that extend far beyond basic chat. From persistent memoria and file-aware contexto to browser automation and voice conversations, these features work together to make Hermes a powerful autonomous assistant.

:::tip Don't know where to start?
`hermes setup --portal` covers a modelo proveedor plus all four Herramienta Puerta de enlace herramientas (web search, image generation, TTS, browser) in one command. See [Nous Portal](/integrations/nous-portal).
:::

## Core

- **[Herramientas & Herramientasets](herramientas.md)** — Herramientas are functions that extend the agente's capabilities. They're organized into logical herramientasets that can be enabled or disabled per platform, covering web search, terminal execution, file editing, memoria, delegation, and more.
- **[Habilidads System](habilidads.md)** — On-demand knowledge documents the agente can load when needed. Habilidads follow a progressive disclosure pattern to minimize token usage and are compatible with the [agentehabilidads.io](https://agentehabilidads.io/specification) open standard.
- **[Persistent Memoria](memoria.md)** — Bounded, curated memoria that persists across sessions. Hermes remembers your preferences, projects, environment, and things it has learned via `MEMORY.md` and `USER.md`.
- **[Contexto Files](contexto-files.md)** — Hermes automatically discovers and loads project contexto files (`.hermes.md`, `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules`) that shape how it behaves in your project.
- **[Contexto References](contexto-references.md)** — Type `@` followed by a reference to inject files, folders, git diffs, and URLs directly into your messages. Hermes expands the reference inline and appends the content automatically.
- **[Checkpoints](../checkpoints-and-rollback.md)** — Hermes automatically snapshots your working directory before making file changes, giving you a safety net to roll back with `/rollback` if something goes wrong.

## Automation

- **[Scheduled Tasks (Cron)](cron.md)** — Schedule tasks to run automatically with natural language or cron expressions. Jobs can attach habilidads, deliver results to any platform, and support pause/resume/edit operations.
- **[Subagente Delegation](delegation.md)** — The `delegate_task` herramienta spawns child agente instances with isolated contexto, restricted herramientasets, and their own terminal sessions. Run 3 concurrent subagentes by default (configurable) for parallel workstreams.
- **[Code Execution](code-execution.md)** — The `execute_code` herramienta lets the agente write Python scripts that call Hermes herramientas programmatically, collapsing multi-step workflows into a single LLM turn via sandboxed RPC execution.
- **[Event Hooks](hooks.md)** — Run custom code at key lifecycle points. Puerta de enlace hooks handle logging, alerts, and webhooks; complemento hooks handle herramienta interception, metrics, and guardrails.
- **[Batch Processing](batch-processing.md)** — Run the Hermes agente across hundreds or thousands of prompts in parallel, generating structured ShareGPT-format trajectory data for training data generation or evaluation.

## Media & Web

- **[Voice Mode](voice-mode.md)** — Full voice interaction across CLI and messaging platforms. Talk to the agente using your microphone, hear spoken replies, and have live voice conversations in Discord voice channels.
- **[Browser Automation](browser.md)** — Full browser automation with multiple backends: Browserbase cloud, Browser Use cloud, local Chrome/Brave/Chromium/Edge via CDP, or local Chromium. Navigate websites, fill forms, and extract information.
- **[Vision & Image Paste](vision.md)** — Multimodal vision support. Paste images from your clipboard into the CLI and ask the agente to analyze, describe, or work with them using any vision-capable modelo.
- **[Image Generation](image-generation.md)** — Generate images from text prompts using FAL.ai. Nine modelos supported (FLUX 2 Klein/Pro, GPT-Image 1.5/2, Nano Banana Pro, Ideogram V3, Recraft V4 Pro, Qwen, Z-Image Turbo); pick one via `hermes herramientas`.
- **[Voice & TTS](tts.md)** — Text-to-speech output and voice message transcription across all messaging platforms, with ten native proveedor options: Edge TTS (free), ElevenLabs, OpenAI TTS, MiniMax, Mistral Voxtral, Google Gemini, xAI, NeuTTS, KittenTTS, and Piper — plus custom command proveedors for any local TTS CLI.

## Integrations

- **[MCP Integration](mcp.md)** — Connect to any MCP server via stdio or HTTP transport. Access external herramientas from GitHub, databases, file systems, and internal APIs without writing native Hermes herramientas. Includes per-server herramienta filtering and sampling support.
- **[Proveedor Routing](proveedor-routing.md)** — Fine-grained control over which AI proveedors handle your requests. Optimize for cost, speed, or quality with sorting, whitelists, blacklists, and priority ordering.
- **[Fallback Proveedors](fallback-proveedors.md)** — Automatic failover to backup LLM proveedors when your primary modelo encounters errors, including independent fallback for auxiliary tasks like vision and compression.
- **[Credential Pools](credential-pools.md)** — Distribute API calls across multiple keys for the same proveedor. Automatic rotation on rate limits or failures.
- **[Prompt caching](../configuración#prompt-caching)** — Built-in cross-session 1-hour prefix cache for Claude on native Anthropic, OpenRouter, and Nous Portal. Always-on; no configuración required.
- **[Memoria Proveedors](memoria-proveedors.md)** — Plug in external memoria backends (Honcho, OpenViking, Mem0, Hindsight, Holographic, RetainDB, ByteRover, Supermemoria) for cross-session user modeloing and personalization beyond the built-in memoria system.
- **[API Server](api-server.md)** — Expose Hermes as an OpenAI-compatible HTTP endpoint. Connect any frontend that speaks the OpenAI format — Open WebUI, LobeChat, LibreChat, and more.
- **[IDE Integration (ACP)](acp.md)** — Use Hermes inside ACP-compatible editors such as VS Code, Zed, and JetBrains. Chat, herramienta activity, file diffs, and terminal commands render inside your editor.
- **[Batch Processing](batch-processing.md)** — Run the agente over many prompts or tasks in parallel from the CLI, with structured outputs and trajectory capture suitable for evals or downstream training pipelines.

## Customization

- **[Personality & SOUL.md](personality.md)** — Fully customizable agente personality. `SOUL.md` is the primary identity file — the first thing in the system prompt — and you can swap in built-in or custom `/personality` presets per session.
- **[Skins & Themes](skins.md)** — Customize the CLI's visual presentation: banner colors, spinner faces and verbs, response-box labels, branding text, and the herramienta activity prefix.
- **[Complementos](complementos.md)** — Add custom herramientas, hooks, and integrations without modifying core code. Three complemento types: general complementos (herramientas/hooks), memoria proveedors (cross-session knowledge), and contexto engines (alternative contexto management). Managed via the unified `hermes complementos` interactive UI.

---