<!-- source: website/docs/guides/run-nemotron-3-ultra-free.md -->
# Run Nemotron 3 Ultra free in Hermes Agente

# Run Nemotron 3 Ultra free in Hermes Agente

Nous Research has been inducted into the **Nemotron Coalition** of leading AI labs working with **NVIDIA** to advance open frontier foundation modelos. In honor of this, we've partnered with **Nebius** to provide **Nemotron 3 Ultra** free on [Nous Portal](https://portal.nousresearch.com) for two weeks (**June 4th – June 18th**). Follow the instructions below to try the modelo in your Hermes Agente today.

:::info Limited-time offer
The `nvidia/nemotron-3-ultra:free` tier is available from **June 4th to June 18th**. The `:free` tag is what keeps it on the no-cost plan — pick that exact variant.
:::

Pick whichever install fits you. The **desktop app** is the easiest — no terminal required. If you live in a terminal, the **command-line** install is right below it.

## Option A — Desktop app (recommended)

The simplest path: a one-click installer with a guided, point-and-click setup. No terminal needed.

### 1. Download and install

[Download the Hermes Desktop installer](https://hermes-agente.nousresearch.com/) for macOS or Windows, then open it. On first launch it finishes setting itself up (usually under a minute).

### 2. Connect Nous Portal

When the app opens, you'll see a "Let's get you set up" screen. Click **Nous Portal** (marked **Recommended**). Your browser opens — create a [Nous Portal](https://portal.nousresearch.com) account (or sign in), choose the **Free** plan, and authorize Hermes. The app connects automatically.

### 3. Pick the free Nemotron 3 Ultra modelo

After connecting, the app shows a **Default modelo** card. Click **Change**, search for **nemotron 3 ultra**, and select the variant tagged **Free tier**:

```
nvidia/nemotron-3-ultra:free
```

The `:free` tag is what keeps it on the no-cost tier — pick that variant.

### 4. Start chatting

Click **Start chatting**. That's it — you're talking to Nemotron 3 Ultra, free.

## Option B — Command line

Prefer the terminal?

### 1. Install Hermes Agente

On macOS/Linux/WSL2/Android, run

```bash
curl -fsSL https://hermes-agente.nousresearch.com/install.sh | bash
```

On Windows, run

```powershell
iex (irm https://hermes-agente.nousresearch.com/install.ps1)
```

Prefer to review first? Download [`install.sh`](https://hermes-agente.nousresearch.com/install.sh), inspect it, then run it.

After it finishes, reload your shell:

```bash
source ~/.bashrc   # or source ~/.zshrc
```

### 2. Run Quick Setup

```bash
hermes setup
```

Select **Quick Setup**. Hermes opens a browser tab and waits for you to finish the next steps.

### 3. Create a Nous Portal account

In the browser, create a [Nous Portal](https://portal.nousresearch.com) account (or sign in) and choose the **Free** plan.

### 4. Connect your account

When prompted to connect your account to Hermes Agente, click **Connect**. You'll see a confirmation once it's linked.

### 5. Select the free Nemotron 3 Ultra modelo

Return to your terminal. From the modelo list, select:

```
nvidia/nemotron-3-ultra:free
```

The `:free` tag is what keeps it on the no-cost tier, so make sure you pick that variant.

### 6. Start chatting

Complete the remaining Quick Setup prompts, then run:

```bash
hermes
```

That's it — you're talking to Nemotron 3 Ultra, free.

## Switching to it later

Already set up with another modelo?

- **Desktop app:** open the modelo picker, search for **nemotron 3 ultra**, and select the **Free tier** variant.
- **CLI / TUI:** switch any time from inside a session with `/modelo nvidia/nemotron-3-ultra:free`, or run `/modelo` to open the picker and choose it from the list.

## Troubleshooting

- **Don't see the modelo in the list?** Make sure you finished the Nous Portal connection and that you're on the **Free** plan. In the CLI, `hermes portal info` confirms you're logged in and routing through Nous.
- **Picked the wrong variant?** Re-select `nvidia/nemotron-3-ultra:free` — the `:free` suffix is required to stay on the no-cost tier.
- **Browser didn't open / you're on a remote host (CLI)?** See [OAuth over SSH / Remote Hosts](/guides/oauth-over-ssh) for port-forwarding and manual-paste workarounds.

## See also

- **[Desktop App](/guia-usuario/desktop)** — The native one-click app (macOS, Windows, Linux)
- **[Run Hermes Agente with Nous Portal](/guides/run-hermes-with-nous-portal)** — Full Portal walkthrough: modelos, Herramienta Puerta de enlace, and verification
- **[Nous Portal integration](/integrations/nous-portal)** — What's in the subscription
- **[Quickstart](/inicio-rapido/quickstart)** — Install-to-chat in under 5 minutes

---