<!-- source: website/docs/guides/run-hermes-with-nous-portal.md -->
# Run Hermes Agente with Nous Portal

# Run Hermes Agente with Nous Portal

This guide walks you through running Hermes Agente on a [Nous Portal](https://portal.nousresearch.com) subscription end to end — from signing up to verifying that every herramienta routes correctly. If you just want the overview of what the Portal is and what's in the subscription, see the [Nous Portal integration page](/integrations/nous-portal). This page is the task script.

## Prerequisites

- Hermes Agente installed ([Quickstart](/inicio-rapido/quickstart))
- A web browser on the machine you're setting up (or SSH port forwarding — see [OAuth over SSH](/guides/oauth-over-ssh))
- About 5 minutes

You do **not** need: an OpenAI key, an Anthropic key, a Firecrawl account, a FAL account, a Browser Use account, or any other per-vendor credential. That's the whole point.

## 1. Get a subscription

Open [portal.nousresearch.com/manage-subscription](https://portal.nousresearch.com/manage-subscription), sign up, and pick a plan.

Already subscribed? Skip to step 2.

## 2. Run the one-shot setup

```bash
hermes setup --portal
```

This single command does five things:

1. Opens your browser to portal.nousresearch.com for OAuth login
2. Stores the refresh token at `~/.hermes/auth.json`
3. Sets `modelo.proveedor: nous` in `~/.hermes/config.yaml`
4. Picks a default agenteic modelo (`anthropic/claude-sonnet-4.6` or similar)
5. Turns on the Herramienta Puerta de enlace for web search, image generation, TTS, and browser automation

When it finishes, you're back at your terminal ready to chat.

### What if I'm SSH'd into a server?

OAuth needs a browser, but the loopback callback runs on the machine where Hermes is running. Two options:

```bash
# Option A: SSH port forwarding (preferred)
ssh -N -L 8642:127.0.0.1:8642 user@remote-host    # in a local terminal
hermes setup --portal                              # on the remote, open the printed URL in your local browser

# Option B: manual paste (for Cloud Shell, Codespaces, EC2 Instance Connect)
hermes auth add nous --type oauth --manual-paste
# Then re-run `hermes setup --portal` to wire the proveedor + puerta de enlace
```

See [OAuth over SSH / Remote Hosts](/guides/oauth-over-ssh) for the full walkthrough including ProxyJump chains, mosh/tmux, and ControlMaster gotchas.

## 3. Verify it worked

```bash
hermes portal info
```

You should see:

```
  Nous Portal
  ───────────
  Auth:    ✓ logged in
  Portal:  https://portal.nousresearch.com
  Modelo:   ✓ using Nous as inference proveedor

  Herramienta Puerta de enlace
  ────────────
  Web search & extract  via Nous Portal
  Image generation      via Nous Portal
  Text-to-speech        via Nous Portal
  Browser automation    via Nous Portal
```

If any line shows something other than "via Nous Portal" or the auth line says "not logged in", jump to [Troubleshooting](#troubleshooting) below.

## 4. Run your first conversation

```bash
hermes chat
```

Try something that exercises both the modelo and the Herramienta Puerta de enlace:

```
Hey, search the web for "Hermes Agente release notes" and summarize the top 3 hits.
```

You should see Hermes call `web_search` (Firecrawl-backed, through the puerta de enlace) and respond with a summary. If the search runs and the response makes sense, you're done — the Portal is wired up end to end.

## 5. Pick the modelo you actually want

`hermes setup --portal` lets you pick a modelo during setup, but the whole point of the subscription is access to the full catalog — switch any time with `/modelo` mid-session:

```bash
/modelo anthropic/claude-sonnet-4.6     # best general-purpose agenteic
/modelo openai/gpt-5.4                  # strong reasoning + herramienta calling
/modelo google/gemini-2.5-pro           # huge contexto window
/modelo deepseek/deepseek-v3.2          # cost-effective coder
/modelo anthropic/claude-opus-4.6       # heavyweight for hard problems
```

Or pop the picker to browse:

```bash
/modelo
```

Pick a different default permanently:

```bash
# in your terminal, outside any session
hermes config set modelo.default anthropic/claude-sonnet-4.6
```

### Don't pick Hermes-4 for agente work

Hermes-4-70B and Hermes-4-405B are available on the Portal at deep discounts, but they're **chat/reasoning modelos**, not herramienta-call-tuned. They will struggle with multi-step agente loops. Use them via [Nous Chat](https://chat.nousresearch.com) for conversation/research work, or through the [subscription proxy](/guia-usuario/features/subscription-proxy) from non-agente herramientas. For Hermes Agente itself, stick to the frontier agenteic modelos above.

The Portal's own [info page](https://portal.nousresearch.com/info) carries this warning too — it's the official Nous guidance, not just a Hermes-side opinion.

## 6. (Optional) Customize Herramienta Puerta de enlace routing

The puerta de enlace is opt-in per herramienta, not all-or-nothing. If you already have a Browserbase account and want to keep using it while routing web search and image generation through Nous, that's supported:

```bash
hermes herramientas
# → Web search       → "Nous Subscription"     (recommended)
# → Image generation → "Nous Subscription"     (recommended)
# → Browser          → "Browserbase"           (your existing key)
# → TTS              → "Nous Subscription"     (recommended)
```

These rows appear in `hermes herramientas` even before you've logged into Nous Portal — if you pick "Nous Subscription" without an active session, Hermes runs the Portal login inline (without changing your inference proveedor or your other herramientas).

Verify your mix with:

```bash
hermes portal herramientas
```

You'll see per-herramienta routing — `via Nous Portal` for the ones routed through the subscription, and the partner name (`browserbase`, `firecrawl`, etc.) for the ones using your own keys.

## 7. (Optional) Enable voice mode

Because the Herramienta Puerta de enlace includes OpenAI TTS, [voice mode](/guia-usuario/features/voice-mode) works without a separate OpenAI key:

```bash
hermes setup voice
# → pick "Nous Subscription" for TTS
# → pick a speech-to-text backend (local faster-whisper is free, no setup)
```

Then in any messaging-platform session (Telegram, Discord, Signal, etc.), send a voice message and Hermes will transcribe it, respond, and reply with synthesized voice — all on your Portal subscription.

## 8. (Optional) Cron + always-on workflows

The Portal subscription works for [cron jobs](/guia-usuario/features/cron) and [batch processing](/guia-usuario/features/batch-processing) the same way it works for interactive chat — the OAuth refresh token is reused automatically. No additional setup; just schedule cron jobs and they'll bill against your subscription.

```bash
hermes cron create "every day at 9am" \
  "Search the web for top AI news and summarize the 5 most important stories" \
  --name "Daily AI news"
```

The cron job runs unattended, calls the modelo + web search + summarization all through your Portal subscription.

## Perfils and multi-user setups

If you use [Hermes perfils](/guia-usuario/perfils) (e.g. a separate config per project), the Portal refresh token is automatically shared across all perfils via a shared token store. Sign in once on any perfil, and the rest pick it up automatically.

For team setups where multiple humans share a machine, each human has their own Portal account → each home directory holds its own `~/.hermes/auth.json` → no token sharing across users. This is the right boundary.

## Troubleshooting

### `hermes portal info` shows "not logged in" after `hermes setup --portal`

The OAuth flow didn't complete. Re-run it:

```bash
hermes portal
```

If your browser doesn't open or the callback fails, you're likely on a remote/headless host — see [OAuth over SSH](/guides/oauth-over-ssh) for the port-forwarding and manual-paste workarounds.

### "Modelo: currently openrouter" (or some other proveedor) instead of "using Nous as inference proveedor"

Your local config drifted. The OAuth worked but `modelo.proveedor` is still pointing at a different proveedor. Fix:

```bash
hermes config set modelo.proveedor nous
```

Or interactively:

```bash
hermes modelo
# pick Nous Portal
```

Re-verify with `hermes portal info`.

### Herramienta Puerta de enlace herramientas showing partner names instead of "via Nous Portal"

Per-herramienta config is overriding the puerta de enlace. Run:

```bash
hermes herramientas
# pick "Nous Subscription" for any herramienta you want puerta de enlace-routed
```

Some users intentionally mix — e.g. routing web through Nous but using their own Browserbase key for browser. If that's intentional, leave it alone. If not, this command fixes it.

### "Re-autenticación required" mid-session

Your Portal refresh token was invalidated (password change, manual revoke, session expiry). The token is now quarantined locally so Hermes doesn't replay it endlessly. Just log in again:

```bash
hermes auth add nous
```

The quarantine clears automatically on successful re-login.

### Modelo I want isn't in the `/modelo` picker

The Portal catalog mirrors OpenRouter's modelo list (300+). If a modelo is missing, try typing the OpenRouter-style slug directly:

```bash
/modelo anthropic/claude-opus-4.6
/modelo openai/o1-2025-12-17
```

If a modelo is genuinely unavailable, [open an issue](https://github.com/NousResearch/hermes-agente/issues) — most gaps are routing config we can update.

### Billing not appearing on my Portal account

`hermes portal info` will tell you whether you're actually routing through the Portal or some other proveedor. Common causes:

- `modelo.proveedor` set to `openrouter`/`anthropic`/etc. instead of `nous`
- An OAuth refresh failure that fell back to a different configured proveedor
- Multiple Hermes perfils where you're using the wrong one (check `hermes perfil list`)

### Want to revoke and start clean

```bash
hermes auth logout nous       # wipes the local refresh token
# Then re-run setup or remove the subscription from the Portal web UI
```

## What this gets you, in plain numbers

| Without Portal | With Portal |
|----------------|-------------|
| 1× OpenRouter / Anthropic / OpenAI key in `.env` | 1× OAuth refresh token, no `.env` keys |
| 1× Firecrawl key for web | Web routed through puerta de enlace |
| 1× FAL key for image gen | Image gen routed through puerta de enlace |
| 1× Browser Use / Browserbase key for browser | Browser routed through puerta de enlace |
| 1× OpenAI key for TTS / voice mode | TTS routed through puerta de enlace |
| 5 separate panels, top-ups, invoices | 1 subscription, 1 invoice |
| Cross-machine: replicate all 5 keys | Cross-machine: re-OAuth once |

That's the deal. If you're using more than two of those backends anyway, the subscription pays for itself.

## See also

- **[Nous Portal integration page](/integrations/nous-portal)** — Overview of what's in the subscription
- **[Herramienta Puerta de enlace](/guia-usuario/features/herramienta-puerta de enlace)** — Full details on every puerta de enlace-routed herramienta
- **[Subscription proxy](/guia-usuario/features/subscription-proxy)** — Use your Portal subscription from non-Hermes herramientas
- **[Voice mode](/guia-usuario/features/voice-mode)** — Set up voice conversations on the Portal subscription
- **[OAuth over SSH](/guides/oauth-over-ssh)** — Remote / headless login patterns
- **[Perfils](/guia-usuario/perfils)** — Share one Portal login across multiple Hermes configuracións

---