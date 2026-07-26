# iHealth — Status

Plain-English log of what's built and what's still open, against the phases in
`PROJECT_BRIEF.md` (Deep approved 2026-07-12).

## UI/UX revamp (2026-07-16, on `develop`)

Deep did a UI/UX pass on his expense tracker app and asked which of those changes
translate to iHealth. Applied here (see full plan discussion for what was skipped
and why — notably, no floating-"+"/discrete-entry-type restructuring, since this
app's one-form-logs-the-whole-day model is intentional per the 2-minute daily-log
acceptance criterion, not a gap to fix):

- **Hero field first**: Weight is now its own lead section, bigger font, autofocused
  on load. Date moved down (after My Trackers, before Notes) since "today" is almost
  always correct. Each meal section now leads with food items/quick-picks; the time
  input moved below as a secondary, optional field.
- **Dismissible quick-pick chips**: breakfast/snack/dinner "Frequent" chips now have
  a "×" to permanently hide one from future quick-picks (`ihealth_dev_hidden_items_v1`)
  without touching history — autocomplete still includes hidden items, only the
  tap-target row is decluttered.
- **Workout autocomplete**: the Workout field now has a recency-ranked `<datalist>`
  built from history (was plain free text with no assistance).
- **Settings moved off the entry form**: Cloud Sync and Backup no longer live inside
  the daily-log form — both now sit in a "⚙️ Settings" block at the top of the
  Dashboard, reachable without touching the logging flow.
- **Dashboard default tab flipped**: Monthly Detail (pre-selected to the current
  month) is now the landing sub-tab instead of the all-time Summary (renamed from
  "Overview") — "how am I doing this month" beats a lifetime aggregate as the first
  thing you see.
- **Excel export**: new "Export (.xlsx)" button next to Backup, client-side via
  SheetJS — Daily Log, Weight Trend, Fasting, Habits, Workouts, and Monthly Summary
  sheets. Same no-native-charts caveat as any lightweight export library; positioned
  as clean data to pivot/chart yourself.
- Fixed a real bug surfaced while wiring the above: `saveDay()` never refreshed the
  meal quick-picks or (new) workout autocomplete after saving — a freshly-logged item
  wouldn't show as a suggestion until you navigated to a different date and back.
  Both now refresh immediately on save.

Tested with 28 new scripted-browser checks across two suites (hero field/autofocus,
reordered meal sections, chip dismiss + persistence + autocomplete-still-includes-
hidden-items, workout autocomplete, settings relocated + inaccessible from Today tab,
default Monthly Detail tab, Excel export — both the graceful-failure path when the
CDN library is blocked, and a stubbed-library pass verifying the actual sheet/row
data is correct: 6 sheets, correct names, full history row counts) — on top of the
existing 37+9 checks from earlier passes, which still hold except where they
asserted the specific behavior just changed on purpose (title text, Cloud Sync
location, default dashboard tab); those are superseded by the new suite rather than
fixed in place, since "fixing" them would mean reverting this revamp.

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
