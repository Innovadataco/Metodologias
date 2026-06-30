<!-- source: website/docs/guia-usuario/features/curator.md -->
# Curator

# Curator

The curator is a background maintenance pass for **agente-created habilidads**. It tracks how often each habilidad is viewed, used, and patched, moves long-unused habilidads through `active → stale → archived` states, and periodically spawns a short auxiliary-modelo review that proposes consolidations or patches drift.

It exists so that habilidads created via the [self-improvement loop](/guia-usuario/features/habilidads#agente-managed-habilidads-habilidad_manage-herramienta) don't pile up forever. Every time the agente solves a novel problem and saves a habilidad, that habilidad lands in `~/.hermes/habilidads/`. Without maintenance, you end up with dozens of narrow near-duplicates that pollute the catalog and waste tokens.

By default (`prune_builtins: true`) the curator can archive **unused bundled built-in habilidads** (shipped with the repo) after `archive_after_days` of non-use, alongside the agente-created habilidads it primarily manages. Hub-installed habilidads (from [agentehabilidads.io](https://agentehabilidads.io)) are always off-limits. Set `curator.prune_builtins: false` to restore the old agente-created-only behavior, where bundled habilidads are never touched. The curator also **never auto-deletes** — the worst outcome is archival into `~/.hermes/habilidads/.archive/`, which is recoverable.

Tracks [issue #7816](https://github.com/NousResearch/hermes-agente/issues/7816).

## How it runs

The curator is triggered by an inactivity check, not a cron daemon. On CLI session start, and on a recurring tick inside the puerta de enlace's cron-ticker thread, Hermes checks whether:

1. Enough time has passed since the last curator run (`interval_hours`, default **7 days**), and
2. The agente has been idle long enough (`min_idle_hours`, default **2 hours**).

If both are true, it spawns a background fork of `AIAgente` — the same pattern used by the memoria/habilidad self-improvement nudges. The fork runs in its own prompt cache and never touches the active conversation.

:::info First-run behavior
On a brand-new install (or the first time a pre-curator install ticks after `hermes update`), the curator **does not run immediately**. The first observation seeds `last_run_at` to "now" and defers the first real pass by one full `interval_hours`. This gives you a full interval to review your habilidad library, pin anything important, or opt out entirely before the curator ever touches it.

If you want to see what the curator *would* do before it runs for real, run `hermes curator run --dry-run` — it produces the same review report without mutating the library.
:::

A run has two phases:

1. **Automatic transitions** (deterministic, no LLM). Habilidads unused for `stale_after_days` (30) become `stale`; habilidads unused for `archive_after_days` (90) are moved to `~/.hermes/habilidads/.archive/`. This is the always-on pruning behavior — it runs whenever the curator is enabled, with no aux-modelo cost.
2. **LLM consolidation** (single aux-modelo pass, `max_iterations=8`) — **OFF by default**. When `curator.consolidate: true`, the forked agente surveys the agente-created habilidads, can read any of them with `habilidad_view`, and decides per-habilidad whether to keep, patch (via `habilidad_manage`), consolidate overlapping ones into class-level umbrellas, or archive via the terminal herramienta. Consolidation treats a habilidad as a full package: if a habilidad has `references/`, `templates/`, `scripts/`, `assets/`, or relative links to those paths, the curator must either keep it standalone, re-home the needed support files and rewrite paths, or archive the entire package unchanged — not flatten only `SKILL.md` into another habilidad's `references/` file.

:::info Consolidation is opt-in
By default the curator only **prunes** — the deterministic inactivity pass marks habilidads stale and archives long-unused ones. The opinionated LLM **consolidation** pass (umbrella-building, merging overlapping habilidads) is off by default because it costs aux-modelo tokens on every run and makes broad structural changes to your library. Turn it on with `curator.consolidate: true`, or run it once on demand with `hermes curator run --consolidate`.
:::

Pinned habilidads are off-limits to both the curator's auto-transitions and the agente's own `habilidad_manage` herramienta. See [Pinning a habilidad](#pinning-a-habilidad) below.

## Configuración

All settings live in `config.yaml` under `curator:` (not `.env` — this isn't a secret). Defaults:

```yaml
curator:
  enabled: true
  interval_hours: 168          # 7 days
  min_idle_hours: 2
  stale_after_days: 30
  archive_after_days: 90
  consolidate: false           # LLM umbrella-building pass — opt-in (prune-only by default)
  prune_builtins: true         # archive unused bundled built-in habilidads too (hub habilidads always exempt)
```

To disable entirely, set `curator.enabled: false`. To keep the always-on pruning but opt into LLM consolidation, set `curator.consolidate: true`.

### Running the review on a cheaper aux modelo

The curator's LLM review pass is a regular auxiliary task slot — `auxiliary.curator` — alongside Vision, Compression, Session Search, etc. "Auto" means "use my main chat modelo"; override the slot to pin a specific proveedor + modelo for the review pass instead.

**Easiest — `hermes modelo`:**

```bash
hermes modelo                   # → "Auxiliary modelos — side-task routing"
                               # → pick "Curator" → pick proveedor → pick modelo
```

The same picker is available in the web panel under the **Modelos** tab.

**Direct config.yaml (equivalent):**

```yaml
auxiliary:
  curator:
    proveedor: openrouter
    modelo: google/gemini-3-flash-preview
    timeout: 600               # generous — reviews can take several minutes
```

Leaving `proveedor: auto` (the default) routes the review pass through whatever your main chat modelo is, matching the behavior of every other auxiliary task.

:::note Legacy config
Earlier releases used a one-off `curator.auxiliary.{proveedor,modelo}` block. That path still works but emits a deprecation log line — please migrate to `auxiliary.curator` above so the curator shares the same plumbing (`hermes modelo`, panel Modelos tab, `base_url`, `api_key`, `timeout`, `extra_body`) as every other aux task.
:::

## CLI

```bash
hermes curator status         # last run, counts, pinned list, LRU top 5
hermes curator run            # trigger a run now (blocks until done). Prune-only unless curator.consolidate: true
hermes curator run --consolidate # force the LLM consolidation pass on for this run, overriding the config default
hermes curator run --background  # fire-and-forget: start the run in a background thread
hermes curator run --dry-run  # preview only — report without any mutations
hermes curator backup         # take a manual snapshot of ~/.hermes/habilidads/
hermes curator rollback       # restore from the newest snapshot
hermes curator rollback --list     # list available snapshots
hermes curator rollback --id <ts>  # restore a specific snapshot
hermes curator rollback -y         # skip the confirmation prompt
hermes curator pause          # stop runs until resumed
hermes curator resume
hermes curator pin <habilidad>    # never auto-transition this habilidad
hermes curator unpin <habilidad>
hermes curator restore <habilidad>  # move an archived habilidad back to active
hermes curator list-archived    # list habilidads currently in ~/.hermes/habilidads/.archive/
hermes curator archive <habilidad>  # manually archive a single habilidad now
hermes curator prune [--days N] # bulk-archive agente-created habilidads idle >= N days (default 90)
```

## Backups and rollback

Before every real curator pass, Hermes takes a tar.gz snapshot of `~/.hermes/habilidads/` at `~/.hermes/habilidads/.curator_backups/<utc-iso>/habilidads.tar.gz`. If a pass archives or consolidates something you didn't want touched, you can undo the whole run with one command:

```bash
hermes curator rollback        # restore newest snapshot (with confirmation)
hermes curator rollback -y     # skip the prompt
hermes curator rollback --list # see all snapshots with reason + size
```

The rollback itself is reversible: before replacing the habilidads tree, Hermes takes another snapshot tagged `pre-rollback to <target-id>`, so a mistaken rollback can be undone by rolling forward to that one with `--id`.

You can also take manual snapshots at any time with `hermes curator backup --reason "before-refactor"`. The `--reason` string lands in the snapshot's `manifest.json` and is shown in `--list`.

Snapshots are pruned to `curator.backup.keep` (default 5) to keep disk usage bounded:

```yaml
curator:
  backup:
    enabled: true
    keep: 5
```

Set `curator.backup.enabled: false` to disable automatic snapshotting. The manual `hermes curator backup` command still works when backups are disabled only if you set `enabled: true` first — the flag gates both paths symmetrically so there's no way to accidentally skip the pre-run snapshot on mutating runs.

`hermes curator status` also lists the five least-recently-used habilidads — a quick way to see what's likely to become stale next.

The same subcommands are available as the `/curator` slash command inside a running session (CLI or puerta de enlace platforms).

## What "agente-created" means

The curator only manages habilidads explicitly marked as **agente-created** in
`~/.hermes/habilidads/.usage.json`. A habilidad qualifies when ALL of the following
are true:

1. Its name is **not** in `~/.hermes/habilidads/.bundled_manifest` (bundled habilidads shipped with the repo).
2. Its name is **not** in `~/.hermes/habilidads/.hub/lock.json` (hub-installed habilidads).
3. Its `.usage.json` entry has `"created_by": "agente"` or `"agente_created": true`.

Currently, only the **background self-improvement review fork** sets this marker
— when it creates a new umbrella habilidad during its periodic review pass (~every 10
agente turns). The background fork runs with a write origin of `"background_review"`
(via `herramientas/habilidad_provenance.py`), which is the only path that triggers the
`mark_agente_created()` call in `habilidad_manage`.

Habilidads the foreground agente creates via `habilidad_manage(action="create")` during a
conversation are **not** marked as agente-created — they are considered
user-directed and the curator intentionally leaves them alone.

:::warning Your hand-written habilidads are NOT curated
If you manually created a `SKILL.md` or pointed Hermes at an external habilidad
directory, that habilidad will have a `.usage.json` entry with `created_by: null`
(or the field absent). The curator will not touch it. The same applies to
habilidads the foreground agente created at your request.

**To see which habilidads the curator actually manages**, run `hermes curator status`.
If the agente-created count is 0, no habilidads are currently in the curator's
jurisdiction — the LLM review pass is skipped and the report will show
`Modelo: (not resolved) via (not resolved)` with `Duration: 0s`.
:::

Habilidads that ARE agente-created follow the full lifecycle:

- `active` → (30d unused) `stale` → (90d unused) `archived`
- Pinned habilidads bypass all auto-transitions
- Archives are recoverable via `hermes curator restore <name>`

If you want to protect a specific habilidad from ever being touched — for example a
hand-authored habilidad you rely on — use `hermes curator pin <name>`. See the next
section.

## Pinning a habilidad

Pinning protects a habilidad from deletion — both the curator's automated archive passes and the agente's `habilidad_manage(action="delete")` herramienta call. Once a habilidad is pinned:

- The **curator** skips it during auto-transitions (`active → stale → archived`), and its LLM review pass is instructed to leave it alone.
- The **agente's `habilidad_manage` herramienta** refuses `delete` on it, pointing the user at `hermes curator unpin <name>`. Patches and edits still go through, so the agente can improve a pinned habilidad's content as pitfalls come up without a pin/unpin/re-pin dance.

Pin and unpin with:

```bash
hermes curator pin <habilidad>
hermes curator unpin <habilidad>
```

The flag is stored as `"pinned": true` on the habilidad's entry in `~/.hermes/habilidads/.usage.json`, so it survives across sessions.

Only **agente-created** habilidads can be pinned — `hermes curator pin` refuses on bundled and hub-installed habilidads with an explanatory message if you try. Hub-installed habilidads are never subject to curator mutation. Bundled built-in habilidads are only touched when `curator.prune_builtins: true` (the default), and even then only archived after `archive_after_days` of non-use — never patched, consolidated, or deleted. Set `curator.prune_builtins: false` to exempt bundled habilidads entirely.

A small set of **protected built-ins** is hardcoded as never-archivable and never-consolidatable, regardless of `curator.prune_builtins`, pin state, or LLM judgment. These back load-bearing UX — for example, `plan` powers the `/plan` slash-command flow — so silently archiving one would turn its slash command into an "Unknown command" error with no signal to you. Protected built-ins are filtered out of the curator's candidate list entirely, so the consolidation pass never sees them.

If you want a stronger guarantee than "no deletion" — for instance, freezing a habilidad's content entirely while the agente still reads it — edit `~/.hermes/habilidads/<name>/SKILL.md` directly with your editor. The pin guards herramienta-driven deletion, not your own filesystem access.

## Uso telemetry

The curator maintains a sidecar at `~/.hermes/habilidads/.usage.json` with one entry per habilidad:

```json
{
  "my-habilidad": {
    "use_count": 12,
    "view_count": 34,
    "last_used_at": "2026-04-24T18:12:03Z",
    "last_viewed_at": "2026-04-23T09:44:17Z",
    "patch_count": 3,
    "last_patched_at": "2026-04-20T22:01:55Z",
    "created_at": "2026-03-01T14:20:00Z",
    "state": "active",
    "pinned": false,
    "archived_at": null
  }
}
```

Counters increment when:

- `view_count`: the agente calls `habilidad_view` on the habilidad.
- `use_count`: the habilidad is loaded into a conversation's prompt.
- `patch_count`: `habilidad_manage patch/edit/write_file/remove_file` runs on the habilidad.

Bundled and hub-installed habilidads are explicitly excluded from telemetry writes.

## Per-run reports

Every curator run writes a timestamped directory under `~/.hermes/logs/curator/`:

```
~/.hermes/logs/curator/
└── 20260429-111512/
    ├── run.json      # machine-readable: full fidelity, stats, LLM output
    └── REPORT.md     # human-readable summary
```

`REPORT.md` is a quick way to see what a given run did — which habilidads transitioned, what the LLM reviewer said, which habilidads it patched. Good for auditing without having to grep `agente.log`.

:::note No candidates? Report shows `(not resolved)`
When the curator has **no agente-created habilidads** to review, the LLM review pass
is skipped entirely. The report header will show
`Modelo: (not resolved) via (not resolved)` with `Duration: 0s` — this does **not**
indicate a configuración error or modelo resolution failure. It simply means there
were no candidates, so no modelo was ever invoked. The auto-transition phase still
runs and reports its counts normally.
:::

### Rename map in the summary

If a run consolidated multiple habilidads under an umbrella (or merged near-duplicates), the user-visible summary printed at the end of the run includes an explicit rename map showing every `old-name → new-name` pair the curator applied. This is in addition to per-habilidad transition lines, so when a wave of renames lands you can spot them at a glance without diffing the JSON report. The hint also surfaces under `hermes curator pin` so you can pin the umbrella name immediately if you want to lock the new label in.

## Restoring an archived habilidad

If the curator archived something you still want:

```bash
hermes curator restore <habilidad-name>
```

This moves the habilidad back from `~/.hermes/habilidads/.archive/` to the active tree and resets its state to `active`. The restore refuses if a bundled or hub-installed habilidad has since been installed under the same name (would shadow upstream).

## Disabling per environment

The curator is on by default. To turn it off:

- **For one perfil only:** edit `~/.hermes/config.yaml` (or the active perfil's config) and set `curator.enabled: false`.
- **For just one run:** `hermes curator pause` — the pause persists across sessions; use `resume` to re-enable.

The curator also refuses to run if `min_idle_hours` hasn't elapsed, so on an active dev machine it naturally only runs during quiet stretches.

## See also

- [Habilidads System](/guia-usuario/features/habilidads) — how habilidads work in general and the self-improvement loop that creates them
- [Memoria](/guia-usuario/features/memoria) — a parallel background review that maintains long-term memoria
- [Bundled Habilidads Catalog](/reference/habilidads-catalog)
- [Issue #7816](https://github.com/NousResearch/hermes-agente/issues/7816) — original proposal and design discussion

---