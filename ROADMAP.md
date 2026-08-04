# iHealth Roadmap

Running log of what's shipped, in progress, and planned — kept up to date as we go
so work can continue across sessions without losing track. Newest first.

## Shipped to `develop`, awaiting Deep's try before promoting to `main` (2026-07-28)

Adapting 4 UI/UX patterns from Deep's expense tracker (mechanics matched exactly,
labels translated to health-tracking domain):

1. **Floating "+" → full-screen add-type picker.** Dashboard becomes the home
   screen (read-only stats/charts/tables). The only way to log anything is a
   circular floating action button, always visible, opening a full-screen picker:
   Log Food / Log Weight / Log Habit / Log Workout, each its own full-screen form.
2. **Hamburger (☰) header menu.** Replaces header buttons — dropdown panel with
   profile/weight summary, theme toggle, divider, link to Settings.
3. **Type-ahead search, hidden until focused.** Extends the pattern already partly
   built for meal items/workout to be consistent everywhere: empty by default,
   prefix-match first then substring, capped results.
4. **Dashboard sub-tabs, cards → chart → table skeleton.** Today / Weekly Trend /
   Habit Streaks, each following the same layout shape.

**Data model change (the big one):** weight, workout, and habit-completions become
append-only event logs — editing/re-logging never overwrites a past entry in place,
it always appends a new dated one. Concretely: `days/YYYY-MM-DD.json` records keep
their per-date file structure (so `index.json`, `manifest.json`,
`Master_Daily_Log.md` generation, and cloud sync all still work conceptually the
same way), but `weight`/`workout`/habit fields change from single scalars to arrays
of timestamped entries (`weightEntries`, `workoutEntries`, `habitEvents`). All 198
days of existing real history get migrated into this shape with zero data loss —
existing scalar values become single-entry arrays, nothing is dropped or guessed.

Food logging (breakfast/snack/dinner item lists) and daily `notes` are **not**
part of this change — items were already append-only within a meal, and a note is
a description of the day, not a historical reading you'd regret overwriting.

**Status: built, migrated, tested, pushed to `develop`.** `index.html` is
substantially rewritten (was ~1180 lines, still one file, same proven cloud-sync
mechanics underneath). `health-data-dev`'s 198 days migrated to the new schema
with a spot-checked zero-loss conversion (weight/workout/habit scalars →
single-entry arrays; multi-dose isabgol and custom-tracker values correctly split
into individual events). 55 scripted-browser checks across two suites:
- Suite 1 (41 checks) against the real migrated 198-day history: home screen /
  FAB / picker mechanics, hamburger open-close, cloud connect loading all 198
  days, append-only guarantees explicitly verified both ways (a new Log Weight
  entry lands on *today* without touching 2026-07-27's historical entry; logging
  weight twice in one sitting produces two entries, not an overwrite), workout
  type-ahead (hidden until typed, relevant suggestions), habit form
  re-save-without-changes producing zero duplicate events, food quick-picks from
  real history, all three dashboard sub-tabs rendering, Excel export against the
  new schema (5 sheets).
- Suite 2 (14 checks) with 22 days of fresh synthetic dummy data seeded directly
  (varied weights/workouts/habits, not just the historical data) specifically to
  exercise Weekly Trend and Habit Streaks with real week-over-week and streak
  signal, plus a full cycle through all four entry-form types back to home.

**Not yet done:** merging to `main`, migrating production `health-data`. Waiting
on Deep to try the `/dev/` build first, per his explicit instruction.

## Ship discipline

- Everything above lands on `develop` first: local logic + a scripted-browser pass
  (weeks of realistic dummy data, click through every sub-tab, the add-picker, and
  each entry type) before it's called done.
- `health-data-dev` gets migrated to the new schema now, since it's an isolated
  copy made exactly for this kind of testing.
- `health-data` (production data) and `main` (production app) are **not** touched
  until Deep tries the dev build and approves promoting it.
- This file gets updated every time something moves from planned → shipped, or a
  new decision gets made, so a fresh session can pick up context without re-deriving
  it.

## Shipped

- **2026-07-27**: July 1-27 daily log added to both data repos (see `health-data`
  repo's commit history for detail); new "Power Nap" boolean tracker.
- **2026-07-26**: UI/UX pass 1 — hero weight field, reordered meal sections,
  dismissible quick-pick chips, workout autocomplete, Settings moved to Dashboard,
  Dashboard defaults to Monthly Detail, Excel export. Also fixed a Pages deploy bug
  (GitHub Environment protection silently blocking `develop` deploys).
- **2026-07-15 → 2026-07-16**: Phases A-D of the original build — daily log app,
  WhatsApp generator (still draft, unconfirmed against Deep's real template),
  dashboard, cloud sync, `develop`/`main` branch + dual Pages deploy setup.

## Known open items (not blocking current work)

- WhatsApp message template is still a draft — needs Deep's review against his
  actual format.
- No repo-admin access from this session for: creating repos, renaming repos,
  enabling Pages, triggering/rerunning workflow runs via API. All of these need
  Deep to do the one-time manual step when they come up.
