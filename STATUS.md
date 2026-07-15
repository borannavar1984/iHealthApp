# iHealth — Status

Plain-English log of what's built and what's still open, against the phases in
`PROJECT_BRIEF.md` (Deep approved 2026-07-12).

## Pages troubleshooting note (2026-07-15)

Deep hit a 404 on the production URL right after switching the repo's Pages source
to "GitHub Actions" in Settings. The workflow itself had actually already deployed
successfully once before (2026-07-14), but that run happened *before* the Pages
source was explicitly saved in Settings — GitHub doesn't retroactively start serving
an old deployment just because the source setting changes after the fact, it needs a
fresh deployment once the source is properly saved. This commit exists to trigger
that fresh deployment (workflow_dispatch isn't available to my GitHub integration,
so a real push is how I force a new run).

## Phase A — Data foundation: done (partial)

- Found Deep's full daily-log history embedded in an uploaded `iHealthDashboard.html`
  (a `const ALL_DATA` blob) — 171 days, 2026-01-01 to 2026-06-29.
- Migrated into `borannavar1984/health-data`: one JSON file per day
  under `days/`, a fast-read `index.json` mirror, a generated `Master_Daily_Log.md`,
  and a `manifest.json` listing the 9 unlogged days in range.
- No values invented anywhere — unset fields are `null`, not guessed.
- **Open:** 2026-07-01 through 2026-07-11 not yet recovered (Deep will send later).

## Phase B — The app: done

- `index.html`: tap-based daily log — weight, steps, workout, breakfast/snack/dinner
  (time + item chips), fasting auto-calculated from yesterday's dinner time + today's
  breakfast time (16 hr target, flagged if under), power nap, habits (water, morning
  detox, deep breathing, isabgol as a 0/1/2 stepper against the "2x" target,
  multivitamin — Flonase intentionally left off the live form since it was
  discontinued 23 Jun 2026, but the field still displays correctly in historical data).
- Custom trackers: "+ Add tracker" lets Deep define a new one (yes/no, count, number,
  or text) from inside the app — saved to `trackers.json` in the data repo, shows up
  immediately and in every future day's form. No app code change needed.
- WhatsApp message generator: builds a formatted message from today's log and copies
  it to the clipboard. **This is a draft format** — I inferred it from the style of
  the migrated notes/breakfast/dinner text, since I don't have an example of Deep's
  actual current template. Needs his review.
- Cloud sync to the private data repo (GitHub Contents API, fine-grained PAT in
  localStorage only), same pattern proven in the iExpense app: per-day file write +
  index.json update, retry-on-conflict, offline-safe (saves locally, syncs later).
- Date-confirmation prompt when logging a day other than today (house rule: catch
  mis-taps / wrong-month entries before they're saved).

## Phase C — The dashboard: done

Split into **Overview** and **Monthly Detail** tabs (Deep flagged the first draft was
missing month-by-month numbers and running data):

- **Overview**: weight trend chart with a 65.0 kg goal line, fasting-hours chart
  (bars colored by whether the 16 hr target was hit), steps chart, **running chart**
  (miles per run — was missing from the first draft), habit adherence chart (% yes
  across all logged days), and a **month-over-month table** (days logged, avg weight,
  avg steps, 16h fasting rate, run miles, workout days per month).
- **Monthly Detail**: month picker, cards for that month (avg weight, total steps,
  fasting rate, run miles + workout day count), and a full day-by-day table (weight,
  steps, fasting, workout incl. run miles, habit-icon summary).
- Habit/meal normalization computed correctly across both the legacy schema (boolean
  isabgol, array-shaped breakfast/dinner, `water: "completed"`) and the new live
  schema (isabgol count, `{time, items}` meals) — see the `norm*()` helpers.
- Stat cards: latest weight, distance to goal, days logged, 16 hr fasting rate.
- Fixed a real bug during this pass: the month-over-month table was being built
  *after* the Chart.js-availability check, inside a block that `return`s early when
  Chart.js fails to load (e.g. offline) — so the table silently never rendered in
  that case. Restructured so chart rendering and the table are independent; the table
  always renders regardless of whether the chart library loaded.

**Quick-pick meal items**: breakfast/snack/dinner inputs now show up to 12
frequently-logged items as tap-to-add chips (learned from history, exact-string
match — similar phrasing counts separately), plus full autocomplete via `<datalist>`
for anything logged before. Tapping a chip adds the item without retyping; duplicate
taps are no-ops. Refreshes whenever `days` changes (date switch, save, cloud connect).

Tested with 37 scripted-browser checks total (mocked GitHub Contents API) across four
suites: fresh daily log entry end-to-end, custom tracker add, fasting calc, WhatsApp
generation, cloud connect/sync, theme toggle, date-change confirmation, reload
persistence; a pass loading the real 171-day migrated history through the app to
confirm legacy-shaped records render with no JS errors; quick-pick chips surfacing
real frequent items (e.g. "Mashed sweet potato" from dinner history) and populating
autocomplete; and the Overview/Monthly Detail split with real history (multi-month
table, running chart, month picker, day-level table).

## Phase D — Hosting & handover: partly done

- Private data repo: done.
- GitHub Pages: **not yet turned on** — needs Deep to flip it on in repo settings (no
  repo-admin access from here). Steps in `SETUP.md`.
- Access token + connect + add-to-home-screen: steps written in `SETUP.md`, not yet
  done by Deep.

## Known gaps / open decisions

1. WhatsApp template needs Deep's confirmation (see Phase B).
2. July 1–10 history gap, deferred by Deep.
3. GitHub Pages not yet enabled (Deep needs to do this — see `SETUP.md` step 2).
