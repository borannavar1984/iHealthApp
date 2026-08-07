# iHealth Roadmap

Running log of what's shipped, in progress, and planned — kept up to date as we go
so work can continue across sessions without losing track. Newest first.

## Shipped: entry-form overhaul, "make it click-click-done" (2026-08-07, later same day)

Deep's core feedback: logging still felt slow and confusing in a few places,
and the FAB fan-menu wasn't a clean circle (screenshot showed labels/icons
crowding together in an uneven "1-2-2 tiers" layout instead of one uniform
arc).

1. **FAB fan-menu is now one true uniform semicircle.** All 5 items (Food,
   Weight, Message, Habit, Workout) sit on a single arc, evenly spaced 36°
   apart — Message naturally lands at the top-center (most prominent) slot
   instead of a separate stem. No overlap at any screen width tested.
2. **Weight entry is minimal by default.** Just the weight number and Save
   — steps/date/time/note are now behind a closed "+ More details"
   disclosure, so the common case really is type-a-number-and-done.
3. **Food form time chips are data-driven, not a wall of options.** Each
   meal now shows just the 2 times you actually log most often (learned
   from history) plus "Other…", instead of 6-7 fixed preset buttons.
4. **Settings: Preset Menu Items.** A dietitian (or Deep) can pre-load
   common food items per meal in Settings — these always show as instant
   tap chips in the food form regardless of logging history, so a new user
   isn't starting from a blank typing field. Synced to the cloud data repo
   (`preset_items.json`) the same way custom trackers are.
5. **Workout form rebuilt around activity type, not free text.** Pick a
   type (Running/Walking/Cycling/Yoga/Swimming/Gym/Other) and only the
   field that applies shows up — distance (mi) for the three you'd measure
   in miles, duration (min) for the rest — plus an optional style/notes
   field (e.g. "Vinyasa" for yoga). Supports multiple workouts in one visit
   (log Yoga *and* a walk same day): "+ Add to Today's Log" stages each one
   as a removable chip, "Save Workout" commits everything staged at once
   (and auto-stages the in-progress selection if Save is tapped directly,
   so a single quick entry never needs the extra tap). WhatsApp message and
   Excel export both show duration alongside distance now.
6. **Incidental fix:** dark/light theme toggle was silently non-functional
   — it called an undefined `nsKey()` function that a try/catch was
   swallowing, so the choice never actually persisted or restored. Fixed
   to use the same key-prefix pattern as everything else.

Tested with 21 new scripted-browser checks (weight minimal-flow + toggle,
data-driven time chips reflecting seeded history, preset item add → shows
as tap chip → adds to the meal, workout type-conditional fields, multi-entry
queue with 2 different activity types saved as distinct entries with correct
metrics, auto-stage safety net) plus the full 61-check suite from earlier
today (habit toggles/fasting, FAB overlap, Message quick-action, real
198-day-history regression) — all 82 passed before merging to `main`.

## Shipped: one-tap "Today's Message" (2026-08-07, later same day)

Deep wants to hand the app to his dietitian and other clients, so sharing
today's log needed to be one tap instead of buried in Settings:

- The "+" fan-out menu now has a 5th item, 💬 **Message**, alongside
  Food/Weight/Habit/Workout — all 5 evenly spaced around the FAB.
- Tapping it opens straight to today's WhatsApp-formatted message, already
  generated (no extra "Generate" tap needed since it's always today) — with
  a **Copy Message** button to paste straight into WhatsApp.
- If nothing's logged yet today, it shows a plain-language nudge ("Nothing
  logged for today yet — add your weight, food, or habits first") instead
  of an empty or misleading message.
- The original Settings → WhatsApp Message section is unchanged, still
  useful for generating a message for a past date; both paths now share one
  clipboard-copy helper instead of duplicating the copy logic.

Tested with 16 new scripted-browser checks (5-item fan menu, empty-state
nudge, generated message contains real logged data, copy actually copies,
old Settings generator still works via the shared helper, other 4 fan items
unaffected) plus the full existing 29-check suite from the habit/fasting
round earlier today — all 45 passed before merging to `main`.

## Shipped (2026-08-07)

Note: between the 2026-08-03 entry below and this one, a separate development
agent (Deep's "Perplexity development agent") pushed its own round of commits
directly to `develop`/`main`/`health-data-dev` — the iExpense visual design
language, a fan-out FAB replacing the quick-log bar, food item typeahead, meal
time-chips, custom-tracker swipe-to-delete, and a real dev/prod localStorage
collision fix — all reviewed and confirmed safe (see "Verified" below) before
this round of work continued on top of it.

This round addresses Deep's next feedback pass in full:

1. **Custom trackers, including built-ins, can now be turned off.** Settings
   → My Trackers lists all 5 built-in habits (💧 Water, 🌅 Morning Detox,
   🫁 Deep Breathing, 🌾 Isabgol, 💊 Multivitamin) with an On/Off toggle next
   to each — they stay as defaults but Deep can disable ones he doesn't want
   to log, without losing their history. Custom trackers keep the existing
   swipe-to-delete. Disabling a built-in only hides it from new logging (the
   Habit form, and dashboard adherence/streak math) — it never touches past
   entries, so old data still displays correctly if re-enabled later.
2. **Onboarding now asks which habits to track.** The first-launch welcome
   screen adds a checklist of the 5 built-in habits (all pre-checked) below
   the name/goal-weight fields — unticking one there is equivalent to
   switching it off in Settings afterward.
3. **Fasting box removed from the food entry form.** It no longer shows a
   live "X hrs" box while logging breakfast/snack/dinner. Fasting hours are
   still computed automatically and silently on every save, and still show
   up exactly where they already did — the dashboard's Fasting Rate cards
   (Weekly/Monthly/Overall) — nothing to add there, it was already
   dashboard-only.
4. **Fasting calc now uses the earliest real intake, not just breakfast
   time.** Previously it was strictly yesterday's dinner time → today's
   breakfast time. Now it also considers today's snack time and a
   Morning Detox habit event's time, and uses whichever is *earliest* as the
   fast-breaking point — e.g. detox water logged at 6:00 AM now correctly
   ends the fast at 6:00 AM even if breakfast is logged later at 8:30 AM.
   Habit entries in the Habit form now carry an optional time (defaults to
   "now", editable), the same idea as the existing meal time-chips, so this
   can be accurate rather than always "whenever I happened to tap Save."
   Recomputed and stored on both food-save and habit-save, since either can
   change the earliest-intake time.

**Verified (Deep asked explicitly):** dev (`ihealth_dev_*` keys, `/dev/`
build, `health-data-dev` repo) and production (`ihealth_*` keys, root build,
`health-data` repo) are on fully separate localStorage namespaces and
separate GitHub data repos — confirmed by diffing both repos' git history.
Production `health-data` is byte-identical to before (5a9da09, untouched,
still old scalar schema — the app's normalizers read it fine). `develop` and
`main` app code are identical (fast-forwarded, no divergence). `health-data-dev`
has one extra commit beyond my last migration — Deep added a "Running"
custom tracker while testing the dev build, which is exactly what that repo
is for; no data-integrity issue.

**Testing:** 29 scripted-browser checks — 17 new (onboarding habit checklist
default-checked + persists unchecked choice, Habit form excludes a disabled
built-in and includes an enabled one, every habit row has a time input,
Settings toggle flips a built-in on/off and persists, fasting box confirmed
absent from the food form, and the core fasting-math case: 20:00 dinner +
06:00 detox water yields 10h fasting vs. the 12.5h a breakfast-only calc
would have given for the same day) plus 12 regression checks against the
real 198-day migrated history pulled through mocked cloud sync (all three
dashboard tabs, both built-in-toggle and custom-tracker-swipe-delete
Settings rows, all four entry forms cycling cleanly). All 29 passed.

Deep's instruction for this round was explicit: "test validate verify
everything in dev, everything looks good then push it to production." Pushed
to `develop` first per house rule, all 29 checks green, then merged straight
to `main` per that same instruction — no separate approval pause this round.

## Shipped to `develop` (2026-08-03)

Major navigation and personalization overhaul:

1. **Editable user profile.** The hamburger menu no longer hardcodes "Deep" —
   the name and goal weight are stored in localStorage (`ihealth_dev_user_v1`)
   and editable via a new Profile Edit overlay. Tapping the profile section in
   the hamburger opens it. Goal weight is now dynamic everywhere (charts, cards,
   WhatsApp message, Excel export) instead of the old `const GOAL_WEIGHT =
   65.0`.

2. **First-time onboarding.** When the app launches with no stored user
   profile, a welcome screen appears asking for the user's name and goal weight.
   Completing it saves the profile and initializes the dashboard. Subsequent
   launches skip it automatically.

3. **FAB removed, quick-log bar added.** The floating "+" button is gone.
   Instead, a 4-button quick-log bar (Weight / Food / Habits / Workout) sits
   at the top of the dashboard — direct entry, no picker overlay needed.

4. **Dashboard tabs restructured.** Three new tabs replace the old
   Today / Weekly Trend / Habit Streaks:
   - **This Month** (default): current-month cards (avg weight, total steps,
     fasting rate, workout count), weight chart with goal line, and a
     last-7-days table.
   - **Monthly Detail**: month picker, per-month cards, weight chart, and a
     day-by-day table for the selected month.
   - **Overall**: all-time cards (latest weight, days logged, fasting rate,
     total workouts), full-history weight chart with goal line, habit
     adherence bar chart, and a month-over-month comparison table.

5. **Daily notes field** added to the food entry form (saves to `rec.notes`).

6. **Date confirmation prompts** on food/weight/workout saves when logging
   for a non-today date (house rule from the project brief).

7. **Backup includes user profile** in the JSON export.

Tested via Playwright screenshots at 390px mobile width: onboarding screen,
post-onboarding dashboard (This Month tab), Monthly Detail tab, Overall tab,
hamburger menu with editable name, and Profile Edit overlay — all rendered
cleanly with no visual defects.

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
