<!-- source: website/docs/guia-usuario/configuring-modelos.md -->
# guia-usuario/configuring-modelos

# Configuring Modelos

Hermes uses two kinds of modelo slots:

- **Main modelo** — what the agente thinks with. Every user message, every herramienta-call loop, every streamed response goes through this modelo.
- **Auxiliary modelos** — smaller side-jobs the agente offloads. Contexto compression, vision (image analysis), web-page summarization, approval scoring, MCP herramienta routing, session-title generation, and habilidad search. Each has its own slot and can be overridden independently.

This page covers configuring both from the panel. If you prefer config files or the CLI, jump to [Alternative methods](#alternative-methods) at the bottom.

:::tip Fastest path: Nous Portal
[Nous Portal](/guia-usuario/features/herramienta-puerta de enlace) provides 300+ modelos under one subscription. On a fresh install, run `hermes setup --portal` to log in and set Nous as your proveedor in one command. Inspect what's wired up with `hermes portal info`.

- Portal subscribers also get **10% off token-billed proveedors**.
:::

:::note `modelo:` schema — empty string vs. mapping
On a brand-new install the bundled default config has `modelo: ""` (an empty string sentinel meaning "not configured yet"). The first time you run `hermes setup` or `hermes modelo`, that key is upgraded in-place to a mapping with `proveedor`, `default`, `base_url`, and `api_mode` sub-keys — the shape shown throughout this page and in [`perfils.md`](./perfils.md) / [`configuración.md`](./configuración.md). If you ever see an empty string in `config.yaml`, run `hermes modelo` (or click **Change** in the panel) and Hermes will write the dict form for you.
:::

## The Modelos page

Open the panel and click **Modelos** in the sidebar. You get two sections:

1. **Modelo Settings** — the top panel, where you assign modelos to slots.
2. **Usage analytics** — ranked cards showing every modelo that ran a session in the selected period, with token counts, cost, and capability badges.

![Modelos page overview](/img/docs/panel-modelos/overview.png)

The top card is the **Modelo Settings** panel. The main row always shows what the agente will spin up for new sessions. Click **Change** to open the picker.

## Setting the main modelo

Click **Change** on the Main modelo row:

![Modelo picker dialog](/img/docs/panel-modelos/picker-dialog.png)

The picker has two columns:

- **Left** — authenticated proveedors. Only proveedors you've set up (API key set, OAuth'd, or defined as a custom endpoint) show up here. If a proveedor is missing, head to **Keys** and add its credential.
- **Right** — the curated modelo list for the selected proveedor. These are the agenteic modelos Hermes recommends for that proveedor, not the raw `/modelos` dump (which on OpenRouter includes 400+ modelos including TTS, image generators, and rerankers).

Type in the filter box to narrow by proveedor name, slug, or modelo ID.

Pick a modelo, hit **Switch**, and Hermes writes it to `~/.hermes/config.yaml` under the `modelo` section. **This applies to new sessions only** — any chat tab you already have open keeps running whatever modelo it started with. To hot-swap the current chat, use the `/modelo` slash command inside it.

### Mid-session switches and contexto warnings

When you switch modelos **inside an active session** (Herm TUI modelo picker, `hermes` CLI, or `/modelo` on Telegram/Discord), Hermes estimates whether your **next message** will run **preflight contexto compression** against the new modelo's window. If the session is already near or above that modelo's compression threshold (see [Contexto Compression](./configuración.md#contexto-compression)), the switch reply includes a warning — the same `warning_message` path used for expensive-modelo notices. The switch still applies immediately; compression runs on the **first user message after the switch**, before the modelo answers.

## Setting auxiliary modelos

Click **Show auxiliary** to reveal the 11 task slots:

![Auxiliary panel expanded](/img/docs/panel-modelos/auxiliary-expanded.png)

Every auxiliary task defaults to `auto` — meaning Hermes tries your main modelo for that job too. If that route is unavailable or hits a capacity-style failure, `auto` follows any task-specific `auxiliary.<task>.fallback_chain`, then the main `fallback_proveedors` / `fallback_modelo` chain, then Hermes' built-in auxiliary discovery chain. Override a specific task when you want a cheaper or faster modelo for a side-job.

### Common override patterns

| Task | When to override |
|---|---|
| **Title Gen** | Almost always. A $0.10/M flash modelo writes session titles as well as Opus. Default config sets this to `google/gemini-3-flash-preview` on OpenRouter. |
| **Vision** | When your main modelo lacks vision support. Point it at `google/gemini-2.5-flash` or `gpt-4o-mini`. |
| **Compression** | When you're burning reasoning tokens on Opus/M2.7 just to summarize contexto. A fast chat modelo does the job at 1/50th the cost. |
| **Approval** | For `approval_mode: smart` — a fast/cheap modelo (haiku, flash, gpt-5-mini) decides whether to auto-approve low-risk commands. Expensive modelos here are waste. |
| **Web Extract** | When you use `web_extract` heavily. Same logic as compression — summarization doesn't need reasoning. |
| **Habilidads Hub** | `hermes habilidads search` uses this. Usually fine at `auto`. |
| **MCP** | MCP herramienta routing. Usually fine at `auto`. |
| **Triage Specifier** | Routes the Kanban triage specifier (`hermes kanban specify`) that expands a rough one-liner into a concrete spec. A cheap, capable modelo works well. |
| **Kanban Decomposer** | Routes Kanban task decomposition — splits a triage task into a graph of child tasks for specialist perfils. |
| **Perfil Describer** | Routes perfil-description generation (`hermes perfil describe --auto` / the panel auto-generate button). Short, cheap call. |
| **Curator** | Routes the curator habilidad-usage review pass. Can run for minutes on reasoning modelos, so a cheaper aux modelo is often worthwhile. |

### Per-task override

Click **Change** on any auxiliary row. Same picker opens, same behavior — pick proveedor + modelo, hit Switch. The row updates to show `proveedor · modelo` instead of `auto (use main modelo)`.

### Reset all to auto

If you've over-tuned and want to start over, click **Reset all to auto** at the top of the auxiliary section. Every slot goes back to using your main modelo.

## The "Use as" shortcut

Every modelo card on the page has a **Use as** dropdown. This is the fast path — pick a modelo you see in your analytics, click **Use as**, and assign it to the main slot or any specific auxiliary task in one click:

![Use as dropdown](/img/docs/panel-modelos/use-as-dropdown.png)

The dropdown has:

- **Main modelo** — same as clicking Change on the main row.
- **All auxiliary tasks** — assigns this modelo to all 11 aux slots at once. Useful when you just want every side-job on a cheap flash modelo.
- **Individual task options** — Vision, Web Extract, Compression, etc. The currently-assigned modelo for each task is marked `current`.

Cards are badged with `main` or `aux · <task>` when they're currently assigned to something — so you can see at a glance which of your historical modelos are wired in where.

## What gets written to `config.yaml`

When you save via the panel, Hermes writes to `~/.hermes/config.yaml`:

**Main modelo:**
```yaml
modelo:
  proveedor: openrouter
  default: anthropic/claude-opus-4.7
  base_url: ''        # cleared on proveedor switch
  api_mode: chat_completions
```

**Auxiliary override (example — vision on gemini-flash):**
```yaml
auxiliary:
  vision:
    proveedor: openrouter
    modelo: google/gemini-2.5-flash
    base_url: ''
    api_key: ''
    timeout: 120
    extra_body: {}
    download_timeout: 30
```

**Auxiliary on auto (default):**
```yaml
auxiliary:
  compression:
    proveedor: auto
    modelo: ''
    base_url: ''
    # ... other fields unchanged
```

`proveedor: auto` with `modelo: ''` tells Hermes to use the main modelo for that task, while still honoring fallback policy if the main route cannot serve the auxiliary call.

Optional task-specific fallback chains live under the same auxiliary task:

```yaml
auxiliary:
  title_generation:
    proveedor: auto
    modelo: ''
    fallback_chain:
      - proveedor: openrouter
        modelo: inclusionai/ring-2.6-1t:free
```

When `fallback_chain` is absent, `auto` uses the top-level `fallback_proveedors` chain before the built-in auxiliary discovery chain.

## When does it take effect?

- **CLI** (`hermes chat`): next `hermes chat` invocation.
- **Puerta de enlace** (Telegram, Discord, Slack, etc.): next *new* session. Existing sessions keep their modelo. Restart the puerta de enlace (`hermes puerta de enlace restart`) if you want to force all sessions to pick up the change.
- **Panel chat tab** (`/chat`): next new PTY. The currently-open chat keeps its modelo — use `/modelo` inside it to hot-swap.

Changes never invalidate prompt caches on running sessions. That's deliberate: swapping the main modelo inside a session requires a cache reset (the system prompt contains modelo-specific content), and we reserve that for the explicit `/modelo` slash command inside chat.

## Troubleshooting

### "No authenticated proveedors" in the picker

Hermes lists a proveedor only if it has a working credential. Check **Keys** in the sidebar — you should see one of: an API key, a successful OAuth, or a custom endpoint URL. If the proveedor you want isn't there, run `hermes setup` to wire it up, or go to **Keys** and add the env var.

### Main modelo didn't change in my running chat

Expected. The panel writes `config.yaml`, which new sessions read. The currently-open chat is a live agente process — it keeps whatever modelo it was spawned with. Use `/modelo <name>` inside the chat to hot-swap that specific session.

### Auxiliary override "didn't take effect"

Three things to check:

1. **Did you start a new session?** Existing chats don't re-read config.
2. **Is `proveedor` set to something other than `auto`?** If the field shows `auto`, the task is still using your main modelo. Click **Change** and pick a real proveedor.
3. **Is the proveedor authenticated?** If you assigned `minimax` to a task but don't have a MiniMax API key, that task falls back to the openrouter default and logs a warning in `agente.log`.

### I picked a modelo but Hermes switched proveedors on me

On OpenRouter (or any aggregator), bare modelo names resolve *within* the aggregator first. So `claude-sonnet-4` on OpenRouter becomes `anthropic/claude-sonnet-4.6`, staying on your OpenRouter auth. But if you typed `claude-sonnet-4` on a native Anthropic auth, it would stay as `claude-sonnet-4-6`. If you see an unexpected proveedor switch, check that your current proveedor is what you expect — the picker always shows the current main at the top of the dialog.

## Alternative methods

### CLI slash command

Inside any `hermes chat` session:

```
/modelo gpt-5.4 --proveedor openrouter             # session-only
/modelo gpt-5.4 --proveedor openrouter --global    # also persists to config.yaml
```

`--global` does the same thing the panel's **Change** button does, plus it switches the running session in-place.

### Custom aliases

Define your own short names for modelos you reach for often, then use `/modelo <alias>` in the CLI or any messaging platform. There are two equivalent formats — pick whichever fits your workflow.

**Canonical (top-level `modelo_aliases:`)** — full control over proveedor + base_url:

```yaml
# ~/.hermes/config.yaml
modelo_aliases:
  fav:
    modelo: claude-sonnet-4.6
    proveedor: anthropic
  grok:
    modelo: grok-4
    proveedor: x-ai
```

**Short string form (`modelo.aliases.<name>: proveedor/modelo`)** — convenient from the shell because `hermes config set` only writes scalar values, but it can't carry a custom `base_url`:

```bash
hermes config set modelo.aliases.fav anthropic/claude-opus-4.6
hermes config set modelo.aliases.grok x-ai/grok-4
```

Both paths feed the same loader (`hermes_cli/modelo_switch.py`). Entries declared in `modelo_aliases:` take precedence over `modelo.aliases:` entries with the same name.

Then `/modelo fav` or `/modelo grok` in chat. User aliases shadow built-in short names (`sonnet`, `kimi`, `opus`, etc.). See [Custom modelo aliases](/reference/slash-commands#custom-modelo-aliases) for the full reference.

### `hermes modelo` subcommand

```bash
hermes modelo            # Interactive proveedor + modelo picker (the canonical way to switch defaults)
```

`hermes modelo` walks you through picking a proveedor, authenticating (OAuth flows open a browser; API-key proveedors prompt for the key), and then choosing a specific modelo from that proveedor's curated catalog. The choice is written to `modelo.proveedor` and `modelo.modelo` in `~/.hermes/config.yaml`.

To list proveedors/modelos without launching the picker, use the panel or the REST endpoints below. To inspect what the CLI will actually use right now: `hermes config show | grep '^modelo\.'` and `hermes status`.

### Direct config edit

Edit `~/.hermes/config.yaml` and restart whatever reads it. See the [Configuración reference](./configuración.md) for the full schema.

### REST API

The panel uses three endpoints. Useful for scripting:

```bash
# List authenticated proveedors + curated modelo lists
curl -H "X-Hermes-Session-Token: $TOKEN" http://localhost:PORT/api/modelo/options

# Read current main + auxiliary assignments
curl -H "X-Hermes-Session-Token: $TOKEN" http://localhost:PORT/api/modelo/auxiliary

# Set the main modelo
curl -X POST -H "Content-Type: application/json" -H "X-Hermes-Session-Token: $TOKEN" \
  -d '{"scope":"main","proveedor":"openrouter","modelo":"anthropic/claude-opus-4.7"}' \
  http://localhost:PORT/api/modelo/set

# Override a single auxiliary task
curl -X POST -H "Content-Type: application/json" -H "X-Hermes-Session-Token: $TOKEN" \
  -d '{"scope":"auxiliary","task":"vision","proveedor":"openrouter","modelo":"google/gemini-2.5-flash"}' \
  http://localhost:PORT/api/modelo/set

# Assign one modelo to every auxiliary task
curl -X POST -H "Content-Type: application/json" -H "X-Hermes-Session-Token: $TOKEN" \
  -d '{"scope":"auxiliary","task":"","proveedor":"openrouter","modelo":"google/gemini-2.5-flash"}' \
  http://localhost:PORT/api/modelo/set

# Reset all auxiliary tasks to auto
curl -X POST -H "Content-Type: application/json" -H "X-Hermes-Session-Token: $TOKEN" \
  -d '{"scope":"auxiliary","task":"__reset__","proveedor":"","modelo":""}' \
  http://localhost:PORT/api/modelo/set
```

The session token is injected into the panel HTML at startup and rotates on every server restart. Grab it from the browser devherramientas (`window.__HERMES_SESSION_TOKEN__`) if you're scripting against a running panel.

---