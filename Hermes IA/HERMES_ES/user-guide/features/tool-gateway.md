<!-- source: website/docs/guia-usuario/features/herramienta-puerta de enlace.md -->
# Nous Herramienta Puerta de enlace

# Nous Herramienta Puerta de enlace

**One subscription. Every herramienta built in.**

The Herramienta Puerta de enlace is included with every paid [Nous Portal](https://portal.nousresearch.com) subscription. It routes Hermes' herramienta calls — web search, image generation, text-to-speech, and cloud browser automation — through infrastructure Nous already runs, so you don't have to sign up with Firecrawl, FAL, OpenAI, Browser Use, or anyone else just to make your agente useful.

<div style={{display: 'flex', gap: '1rem', flexWrap: 'wrap', margin: '1.5rem 0'}}>
  <a href="https://portal.nousresearch.com/manage-subscription" style={{background: 'var(--ifm-color-primary)', color: 'white', padding: '0.75rem 1.5rem', borderRadius: '6px', textDecoration: 'none', fontWeight: 'bold'}}>Start or manage subscription →</a>
</div>

## What's included

| | Herramienta | What you get |
|---|---|---|
| 🔍 | **Web search & extract** | Agente-grade web search and full-page extraction via Firecrawl. No rate limits to worry about — the puerta de enlace handles scaling. |
| 🎨 | **Image generation** | Nine modelos under one endpoint: **FLUX 2 Klein 9B**, **FLUX 2 Pro**, **Z-Image Turbo**, **Nano Banana Pro** (Gemini 3 Pro Image), **GPT Image 1.5**, **GPT Image 2**, **Ideogram V3**, **Recraft V4 Pro**, **Qwen Image**. Pick per-generation with a flag, or let Hermes default to FLUX 2 Klein. |
| 🔊 | **Text-to-speech** | OpenAI TTS voices wired into the `text_to_speech` herramienta. Drop voice notes into Telegram, generate audio for pipelines, narrate anything. |
| 🌐 | **Cloud browser automation** | Headless Chromium sessions via Browser Use. `browser_navigate`, `browser_click`, `browser_type`, `browser_vision` — all the agente-driving primitives, no Browserbase account required. |

All four are pay-as-you-use billed against your Nous subscription. Use any combination — run the puerta de enlace for web and images while keeping your own ElevenLabs key for TTS, or route everything through Nous.

## Why it's here

Building an agente that can actually *do things* means stitching together 5+ API subscriptions — each with their own signup, rate limits, billing, and quirks. The puerta de enlace collapses that into one account:

- **One bill.** Pay Nous; we handle the rest.
- **One signup.** No Firecrawl, FAL, Browser Use, or OpenAI audio accounts to manage.
- **One key.** Your Nous Portal OAuth covers every herramienta.
- **Same quality.** Same backends the direct-key route uses — just fronted by us.

Bring your own keys anytime — per-herramienta, whenever you want to. The puerta de enlace isn't a lock-in, it's a shortcut.

## Get started

There are three ways in — pick whichever fits where you are:

```bash
hermes setup --portal     # Fresh install: Nous OAuth + set Nous as proveedor + turn on the Herramienta Puerta de enlace in one go
```

```bash
hermes modelo              # Switch your inference proveedor to Nous Portal — Hermes then offers to turn on the puerta de enlace for all herramientas
```

```bash
hermes herramientas              # Enable the puerta de enlace per-herramienta — pick "Nous Subscription" for any herramienta you want
```

`hermes setup --portal` and `hermes modelo` are the all-at-once paths: log in once, optionally flip every herramienta to the puerta de enlace. `hermes herramientas` is the à la carte path — turn on just the herramientas you want, one at a time.

**You don't have to log in first.** With `hermes herramientas`, the Nous-managed backends (Web search, Image, Video, TTS, Browser) are always listed, even if you've never signed into Nous Portal. Select one and Hermes runs the Portal login right there if you aren't already authenticated — no need to run `hermes modelo` beforehand. If your Nous OAuth is already active, selecting the backend enables it immediately with no extra prompt. This path only logs you in and turns on the one herramienta you picked — it does **not** switch your inference proveedor, and it does **not** prompt you to enable the puerta de enlace for every other herramienta.

Check what's active at any time:

```bash
hermes portal info        # Portal auth + Herramienta Puerta de enlace routing summary
hermes portal herramientas       # Puerta de enlace catalog with current routing per herramienta
hermes status             # Full system status (Herramienta Puerta de enlace is one section)
```

`hermes portal info` shows a section like:

```
◆ Nous Herramienta Puerta de enlace
  Nous Portal     ✓ managed herramientas available
  Web herramientas       ✓ active via Nous subscription
  Image gen       ✓ active via Nous subscription
  TTS             ✓ active via Nous subscription
  Browser         ○ active via Browser Use key
```

Herramientas marked "active via Nous subscription" are going through the puerta de enlace. Anything else is using your own keys.

## Eligibility

The Herramienta Puerta de enlace is a **paid-subscription** feature. Free-tier Nous accounts can use Portal for inference but don't include managed herramientas — [upgrade your plan](https://portal.nousresearch.com/manage-subscription) to unlock the puerta de enlace.

Some accounts are also entitled to a **free herramienta pool** — a small managed-herramienta allowance that covers puerta de enlace herramienta calls without a paid subscription. When a free pool is available, the puerta de enlace surfaces it and shows a setup prompt on first use, so you can opt in and start using managed herramientas right away.

## Mix and match

The puerta de enlace is per-herramienta. Turn it on for just what you want:

- **All herramientas through Nous** — easiest; one subscription, done.
- **Puerta de enlace for web + images, bring your own TTS** — keep your ElevenLabs voice, let Nous handle the rest.
- **Puerta de enlace only for things you don't have keys for** — "I already pay for Browserbase, but I don't want a Firecrawl account" works fine.

Switch any herramienta at any time via:

```bash
hermes herramientas          # Interactive picker for each herramienta category
```

Select the herramienta, pick **Nous Subscription** as the proveedor (or any direct proveedor you prefer). No config editing required. If you aren't logged into Nous Portal yet, picking **Nous Subscription** kicks off the Portal login inline — you don't need to authenticate through `hermes modelo` first.

## Using individual image modelos

Image generation defaults to FLUX 2 Klein 9B for speed. Override per-call by passing the modelo ID to the `image_generate` herramienta:

| Modelo | ID | Best for |
|---|---|---|
| FLUX 2 Klein 9B | `fal-ai/flux-2/klein/9b` | Fast, good default |
| FLUX 2 Pro | `fal-ai/flux-2-pro` | Higher fidelity FLUX |
| Z-Image Turbo | `fal-ai/z-image/turbo` | Stylized, fast |
| Nano Banana Pro | `fal-ai/nano-banana-pro` | Google Gemini 3 Pro Image |
| GPT Image 1.5 | `fal-ai/gpt-image-1.5` | OpenAI image gen, text+image |
| GPT Image 2 | `fal-ai/gpt-image-2` | OpenAI latest |
| Ideogram V3 | `fal-ai/ideogram/v3` | Strong prompt adherence + typography |
| Recraft V4 Pro | `fal-ai/recraft/v4/pro/text-to-image` | Vector-style, graphic design |
| Qwen Image | `fal-ai/qwen-image` | Alibaba multimodal |

The set evolves — `hermes herramientas` → Image Generation shows the current live list.

---

## Configuración reference

Most users never need to touch this — `hermes modelo` and `hermes herramientas` cover every workflow interactively. This section is for writing config.yaml directly or scripting setups.

### Per-herramienta `use_puerta de enlace` flag

Each herramienta's config block takes a `use_puerta de enlace` boolean:

```yaml
web:
  backend: firecrawl
  use_puerta de enlace: true

image_gen:
  use_puerta de enlace: true

tts:
  proveedor: openai
  use_puerta de enlace: true

browser:
  cloud_proveedor: browser-use
  use_puerta de enlace: true
```

Precedence: `use_puerta de enlace: true` routes through Nous regardless of any direct keys in `.env`. `use_puerta de enlace: false` (or absent) uses direct keys if available and only falls back to the puerta de enlace when none exist.

### Disabling the puerta de enlace

```yaml
web:
  use_puerta de enlace: false   # Hermes now uses FIRECRAWL_API_KEY from .env
```

`hermes herramientas` automatically clears the flag when you pick a non-puerta de enlace proveedor, so this usually happens for you.

### Self-hosted puerta de enlace (advanced)

Running your own Nous-compatible puerta de enlace? Override endpoints in `~/.hermes/.env`:

```bash
TOOL_GATEWAY_DOMAIN=your-domain.example.com
TOOL_GATEWAY_SCHEME=https
TOOL_GATEWAY_USER_TOKEN=your-token        # normally auto-populated from Portal login
FIRECRAWL_GATEWAY_URL=https://...         # override one endpoint specifically
```

These knobs exist for custom infrastructure setups (enterprise deployments, dev environments). Regular subscribers never set them.

## FAQ

### Does it work with Telegram / Discord / the other messaging puerta de enlaces?

Yes. Herramienta Puerta de enlace operates at the herramienta-execution layer, not the CLI. Every interface that can call a herramienta — CLI, Telegram, Discord, Slack, IRC, Teams, the API server, anything — benefits from it transparently.

### What happens if my subscription expires?

Herramientas routed through the puerta de enlace stop working until you renew or swap in direct API keys via `hermes herramientas`. Hermes shows a clear error pointing at the portal.

### Can I see usage or costs per herramienta?

Yes — the [Nous Portal panel](https://portal.nousresearch.com) breaks usage down by herramienta so you can see what's driving your bill.

### Is Modal (serverless terminal) included?

Modal is available as an **optional add-on** through the Nous subscription, not part of the default Herramienta Puerta de enlace bundle. Configure it via `hermes setup terminal` or directly in `config.yaml` when you want a remote sandbox for shell execution.

### Do I need to delete my existing API keys when I enable the puerta de enlace?

No — keep them in `.env`. When `use_puerta de enlace: true`, Hermes skips direct keys and uses the puerta de enlace. Flip the flag back to `false` and your keys become the source again. The puerta de enlace isn't a lock-in.

---