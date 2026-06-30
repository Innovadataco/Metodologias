<!-- source: website/docs/guia-usuario/features/computer-use.md -->
# guia-usuario/features/computer-use.md

# Computer Use

Hermes Agente can drive your desktop — clicking, typing, scrolling,
dragging — in the **background** on **macOS, Windows, and Linux**. Your
cursor doesn't move, keyboard focus doesn't change, and your virtual
desktops / Spaces don't switch on you. You and the agente co-work on the
same machine.

Unlike most computer-use integrations, this works with **any herramienta-capable
modelo** — Claude, GPT, Gemini, or an open modelo on a local
OpenAI-compatible endpoint. There's no Anthropic-native schema to worry
about.

## How it works

The `computer_use` herramientaset speaks MCP over stdio to
[`cua-driver`](https://github.com/trycua/cua), an open-source background
computer-use driver. Each platform uses the appropriate accessibility +
input stack under the hood:

| Platform | Accessibility tree | Input dispatch |
|---|---|---|
| macOS | AX (private SkyLight SPIs) | `SLPSPostEventRecordTo` — pid-scoped, no cursor warp |
| Windows | UIAutomation | `SendInput` + `PostMessage` — no focus steal |
| Linux | AT-SPI (X11 + Wayland) | XTest (X11) / virtual-keyboard (Wayland) |

The result is the same on every platform: the agente can read the
accessibility tree of any visible window AND post synthesized events
without bringing it to front, switching virtual desktops, or moving the
real OS cursor.

For the underlying contract — *why* background mode matters, the
no-foreground invariant, click-dispatch internals — see
**[cua.ai/docs/explanation/the-no-foreground-contract](https://cua.ai/docs/explanation/the-no-foreground-contract)**.

## Enabling

Pick whichever path is most convenient — both run the same upstream
installer:

**Option 1: dedicated CLI command (most direct).**

```
hermes computer-use install
```

This fetches and runs the upstream cua-driver installer — `install.sh`
on macOS/Linux, `install.ps1` on Windows. Use `hermes computer-use
status` to verify the install.

**Option 2: enable the herramientaset interactively.**

1. Run `hermes herramientas`, pick `🖱️  Computer Use (macOS/Windows/Linux)`.
2. The setup runs the upstream installer (same as Option 1).

After installing, regardless of which path you took, grant the
platform-appropriate prereqs:

| Platform | Prereqs |
|---|---|
| **macOS** | System Settings → Privacy & Security → **Accessibility** + **Screen Recording** → allow your terminal (or Hermes app). `hermes computer-use doctor` will tell you which permission is missing. |
| **Windows** | None at install time. If you're driving over SSH (not RDP / console), you need the autostart pattern — see [cua.ai/docs/how-to-guides/driver/windows-ssh](https://cua.ai/docs/how-to-guides/driver/windows-ssh) for the Session 0 ↔ Session 1+ proxy. |
| **Linux** | A reachable display server: `DISPLAY` set for X11, or `XDG_SESSION_TYPE=wayland`. Wayland sessions need an XWayland bridge for capture. AT-SPI must be on (default on GNOME/KDE/Xfce). |

Then start a session with the herramientaset enabled:

```
hermes -t computer_use chat
```

or add `computer_use` to your enabled herramientasets in `~/.hermes/config.yaml`.

## `hermes computer-use doctor` — your first triage stop

`hermes computer-use doctor` runs cua-driver's structured
`health_report` MCP herramienta and prints a per-check matrix. It's the single
fastest way to find out *why* an action isn't working.

```
$ hermes computer-use doctor
⚠️  cua-driver 0.5.8 on darwin — degraded
  ✅ binary_version: cua-driver 0.5.8
  ✅ platform_supported: macOS 26.4.1 (arm64)
  ✅ session_active: MCP session is active.
  ❌ bundle_identity: Process has no CFBundleIdentifier.
      → Run the binary inside CuaDriver.app so TCC grants attribute correctly.
  ✅ tcc_accessibility: Accessibility is granted.
  ✅ tcc_screen_recording: Screen Recording is granted.
  ✅ ax_capability: AX is trusted and reachable.
  ✅ screen_capture_capability: ScreenCaptureKit reachable; 1 display(s) shareable.
```

- **Exit code 0** when overall is `ok` — everything's wired up.
- **Exit code 1** when `degraded` or `failed` — at least one check failed; the hint on each failure tells you what to fix.
- **Exit code 2** when the cua-driver binary itself isn't reachable.

Useful flags:

- `--include CHECK` — run only the listed checks (repeat for multiple)
- `--skip CHECK` — skip a check (wins over `--include`)
- `--json` — emit the raw structured payload, same shape as the
  `herramientas/call health_report` MCP response

The check matrix is platform-aware: `bundle_identity` / `tcc_*` are
`skip` on Windows + Linux because those concepts don't apply.
`ax_capability` checks AX on macOS, UIA on Windows, AT-SPI on Linux —
each with the right diagnostic hint when it can't reach.

## The agente cursor and sessions

When the agente acts, you'll see a **tinted overlay cursor** glide
across the screen to where each click / type / scroll lands. The real
OS cursor never moves — the overlay is a visual cue that says "the
agente is acting here." Each Hermes run declares its own cua-driver
**session id** (something like `hermes-3a7b9c14d2e8`); the cursor's
identity is keyed to that session, so concurrent runs / subagentes each
get their own cursor without stepping on each other.

Tune the cursor with `cua-driver`'s CLI flags or the runtime
`set_agente_cursor_style` MCP herramienta — see
[cua.ai/docs/how-to-guides/driver/personalize-cursor](https://cua.ai/docs/how-to-guides/driver/personalize-cursor)
for the full menu (built-in `arrow` vs `teardrop` silhouette, custom
SVG / PNG / ICO via `--cursor-icon`, runtime gradient colors, bloom
halo).

## Going deeper — the cua-driver habilidad pack

Hermes intentionally keeps its habilidad (`habilidads/computer-use/SKILL.md`)
focused on the Hermes-side `computer_use` action vocabulary — the
single source of truth the agente loads. For the deeper material —
platform-specific deep dives, recording semantics, browser page
interaction — point your agente harness at the cua-driver habilidad pack
the cua-driver team ships and maintains directly:

```
cua-driver habilidads install
```

This symlinks the pack into your agente harness' habilidad directory. After
running it, an agente gets access to:

| File | Topic |
|---|---|
| `SKILL.md` | The cross-platform core (snapshot invariant, no-foreground contract, click dispatch, AX-tree mechanics) |
| `MACOS.md` | macOS specifics: no-foreground contract, AXMenuBar navigation, SkyLight click dispatch, Apple Events JS bridge |
| `WINDOWS.md` | Windows specifics: UIA tree, UWP / `ApplicationFrameHost` hosting, Session 0 isolation, autostart pattern |
| `LINUX.md` | Linux specifics: AT-SPI tree, X11 / Wayland, terminal-emulator detection |
| `RECORDING.md` | Trajectory + video recording semantics |
| `WEB_APPS.md` | Browser-page interaction tips |
| `TESTS.md` | Replay-by-trajectory workflow |

These are **platform deep dives, not duplicates of the Hermes habilidad** —
when an agente reports "on Windows, my click landed on the wrong
element," it reads `WINDOWS.md` for the UIA / UWP contexto that
explains why and what to do differently.

`cua-driver habilidads status` shows what's installed and which agente
harnesses it's linked into. Today the autodetect list covers Claude
Code, Codex, OpenCode, OpenClaw, and Antigravity; **Hermes
autodetection is planned as a follow-up in `trycua/cua`** — until
then, run `cua-driver habilidads install` once and point your harness at
the resulting `~/.cua-driver/habilidads/cua-driver` directory (or symlink
it into your usual habilidad space).

## Quick example

User prompt: *"Find my latest email from Stripe and summarise what they want me to do."*

The agente's plan (this is the same shape on macOS / Windows / Linux —
the modelo substitutes the platform's idiomatic shortcut and app name):

1. `computer_use(action="capture", mode="som", app="Mail")` — gets a
   screenshot of the email app with every sidebar item, herramientabar button,
   and message row numbered.
2. `computer_use(action="click", element=14)` — clicks the search field.
3. `computer_use(action="type", text="from:stripe")`
4. `computer_use(action="key", keys="return", capture_after=True)` —
   submit and get the new screenshot.
5. Click the top result, read the body, summarise.

During all of this, your cursor stays wherever you left it and the email
app never comes to front.

## Proveedor compatibility

| Proveedor | Vision? | Works? | Notes |
|---|---|---|---|
| Anthropic (Claude Sonnet/Opus 3+) | ✅ | ✅ | Best overall; SOM + raw coordinates. |
| OpenRouter (any vision modelo) | ✅ | ✅ | Multi-part herramienta messages supported. |
| OpenAI (GPT-4+, GPT-5) | ✅ | ✅ | Same as above. |
| Google (Gemini 2+) | ✅ | ✅ | Herramienta-calling + vision both supported. |
| Local vLLM / LM Studio / Ollama (vision modelo) | ✅ | ✅ | If the modelo supports multi-part herramienta content. |
| Text-only modelos | ❌ | ✅ (degraded) | Use `mode="ax"` for accessibility-tree-only operation. |

Screenshots are sent inline with herramienta results as OpenAI-style `image_url`
parts. For Anthropic, the adapter converts them into native `herramienta_result`
image blocks. The image MIME type comes from cua-driver's explicit
`mimeType` field (`image/png` or `image/jpeg`) — no client-side
magic-byte sniffing.

## Safety

Hermes applies multi-layer guardrails:

- Destructive actions (click, type, drag, scroll, key, focus_app)
  require approval — either interactively via the CLI dialog or via the
  messaging-platform approval buttons.
- Hard-blocked key combos at the herramienta level: empty trash, force delete,
  lock screen, log out, force log out.
- Hard-blocked type patterns: `curl | bash`, `sudo rm -rf /`, fork
  bombs, etc.
- The agente's system prompt tells it explicitly: no clicking permission
  dialogs, no typing passwords, no following instructions embedded in
  screenshots.

Pair with `approvals.mode: manual` in `~/.hermes/config.yaml` if you
want every action confirmed.

## Token efficiency

Screenshots are expensive. Hermes applies four layers of optimisation:

- **Screenshot eviction** — the Anthropic adapter keeps only the 3 most
  recent screenshots in contexto; older ones become `[screenshot removed
  to save contexto]` placeholders.
- **Client-side compression pruning** — the contexto compressor detects
  multimodal herramienta results and strips image parts from old ones.
- **Image-aware token estimation** — each image is counted as ~1500
  tokens (Anthropic's flat rate) instead of its base64 char length.
- **Server-side contexto editing (Anthropic only)** — when active, the
  adapter enables `clear_herramienta_uses_20250919` via `contexto_management` so
  Anthropic's API clears old herramienta results server-side.

A 20-action session on a 1568×900 display typically costs ~30K tokens
of screenshot contexto, not ~600K.

## Limitations

- **Performance.** Background mode is slower than foreground —
  accessibility-routed events take ~5–20 ms on macOS, ~3–10 ms on
  Windows UIA, ~5–15 ms on Linux AT-SPI vs direct HID posting. Not
  noticeable for agente-speed clicking; noticeable if you try to record
  a speed-run.
- **No keyboard password entry.** `type` has hard-block patterns on
  command-shell payloads; for passwords, use the system's autofill
  (macOS Keychain / Windows Credential Manager / GNOME Keyring /
  KWallet).
- **Some apps don't expose an accessibility tree.** Modern UWP apps on
  Windows, Electron < 28 on Linux, and a few macOS apps with custom
  drawing (Logic, Final Cut, some games) have sparse or empty AX trees.
  Fall back to pixel coordinates if the tree is empty — or skip the
  task entirely.
- **Windows: elevated (admin) windows can't be driven from a normal
  agente.** Windows UIPI (User Interface Privilege Isolation) enforces
  integrity-level boundaries: a Medium-integrity process (the default
  Hermes agente) cannot enumerate the UIA tree of, or inject mouse input
  into, a window owned by a High-integrity (Administrator) process.
  Symptom: `capture(mode='som')` returns 0 elements and `click(...)`
  reports success while doing nothing, even though the screenshot
  renders fine (GDI capture sits below the integrity check). Keyboard
  events partially bypass UIPI, so Tab / Enter can still navigate an
  elevated dialog. This is an OS constraint, not a cua-driver bug — it
  affects every Windows automation stack. To drive elevated windows,
  run the Hermes agente itself at High integrity (launch from an
  elevated terminal); otherwise target non-elevated windows.
- **Platform-specific deployment gotchas:**
  - **macOS** uses private SkyLight SPIs. Apple can change them in any
    OS update. Hermes warns when the installed cua-driver is older than
    the version it was tested against.
  - **Windows** SSH sessions run in **Session 0**, which has no
    interactive desktop. Drive Hermes from inside the RDP / console
    session, or set up cua-driver's autostart Scheduled Task —
    [windows-ssh](https://cua.ai/docs/how-to-guides/driver/windows-ssh)
    has the recipe.
  - **Linux** requires a reachable display server. Headless servers
    need Xvfb (`Xvfb :99 -screen 0 1920x1080x24`) before
    `computer_use` can capture or inject events. Pure Wayland sessions
    need an XWayland bridge for screen capture (cua-driver's Wayland
    inject path handles input independently).

For cross-platform GUI automation without the desktop overhead (and
without TCC / Session 0 / X11 setup), the `browser` herramientaset uses a
real headless Chromium and is the right answer for web-only tasks.

## Configuración

Override the driver binary path (tests / CI / local builds):

```
HERMES_CUA_DRIVER_CMD=/path/to/your/cua-driver
```

Swap the backend entirely (for testing):

```
HERMES_COMPUTER_USE_BACKEND=noop   # records calls, no side effects
```

### Telemetry

cua-driver ships with anonymous usage telemetry (PostHog) enabled by default
upstream. **Hermes disables it for you** — on every cua-driver invocation
(the MCP backend, `status`, `doctor`, and install) Hermes sets
`CUA_DRIVER_RS_TELEMETRY_ENABLED=0` in the driver's environment.

To opt back in (let cua-driver use its own default and send telemetry), set
this in `config.yaml`:

```yaml
computer_use:
  cua_telemetry: true   # default: false (telemetry off)
```

When it's on, `hermes computer-use doctor` reports `telemetry: enabled`;
when off (the default), it reports `telemetry: disabled via
CUA_DRIVER_RS_TELEMETRY_ENABLED`.

## Testing against a local cua-driver build

When you're developing cua-driver itself — or want to test an
unreleased fix — point Hermes at a binary you built from source instead
of the published release. Hermes resolves the driver with
`shutil.which("cua-driver")` and **does not enforce
`HERMES_CUA_DRIVER_VERSION`**, so a local build (reported as
`0.0.0-local-*`) is accepted as-is. Two approaches:

### Option A — `install-local` (build + put it on PATH)

From your `trycua/cua` checkout, run the upstream local installer. It
builds the Rust backend in release mode and drops `cua-driver` into the
same install layout the production installer uses, adding its bin dir
to your PATH:

```powershell
# Windows (PowerShell), from the cua repo root
./libs/cua-driver/scripts/install-local.ps1 -NoAutoStart
```

```bash
# macOS / Linux, from the cua repo root  (defaults to a debug build without --release)
./libs/cua-driver/scripts/install-local.sh --release
```

- Windows stages the build under `%USERPROFILE%\.cua-driver\packages\…`
  and junctions
  `%LOCALAPPDATA%\Programs\Cua\cua-driver\bin` (added to your User
  PATH) to it. macOS/Linux symlinks `cua-driver` into `~/.local/bin`
  (override with `--bin-dir <path>`).
- `-NoAutoStart` skips registering the `cua-driver-serve` logon daemon
  — you don't need it for Hermes testing (see notes).

Then open a fresh shell (so the PATH change is visible) and confirm:

```
cua-driver --version                 # local builds report 0.0.0-local-release
# Windows:      (Get-Command cua-driver).Source
# macOS/Linux:  which cua-driver
```

### Option B — point Hermes straight at the built binary (fastest loop)

Skip the install ceremony entirely: `cargo build` and set
`HERMES_CUA_DRIVER_CMD` to the resulting binary. Best for rapid
edit/build/test.

```bash
cargo build -p cua-driver            # add --release for a release build; run from libs/cua-driver/rust
```

```
# Windows (.env)
HERMES_CUA_DRIVER_CMD=C:\path\to\cua\libs\cua-driver\rust\target\debug\cua-driver.exe
# macOS / Linux (.env)
HERMES_CUA_DRIVER_CMD=/path/to/cua/libs/cua-driver/rust/target/debug/cua-driver
```

### Confirm Hermes is using your build

- `hermes computer-use status` prints the resolved binary path and
  version.
- `hermes computer-use doctor` confirms the binary is reachable and
  exercises the full MCP path end-to-end.
- In a session, `computer_use(action="capture")` exercises the spawned
  `cua-driver mcp` child process.

### Notes & gotchas

- **Hermes spawns its own `cua-driver mcp` child over stdio** — it does
  *not* attach to the long-running `cua-driver serve` autostart daemon
  or its named pipe. So the scheduled task / LaunchAgente is unnecessary
  for testing (`-NoAutoStart` is fine). The autostart daemon and the
  Windows UIAccess worker (`cua-driver-uia.exe`) only matter for
  foreground-safe input on some apps (e.g. WPF); the standard herramienta
  surface works through the stdio child. On Windows SSH sessions, the
  autostart pattern IS needed — see the Limitations section.
- **Locked binary on Windows.** A running `cua-driver-serve` daemon can
  hold `cua-driver.exe` and block an overwrite on rebuild.
  `install-local.ps1` renames the locked binary out of the way
  automatically; if you `cargo build` manually (Option B), stop it
  first with `cua-driver autostart disable` (or `schtasks /End /TN
  cua-driver-serve`).
- **Rebuild loop.** After editing cua-driver source, re-run
  `install-local` (rebuilds, restages, flips the `current` junction)
  for Option A, or just re-`cargo build` for Option B — no Hermes
  change needed either way.
- **Local builds skip the version check.** Hermes warns when the
  installed cua-driver is older than its per-OS tested baseline, but
  exempts `0.0.0-local-*` dev builds — so your local build never
  triggers that warning.

## Troubleshooting

**First action when anything's off: run `hermes computer-use doctor`.**
The structured per-check matrix tells you (and any agente helping you
debug) exactly what's wrong.

Specific failure modes the doctor doesn't catch:

**`computer_use backend unavailable: cua-driver is not installed`** —
Run `hermes computer-use install` to fetch the cua-driver binary, or
run `hermes herramientas` and enable the Computer Use herramientaset.

**Clicks seem to have no effect** — Capture and verify. A modal you
didn't see may be blocking input. Dismiss it with `escape` or the close
button.

**Element indices are stale** — SOM indices are only valid until the
next `capture`. Re-capture after any state-changing action. The
wrapper carries opaque `element_token`s for stale detection — you'll
see an explicit error rather than a wrong click.

**"blocked pattern in type text"** — The text you tried to `type`
matches the dangerous-shell-pattern list. Break the command up or
reconsider.

**Empty captures on Linux** — `DISPLAY` not set, or you're on pure
Wayland without an XWayland bridge. `hermes computer-use doctor` will
flag this as `ax_capability: fail` with a `Set DISPLAY (X11)…` hint.

**Empty captures on Windows over SSH** — You're in Session 0 (the
services session). Drive from RDP / console directly, or set up the
autostart pattern — see
[cua.ai/docs/how-to-guides/driver/windows-ssh](https://cua.ai/docs/how-to-guides/driver/windows-ssh).

## See also

- **Hermes-side habilidad** — `habilidads/computer-use/SKILL.md` — teaches the
  Hermes `computer_use` action vocabulary; this is what the agente loads.
- **cua-driver habilidad pack** — for platform-specific deep dives
  (macOS no-foreground contract, Windows UIA + Session 0, Linux AT-SPI
  + X11/Wayland, recording, browser pages), run
  `cua-driver habilidads install` and read `MACOS.md` / `WINDOWS.md` /
  `LINUX.md` / `RECORDING.md` / `WEB_APPS.md`. Once `cua-driver habilidads
  install` autodetects Hermes (planned follow-up), this happens
  automatically on install.
- **cua.ai/docs** — the cua-driver project's documentation:
  - [What is computer use?](https://cua.ai/docs/explanation/what-is-computer-use) — concept intro
  - [The no-foreground contract](https://cua.ai/docs/explanation/the-no-foreground-contract) — *why* background mode matters
  - [Install reference](https://cua.ai/docs/how-to-guides/driver/install) — cross-platform install details
  - [Personalize the agente cursor](https://cua.ai/docs/how-to-guides/driver/personalize-cursor) — built-in shapes, custom assets, runtime overrides
  - [Drive Windows over SSH](https://cua.ai/docs/how-to-guides/driver/windows-ssh) — the Session 0 → Session 1+ autostart pattern
  - [Keep cua-driver running](https://cua.ai/docs/how-to-guides/driver/keep-running) — autostart / daemon lifecycle
  - [Connect your agente](https://cua.ai/docs/how-to-guides/driver/connect-your-agente) — register cua-driver with various harnesses (Hermes among them)
- [cua-driver source (trycua/cua)](https://github.com/trycua/cua)
- [Browser automation](./browser.md) for cross-platform web tasks where you don't need to drive native apps.

---