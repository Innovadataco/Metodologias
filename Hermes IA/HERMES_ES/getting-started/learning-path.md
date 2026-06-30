<!-- source: website/docs/inicio-rapido/learning-path.md -->
# Learning Path

# Learning Path

Hermes Agente can do a lot — CLI assistant, Telegram/Discord bot, task automation, RL training, and more. This page helps you figure out where to start and what to read based on your experience level and what you're trying to accomplish.

:::tip Start Here
If you haven't installed Hermes Agente yet, begin with the [Installation guide](/inicio-rapido/installation) and then run through the [Quickstart](/inicio-rapido/quickstart). Everything below assumes you have a working installation.
:::

:::tip First-time proveedor setup
First-time users almost always want `hermes setup --portal` — one OAuth covers a modelo plus the four Herramienta Puerta de enlace herramientas (search/image/TTS/browser). See [Nous Portal](/integrations/nous-portal).
:::

## How to Use This Page

- **Know your level?** Jump to the [experience-level table](#by-experience-level) and follow the reading order for your tier.
- **Have a specific goal?** Skip to [By Use Case](#by-use-case) and find the scenario that matches.
- **Just browsing?** Check the [Key Features](#key-features-at-a-glance) table for a quick overview of everything Hermes Agente can do.

## By Experience Level

| Level | Goal | Recommended Reading | Time Estimate |
|---|---|---|---|
| **Beginner** | Get up and running, have basic conversations, use built-in herramientas | [Installation](/inicio-rapido/installation) → [Quickstart](/inicio-rapido/quickstart) → [CLI Usage](/guia-usuario/cli) → [Configuración](/guia-usuario/configuración) | ~1 hour |
| **Intermediate** | Set up messaging bots, use advanced features like memoria, cron jobs, and habilidads | [Sessions](/guia-usuario/sessions) → [Messaging](/guia-usuario/messaging) → [Herramientas](/guia-usuario/features/herramientas) → [Habilidads](/guia-usuario/features/habilidads) → [Memoria](/guia-usuario/features/memoria) → [Cron](/guia-usuario/features/cron) | ~2–3 hours |
| **Advanced** | Build custom herramientas, create habilidads, train modelos with RL, contribute to the project | [Architecture](/guia-desarrollador/architecture) → [Adding Herramientas](/guia-desarrollador/adding-herramientas) → [Creating Habilidads](/guia-desarrollador/creating-habilidads) → [Contributing](/guia-desarrollador/contributing) | ~4–6 hours |

## By Use Case

Pick the scenario that matches what you want to do. Each one links you to the relevant docs in the order you should read them.

### "I want a CLI coding assistant"

Use Hermes Agente as an interactive terminal assistant for writing, reviewing, and running code.

1. [Installation](/inicio-rapido/installation)
2. [Quickstart](/inicio-rapido/quickstart)
3. [CLI Usage](/guia-usuario/cli)
4. [Code Execution](/guia-usuario/features/code-execution)
5. [Contexto Files](/guia-usuario/features/contexto-files)
6. [Tips & Tricks](/guides/tips)

:::tip
Pass files directly into your conversation with contexto files. Hermes Agente can read, edit, and run code in your projects.
:::

### "I want a Telegram/Discord bot"

Deploy Hermes Agente as a bot on your favorite messaging platform.

1. [Installation](/inicio-rapido/installation)
2. [Configuración](/guia-usuario/configuración)
3. [Messaging Overview](/guia-usuario/messaging)
4. [Telegram Setup](/guia-usuario/messaging/telegram)
5. [Discord Setup](/guia-usuario/messaging/discord)
6. [Voice Mode](/guia-usuario/features/voice-mode)
7. [Use Voice Mode with Hermes](/guides/use-voice-mode-with-hermes)
8. [Security](/guia-usuario/security)

For full project examples, see:
- [Daily Briefing Bot](/guides/daily-briefing-bot)
- [Team Telegram Assistant](/guides/team-telegram-assistant)

### "I want to automate tasks"

Schedule recurring tasks, run batch jobs, or chain agente actions together.

1. [Quickstart](/inicio-rapido/quickstart)
2. [Cron Scheduling](/guia-usuario/features/cron)
3. [Batch Processing](/guia-usuario/features/batch-processing)
4. [Delegation](/guia-usuario/features/delegation)
5. [Hooks](/guia-usuario/features/hooks)

:::tip
Cron jobs let Hermes Agente run tasks on a schedule — daily summaries, periodic checks, automated reports — without you being present.
:::

### "I want to build custom herramientas/habilidads"

Extend Hermes Agente with your own herramientas and reusable habilidad packages.

1. [Complementos](/guia-usuario/features/complementos)
2. [Build a Hermes Complemento](/guides/build-a-hermes-complemento)
3. [Herramientas Overview](/guia-usuario/features/herramientas)
4. [Habilidads Overview](/guia-usuario/features/habilidads)
5. [MCP (Modelo Contexto Protocol)](/guia-usuario/features/mcp)
6. [Architecture](/guia-desarrollador/architecture)
7. [Adding Herramientas](/guia-desarrollador/adding-herramientas)
8. [Creating Habilidads](/guia-desarrollador/creating-habilidads)

:::tip
For most custom herramienta creation, start with complementos. The [Adding Herramientas](/guia-desarrollador/adding-herramientas)
page is for built-in Hermes core development, not the usual user/custom-herramienta path.
:::

### "I want to train modelos"

Use reinforcement learning to fine-tune modelo behavior with Hermes Agente's RL training pipeline (powered by [Atropos](https://github.com/NousResearch/atropos)).

1. [Quickstart](/inicio-rapido/quickstart)
2. [Configuración](/guia-usuario/configuración)
3. [Atropos RL Environments](https://github.com/NousResearch/atropos) (external)
4. [Proveedor Routing](/guia-usuario/features/proveedor-routing)
5. [Architecture](/guia-desarrollador/architecture)

:::tip
RL training works best when you already understand the basics of how Hermes Agente handles conversations and herramienta calls. Run through the Beginner path first if you're new.
:::

### "I want to use it as a Python library"

Integrate Hermes Agente into your own Python applications programmatically.

1. [Installation](/inicio-rapido/installation)
2. [Quickstart](/inicio-rapido/quickstart)
3. [Python Library Guide](/guides/python-library)
4. [Architecture](/guia-desarrollador/architecture)
5. [Herramientas](/guia-usuario/features/herramientas)
6. [Sessions](/guia-usuario/sessions)

## Key Features at a Glance

Not sure what's available? Here's a quick directory of major features:

| Feature | What It Does | Link |
|---|---|---|
| **Herramientas** | Built-in herramientas the agente can call (file I/O, search, shell, etc.) | [Herramientas](/guia-usuario/features/herramientas) |
| **Habilidads** | Installable complemento packages that add new capabilities | [Habilidads](/guia-usuario/features/habilidads) |
| **Memoria** | Persistent memoria across sessions | [Memoria](/guia-usuario/features/memoria) |
| **Contexto Files** | Feed files and directories into conversations | [Contexto Files](/guia-usuario/features/contexto-files) |
| **MCP** | Connect to external herramienta servers via Modelo Contexto Protocol | [MCP](/guia-usuario/features/mcp) |
| **Cron** | Schedule recurring agente tasks | [Cron](/guia-usuario/features/cron) |
| **Delegation** | Spawn sub-agentes for parallel work | [Delegation](/guia-usuario/features/delegation) |
| **Code Execution** | Run Python scripts that call Hermes herramientas programmatically | [Code Execution](/guia-usuario/features/code-execution) |
| **Browser** | Web browsing and scraping | [Browser](/guia-usuario/features/browser) |
| **Hooks** | Event-driven callbacks and middleware | [Hooks](/guia-usuario/features/hooks) |
| **Batch Processing** | Process multiple inputs in bulk | [Batch Processing](/guia-usuario/features/batch-processing) |
| **Proveedor Routing** | Route requests across multiple LLM proveedors | [Proveedor Routing](/guia-usuario/features/proveedor-routing) |

## What to Read Next

Based on where you are right now:

- **Just finished installing?** → Head to the [Quickstart](/inicio-rapido/quickstart) to run your first conversation.
- **Completed the Quickstart?** → Read [CLI Usage](/guia-usuario/cli) and [Configuración](/guia-usuario/configuración) to customize your setup.
- **Comfortable with the basics?** → Explore [Herramientas](/guia-usuario/features/herramientas), [Habilidads](/guia-usuario/features/habilidads), and [Memoria](/guia-usuario/features/memoria) to unlock the full power of the agente.
- **Setting up for a team?** → Read [Security](/guia-usuario/security) and [Sessions](/guia-usuario/sessions) to understand access control and conversation management.
- **Ready to build?** → Jump into the [Developer Guide](/guia-desarrollador/architecture) to understand the internals and start contributing.
- **Want practical examples?** → Check out the [Guides](/guides/tips) section for real-world projects and tips.

:::tip
You don't need to read everything. Pick the path that matches your goal, follow the links in order, and you'll be productive quickly. You can always come back to this page to find your next step.
:::

---