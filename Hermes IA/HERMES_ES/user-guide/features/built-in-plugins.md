<!-- source: website/docs/guia-usuario/features/built-in-complementos.md -->
# Built-in Complementos

# Built-in Complementos

Hermes ships a small set of complementos bundled with the repository. They live under `<repo>/complementos/<name>/` and load automatically alongside user-installed complementos in `~/.hermes/complementos/`. They use the same complemento surface as third-party complementos — hooks, herramientas, slash commands — just maintained in-tree.

See the [Complementos](/guia-usuario/features/complementos) page for the general complemento system, and [Build a Hermes Complemento](/guides/build-a-hermes-complemento) to write your own.

## How discovery works

The `ComplementoManager` scans four sources, in order:

1. **Bundled** — `<repo>/complementos/<name>/` (what this page documents)
2. **User** — `~/.hermes/complementos/<name>/`
3. **Project** — `./.hermes/complementos/<name>/` (requires `HERMES_ENABLE_PROJECT_PLUGINS=1`)
4. **Pip entry points** — `hermes_agente.complementos`

On name collision, later sources win — a user complemento named `disk-cleanup` would replace the bundled one.

`complementos/memoria/` and `complementos/contexto_engine/` are deliberately excluded from bundled scanning. Those directories use their own discovery paths because memoria proveedors and contexto engines are single-select proveedors configured through `hermes memoria setup` / `contexto.engine` in config.

## Bundled complementos are opt-in

Bundled complementos ship disabled. Discovery finds them (they appear in `hermes complementos list` and the interactive `hermes complementos` UI), but none load until you explicitly enable them:

```bash
hermes complementos enable disk-cleanup
```

Or via `~/.hermes/config.yaml`:

```yaml
complementos:
  enabled:
    - disk-cleanup
```

This is the same mechanism user-installed complementos use. Bundled complementos are never auto-enabled — not on fresh install, not for existing users upgrading to a newer Hermes. You always opt in explicitly.

To turn a bundled complemento off again:

```bash
hermes complementos disable disk-cleanup
# or: remove it from complementos.enabled in config.yaml
```

## Currently shipped

The repo ships these bundled complementos under `complementos/`. All are opt-in — enable them via `hermes complementos enable <name>`.

| Complemento | Kind | Purpose |
|---|---|---|
| `disk-cleanup` | hooks + slash command | Auto-track ephemeral files and clean them on session end |
| `security-guidance` | hooks | Pattern-match dangerous code on `write_file`/`patch` and append a security warning (or block) — 25 rules (Apache-2.0 fork of Anthropic's `claude-complementos-official` patterns) |
| `observability/langfuse` | hooks | Trace turns / LLM calls / herramientas to [Langfuse](https://langfuse.com) |
| `observability/nemo_relay` | hooks | Relay observability events (turns / LLM calls / herramientas) to an NVIDIA NeMo endpoint |
| `teams_pipeline` | standalone | Microsoft Teams meeting pipeline — Graph-backed, transcript-first meeting summaries |
| `spotify` | backend (7 herramientas) | Native Spotify playback, queue, search, playlists, albums, library |
| `google_meet` | standalone | Join Meet calls, live-caption transcription, optional realtime duplex audio |
| `image_gen/openai` | image backend | OpenAI `gpt-image-2` image generation backend (alternative to FAL) |
| `image_gen/openai-codex` | image backend | OpenAI image generation via Codex OAuth |
| `image_gen/xai` | image backend | xAI `grok-2-image` backend |
| `hermes-achievements` | panel tab | Steam-style collectible badges generated from your real Hermes session history |
| `kanban/panel` | panel tab | Kanban board UI for the multi-agente dispatcher — tasks, comments, fan-out, board switching. See [Kanban Multi-Agente](./kanban.md). |

Memoria proveedors (`complementos/memoria/*`) and contexto engines (`complementos/contexto_engine/*`) are listed separately on [Memoria Proveedors](./memoria-proveedors.md) — they're managed through `hermes memoria` and `hermes complementos` respectively. The full per-complemento detail for the two long-running hooks-based complementos follows.

### disk-cleanup

Auto-tracks and removes ephemeral files created during sessions — test scripts, temp outputs, cron logs, stale chrome perfils — without requiring the agente to remember to call a herramienta.

**How it works:**

| Hook | Behaviour |
|---|---|
| `post_herramienta_call` | When `write_file` / `terminal` / `patch` creates a file matching `test_*`, `tmp_*`, or `*.test.*` inside `HERMES_HOME` or `/tmp/hermes-*`, track it silently as `test` / `temp` / `cron-output`. |
| `on_session_end` | If any test files were auto-tracked during the turn, run the safe `quick` cleanup and log a one-line summary. Stays silent otherwise. |

**Deletion rules:**

| Category | Threshold | Confirmation |
|---|---|---|
| `test` | every session end | Never |
| `temp` | >7 days since tracked | Never |
| `cron-output` | >14 days since tracked | Never |
| empty dirs under HERMES_HOME | always | Never |
| `research` | >30 days, beyond 10 newest | Always (deep only) |
| `chrome-perfil` | >14 days since tracked | Always (deep only) |
| files >500 MB | never auto | Always (deep only) |

**Slash command** — `/disk-cleanup` available in both CLI and puerta de enlace sessions:

```
/disk-cleanup status                     # breakdown + top-10 largest
/disk-cleanup dry-run                    # preview without deleting
/disk-cleanup quick                      # run safe cleanup now
/disk-cleanup deep                       # quick + list items needing confirmation
/disk-cleanup track <path> <category>    # manual tracking
/disk-cleanup forget <path>              # stop tracking (does not delete)
```

**State** — everything lives at `$HERMES_HOME/disk-cleanup/`:

| File | Contents |
|---|---|
| `tracked.json` | Tracked paths with category, size, and timestamp |
| `tracked.json.bak` | Atomic-write backup of the above |
| `cleanup.log` | Append-only audit trail of every track / skip / reject / delete |

**Safety** — cleanup only ever touches paths under `HERMES_HOME` or `/tmp/hermes-*`. Windows mounts (`/mnt/c/...`) are rejected. Well-known top-level state dirs (`logs/`, `memories/`, `sessions/`, `cron/`, `cache/`, `habilidads/`, `complementos/`, `disk-cleanup/` itself) are never removed even when empty — a fresh install does not get gutted on first session end.

**Enabling:** `hermes complementos enable disk-cleanup` (or check the box in `hermes complementos`).

**Disabling again:** `hermes complementos disable disk-cleanup`.

### security-guidance

Fast pattern-matched security warnings on file writes. When the agente's `write_file` / `patch` / `habilidad_manage` calls carry content matching a known-dangerous code pattern — `pickle.load`, `yaml.load` without `SafeLoader`, `eval(`, `os.system`, `subprocess(...,  shell=True)`, JS `child_process.exec`, React `dangerouslySetInnerHTML`, raw `.innerHTML =` / `.outerHTML =` / `document.write`, Node `crypto.createCipher`, AES ECB mode, TLS verification disabled, XXE-prone `xml.etree` / `minidom` parsers, `<script src="//..." >` without SRI, `torch.load` without `weights_only=True`, GitHub Actions `${{ github.event.* }}` injection — the complemento appends a `⚠️ Security guidance` block to the herramienta's result.

The file is still written. The modelo reads the warning in the next turn's herramienta message and can either fix the code or document why the construct is safe in this contexto. Pattern matching has a non-trivial false-positive rate, which is why warn (not block) is the default.

**Coverage:** 25 rules total, covering unsafe deserialization, command injection, XSS sinks, crypto footguns, XXE, supply-chain (SRI), and CI/CD workflow injection. The pattern data is a verbatim Apache-2.0 fork of [Anthropic's `claude-complementos-official`](https://github.com/anthropics/claude-complementos-official/tree/main/complementos/security-guidance/hooks) — see the complemento's `LICENSE` and `NOTICE` files for attribution.

**Modes:**

| Env var | Effect |
|---|---|
| (unset) | **warn mode** (default) — file is written, warning appended to result |
| `SECURITY_GUIDANCE_BLOCK=1` | **block mode** — write refused, warning returned as the block reason |
| `SECURITY_GUIDANCE_DISABLE=1` | kill switch — complemento loads but does nothing |

**Enabling:** `hermes complementos enable security-guidance` (or check the box in `hermes complementos`).

**Disabling again:** `hermes complementos disable security-guidance`.

**What it does not do (yet):** the upstream Anthropic complemento has two more layers — an LLM diff review on each agente turn that touched files, and an agenteic commit-time review that traces data flow across files. Neither is ported. The agente can already run those reviews on demand via `delegate_task`.

### observability/langfuse

Traces Hermes turns, LLM calls, and herramienta invocations to [Langfuse](https://langfuse.com) — an open-source LLM observability platform. One span per turn, one generation per API call, one herramienta observation per herramienta call. Usage totals, per-type token counts, and cost estimates come out of Hermes' canonical `agente.usage_pricing` numbers, so the Langfuse panel sees the same breakdown (input / output / `cache_read_input_tokens` / `cache_creation_input_tokens` / `reasoning_tokens`) that appears in `hermes logs`.

The complemento is fail-open: no SDK installed, no credentials, or a transient Langfuse error — all turn into a silent no-op in the hook. The agente loop is never impacted.

**Setup (interactive — recommended):**

```bash
hermes herramientas          # → Langfuse Observability → Cloud or Self-Hosted
```

The wizard collects your keys, `pip install`s the `langfuse` SDK, and adds `observability/langfuse` to `complementos.enabled` for you. Restart Hermes and the next turn ships a trace.

**Setup (manual):**

```bash
pip install langfuse
hermes complementos enable observability/langfuse
```

Then put the credentials in `~/.hermes/.env`:

```bash
HERMES_LANGFUSE_PUBLIC_KEY=pk-lf-...
HERMES_LANGFUSE_SECRET_KEY=sk-lf-...
HERMES_LANGFUSE_BASE_URL=https://cloud.langfuse.com   # or your self-hosted URL
```

**How it works:**

| Hook | Behaviour |
|---|---|
| `pre_api_request` / `pre_llm_call` | Open (or reuse) a per-turn root span "Hermes turn". Start a `generation` child observation for this API call with serialized recent messages as input. |
| `post_api_request` / `post_llm_call` | Close the generation, attach `usage_details`, `cost_details`, `finish_reason`, assistant output + herramienta calls. If no herramienta calls and non-empty content, close the turn. |
| `pre_herramienta_call` | Start a `herramienta` child observation with sanitized `args`. |
| `post_herramienta_call` | Close the herramienta observation with sanitized `result`. `read_file` payloads get summarized (head + tail + omitted-line count) so a huge file read stays under `HERMES_LANGFUSE_MAX_CHARS`. |

Session grouping keys off the Hermes session ID (or task ID for sub-agentes) via `langfuse.propagate_attributes`, so everything in a single `hermes chat` session lives under one Langfuse session.

**Verify:**

```bash
hermes complementos list                 # observability/langfuse should show "enabled"
hermes chat -q "hello"              # check the Langfuse UI for a "Hermes turn" trace
```

**Optional tuning** (in `.env`):

| Variable | Default | Purpose |
|---|---|---|
| `HERMES_LANGFUSE_ENV` | — | Environment tag on traces (`production`, `staging`, …) |
| `HERMES_LANGFUSE_RELEASE` | — | Release/version tag |
| `HERMES_LANGFUSE_SAMPLE_RATE` | `1.0` | Sampling rate passed to the SDK (0.0–1.0) |
| `HERMES_LANGFUSE_MAX_CHARS` | `12000` | Per-field truncation for message content / herramienta args / herramienta results |
| `HERMES_LANGFUSE_DEBUG` | `false` | Verbose complemento logging to `agente.log` |

Hermes-prefixed and standard SDK env vars (`LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`, `LANGFUSE_BASE_URL`) are both accepted — Hermes-prefixed wins when both are set.

**Performance:** the Langfuse client is cached after the first hook call. If credentials or SDK are missing, that decision is also cached — subsequent hooks fast-return without re-checking env vars or reloading config.

**Disabling:** `hermes complementos disable observability/langfuse`. The complemento module is still discovered, but no module code runs until you re-enable.

### google_meet

Lets the agente **join, transcribe, and participate in Google Meet calls** — take notes on a meeting, summarize the back-and-forth after, follow up on specific points, and (optionally) speak replies back into the call via TTS.

**What it adds:**

- A headless virtual participant that joins a Meet URL using browser automation
- Live transcription of the meeting audio via the configured STT proveedor
- A `meet_summarize` / `meet_speak` / `meet_followup` herramientaset the agente invokes to act on what it heard
- Post-meeting artifacts (transcript, speaker-attributed notes, action items) saved under `~/.hermes/cache/google_meet/<meeting_id>/`

**Setup:**

```bash
hermes complementos enable google_meet
# Prompts you to sign in via the complemento's OAuth flow on first use —
# needs a Google account with Meet access. Host approval may be required
# if the meeting enforces "only invited participants can join".
```

Usage from chat:

> "Join meet.google.com/abc-defg-hij and take notes. After the call, send me a summary with action items."

The agente kicks off the meeting join, streams the transcription back into its contexto as the call proceeds, and produces a structured summary when the meeting ends (or when you tell it to stop).

**When to use it:** recurring standups where you want a bot to transcribe + summarize for async attendees; deposition-style interviews where you want structured notes; any case where you'd otherwise need Fireflies / Otter / Grain. When you'd rather not have an AI listening in — don't enable it.

**Disabling:** `hermes complementos disable google_meet`. Any cached transcripts and recordings stay in `~/.hermes/cache/google_meet/` until you remove them.

### hermes-achievements

Adds a **Steam-style achievements tab to the panel** — 60+ collectible, tiered badges generated from your real Hermes session history. Herramienta-chain feats, debugging patterns, vibe-coding streaks, habilidad/memoria usage, modelo/proveedor variety, lifestyle quirks (weekend and night sessions). Originally authored by [@PCinkusz](https://github.com/PCinkusz) as an external complemento; brought in-tree so it stays in lockstep with Hermes feature changes.

**How it works:**

- Scans your entire `~/.hermes/state.db` session history on the panel backend
- Per-session stats are cached by `(started_at, last_active)` fingerprint, so only new or changed sessions re-analyze on subsequent scans
- First-ever scan runs in a background thread — the panel never blocks waiting for it, even on databases with thousands of sessions
- Unlock state is persisted to `$HERMES_HOME/complementos/hermes-achievements/state.json`

**Tier progression:** Copper → Silver → Gold → Diamond → Olympian. Each card exposes a "What counts" section listing the exact metric being tracked.

**Achievement states:**

| State | Meaning |
|---|---|
| Unlocked | At least one tier achieved |
| Discovered | Known achievement, progress visible, not yet earned |
| Secret | Hidden until Hermes detects the first related signal in your history |

**API** — routes mount under `/api/complementos/hermes-achievements/`:

| Endpoint | Purpose |
|---|---|
| `GET /achievements` | Full catalog with per-badge unlock state (returns a pending placeholder while the first cold scan is running) |
| `GET /scan-status` | State of the background scanner: `idle` / `running` / `failed`, last duration, run count |
| `GET /recent-unlocks` | Twenty most recently unlocked badges, newest first |
| `GET /sessions/{id}/badges` | Badges earned primarily in one specific session |
| `POST /rescan` | Manual synchronous rescan (blocks; use when the user clicks the rescan button) |
| `POST /reset-state` | Clear unlock history and cached snapshot |

**State files** — live under `$HERMES_HOME/complementos/hermes-achievements/`:

| File | Contents |
|---|---|
| `state.json` | Unlock history: which badges you've earned and when. Stable across Hermes updates. |
| `scan_snapshot.json` | Last completed scan payload (served immediately on panel load) |
| `scan_checkpoint.json` | Per-session stats cache keyed by fingerprint (makes warm rescans fast) |

**Performance notes:**

- Cold scan on ~8,000 sessions takes a few minutes. It runs in a background thread on first panel request; the UI sees a pending placeholder and polls `/scan-status`.
- **Incremental results during a cold scan** — the scanner publishes a partial snapshot every ~250 sessions so each panel refresh shows more badges unlocked as the scan progresses. No minute-long stare at zeros.
- Warm rescan reuses per-session stats for every session whose `started_at` + `last_active` fingerprint matches the checkpoint — completes in seconds even on large histories.
- The in-memoria snapshot TTL is 120s; stale requests serve the old snapshot immediately and kick a background refresh. You never wait on a spinner just because TTL expired.

**Enabling:** Nothing to enable — `hermes-achievements` is a panel-only complemento (no lifecycle hooks, no modelo-visible herramientas). It auto-registers as a tab in `hermes panel` on first launch. The `complementos.enabled` config only gates lifecycle/herramienta complementos; panel complementos are discovered purely via their `panel/manifest.json`.

**Opting out:** Delete or rename `complementos/hermes-achievements/panel/manifest.json`, or override it with a user complemento of the same name in `~/.hermes/complementos/hermes-achievements/` that ships no panel. The complemento's state files under `$HERMES_HOME/complementos/hermes-achievements/` survive — reinstalling preserves your unlock history.

## Adding a bundled complemento

Bundled complementos are written exactly like any other Hermes complemento — see [Build a Hermes Complemento](/guides/build-a-hermes-complemento). The only differences are:

- Directory lives at `<repo>/complementos/<name>/` instead of `~/.hermes/complementos/<name>/`
- Manifest source is reported as `bundled` in `hermes complementos list`
- User complementos with the same name override the bundled version

A complemento is a good candidate for bundling when:

- It has no optional dependencies (or they're already `pip install .[all]` deps)
- The behaviour benefits most users and is opt-out rather than opt-in
- The logic ties into lifecycle hooks that the agente would otherwise have to remember to invoke
- It complements a core capability without expanding the modelo-visible herramienta surface

Counter-examples — things that should stay as user-installable complementos, not bundled: third-party integrations with API keys, niche workflows, large dependency trees, anything that would meaningfully change agente behaviour by default.

---