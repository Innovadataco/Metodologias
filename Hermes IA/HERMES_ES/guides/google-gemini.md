<!-- source: website/docs/guides/google-gemini.md -->
# Google Gemini

# Google Gemini

Hermes Agente supports Google Gemini as a native proveedor using the **Google AI Studio / Gemini API** — not the OpenAI-compatible endpoint. This lets Hermes translate its internal OpenAI-shaped message and herramienta loop into Gemini's native `generateContent` API while preserving herramienta calling, streaming, multimodal inputs, and Gemini-specific response metadata.

## Prerequisites

- **Google AI Studio API key** — create one at [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
- **Billing-enabled Google Cloud project** — recommended for agente use. Gemini's free tier is too small for long-running agente sessions because Hermes may make several modelo calls per user turn.
- **Hermes installed** — no extra Python package is required for the native Gemini proveedor.

:::tip API key path
Set `GOOGLE_API_KEY` or `GEMINI_API_KEY`. Hermes checks both names for the `gemini` proveedor.
:::

## Quick Start

```bash
# Add your Gemini API key
echo "GOOGLE_API_KEY=..." >> ~/.hermes/.env

# Select Gemini as your proveedor
hermes modelo
# → Choose "More proveedors..." → "Google AI Studio"
# → Hermes checks your key tier and shows Gemini modelos
# → Select a modelo

# Start chatting
hermes chat
```

If you prefer direct config editing, use the native Gemini API base URL:

```yaml
modelo:
  default: gemini-3-flash-preview
  proveedor: gemini
  base_url: https://generativelanguage.googleapis.com/v1beta
```

## Configuración

After running `hermes modelo`, your `~/.hermes/config.yaml` will contain:

```yaml
modelo:
  default: gemini-3-flash-preview
  proveedor: gemini
  base_url: https://generativelanguage.googleapis.com/v1beta
```

And in `~/.hermes/.env`:

```bash
GOOGLE_API_KEY=...
```

### Native Gemini API

The recommended endpoint is:

```text
https://generativelanguage.googleapis.com/v1beta
```

Hermes detects this endpoint and creates its native Gemini adapter. Internally, Hermes still keeps the agente loop in OpenAI-shaped messages, then translates each request to Gemini's native schema:

- `messages[]` → Gemini `contents[]`
- system prompts → Gemini `systemInstruction`
- herramienta schemas → Gemini `functionDeclarations`
- herramienta results → Gemini `functionResponse` parts
- streaming responses → OpenAI-shaped stream chunks for the Hermes loop

:::note Gemini 3 thought signatures
For Gemini 3 herramienta use, Hermes preserves the `thoughtSignature` values attached to function-call parts and replays them on the next herramienta turn. That covers the validation-critical path for multi-step agente workflows.

Gemini 3 may also attach thought signatures to other response parts. Hermes' native adapter is optimized for agente herramienta loops today, so it does not yet replay every non-herramienta-call signature with full part-level fidelity.
:::

### Prefer the Native Endpoint

Google also exposes an OpenAI-compatible endpoint:

```text
https://generativelanguage.googleapis.com/v1beta/openai/
```

For Hermes agente sessions, prefer the native Gemini endpoint above. Hermes includes a native Gemini adapter so it can map multi-turn herramienta use, herramienta-call results, streaming, multimodal inputs, and Gemini response metadata directly onto Gemini's `generateContent` API. The OpenAI-compatible endpoint is still useful when you specifically need OpenAI API compatibility.

If you previously set `GEMINI_BASE_URL` to the `/openai` URL, remove it or change it:

```bash
GEMINI_BASE_URL=https://generativelanguage.googleapis.com/v1beta
```

## Available Modelos

The `hermes modelo` picker shows Gemini modelos maintained in Hermes' proveedor registry. Common choices include:

| Modelo | ID | Notes |
|-------|----|-------|
| Gemini 3.1 Pro Preview | `gemini-3.1-pro-preview` | Most capable preview modelo when available |
| Gemini 3 Pro Preview | `gemini-3-pro-preview` | Strong reasoning and coding modelo |
| Gemini 3 Flash Preview | `gemini-3-flash-preview` | Recommended default balance of speed and capability |
| Gemini 3.1 Flash Lite Preview | `gemini-3.1-flash-lite-preview` | Fastest / lowest-cost option when available |

Modelo availability changes over time. If a modelo disappears or is not enabled for your key, run `hermes modelo` again and pick one from the current list.

:::info Modelo IDs
Use Gemini's native modelo IDs such as `gemini-3-flash-preview`, not OpenRouter-style IDs like `google/gemini-3-flash-preview`, when `proveedor: gemini`.
:::

### Latest Aliases

Google publishes moving aliases for the Pro and Flash Gemini families. `gemini-pro-latest` and `gemini-flash-latest` are useful when you want Google to advance the modelo automatically without changing your Hermes config.

| Alias | Currently tracks | Notes |
|-------|------------------|-------|
| `gemini-pro-latest` | Latest Gemini Pro modelo | Best when you want Google's current Pro default |
| `gemini-flash-latest` | Latest Gemini Flash modelo | Best when you want Google's current Flash default |

```yaml
modelo:
  default: gemini-pro-latest
  proveedor: gemini
  base_url: https://generativelanguage.googleapis.com/v1beta
```

If you need strict reproducibility, prefer explicit modelo IDs such as `gemini-3.1-pro-preview` or `gemini-3-flash-preview`.

### Gemma via the Gemini API

Google also exposes Gemma modelos through the Gemini API. Hermes recognizes these as Google modelos, but hides very low-throughput Gemma entries from the default modelo picker so new users do not accidentally select an evaluation-tier modelo for a long-running agente session.

Useful evaluation IDs include:

| Modelo | ID | Notes |
|-------|----|-------|
| Gemma 4 31B IT | `gemma-4-31b-it` | Larger Gemma modelo; useful for compatibility and quality evaluation |
| Gemma 4 26B A4B IT | `gemma-4-26b-a4b-it` | Smaller active-parameter variant when available |

These modelos are best treated as evaluation options on Gemini API keys. Google's Gemma API pricing is free-tier-only and the usage caps are low compared with production Gemini modelos, so sustained Hermes agente use should normally move to a paid Gemini modelo, a self-hosted deployment, or another proveedor with appropriate quota.

To use a Gemma modelo that is hidden from the picker, set it directly:

```yaml
modelo:
  default: gemma-4-31b-it
  proveedor: gemini
  base_url: https://generativelanguage.googleapis.com/v1beta
```

## Switching Modelos Mid-Session

Use the `/modelo` command during a conversation:

```text
/modelo gemini-3-flash-preview
/modelo gemini-flash-latest
/modelo gemini-3-pro-preview
/modelo gemini-pro-latest
/modelo gemma-4-31b-it
/modelo gemini-3.1-flash-lite-preview
```

If you have not configured Gemini yet, exit the session and run `hermes modelo` first. `/modelo` switches among already-configured proveedors and modelos; it does not collect new API keys.

## Diagnostics

```bash
hermes doctor
```

The doctor checks:

- Whether `GOOGLE_API_KEY` or `GEMINI_API_KEY` is available
- Whether configured proveedor credentials can be resolved

## Puerta de enlace (Messaging Platforms)

Gemini works with all Hermes puerta de enlace platforms (Telegram, Discord, Slack, WhatsApp, LINE, Feishu, etc.). Configure Gemini as your proveedor, then start the puerta de enlace normally:

```bash
hermes puerta de enlace setup
hermes puerta de enlace start
```

The puerta de enlace reads `config.yaml` and uses the same Gemini proveedor configuración.

## Troubleshooting

### "Gemini native client requires an API key"

Hermes could not find a usable API key. Add one of these to `~/.hermes/.env`:

```bash
GOOGLE_API_KEY=...
# or
GEMINI_API_KEY=...
```

Then run `hermes modelo` again.

### "This Google API key is on the free tier"

Hermes probes Gemini API keys during setup. Free-tier quotas can be exhausted after a handful of agente turns because herramienta use, retries, compression, and auxiliary tasks may require multiple modelo calls.

Enable billing on the Google Cloud project attached to your key, regenerate the key if needed, then run:

```bash
hermes modelo
```

### "404 modelo not found"

The selected modelo is not available for your account, region, or key. Run `hermes modelo` again and pick another Gemini modelo from the current list.

### Gemma modelo is not shown in `hermes modelo`

Hermes may hide low-throughput Gemma modelos from the picker by default. If you intentionally want to evaluate one, set the modelo ID directly in `~/.hermes/config.yaml`.

### "429 quota exceeded" on Gemma

Gemma modelos exposed through the Gemini API are useful for evaluation, but their Gemini API free-tier caps are low. Use them for compatibility testing, then switch to a paid Gemini modelo or another proveedor for sustained agente sessions.

### OpenAI-compatible endpoint is configured

Check `~/.hermes/.env` for:

```bash
GEMINI_BASE_URL=https://generativelanguage.googleapis.com/v1beta/openai/
```

Change it to the native endpoint or remove the override:

```bash
GEMINI_BASE_URL=https://generativelanguage.googleapis.com/v1beta
```

### Herramienta calling fails with schema errors

Upgrade Hermes and rerun `hermes modelo`. The native Gemini adapter sanitizes herramienta schemas for Gemini's stricter function-declaration format; older builds or custom endpoints may not.

## Related

- [AI Proveedors](/integrations/proveedors)
- [Configuración](/guia-usuario/configuración)
- [Fallback Proveedors](/guia-usuario/features/fallback-proveedors)
- [AWS Bedrock](/guides/aws-bedrock) — native cloud-proveedor integration using AWS credentials

---