# iHealth — Status

Plain-English log of what's built and what's still open, against the phases in
`PROJECT_BRIEF.md` (Deep approved 2026-07-12).

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

- Weight trend chart with a 65.0 kg goal line.
- Fasting-hours chart, bars colored by whether the 16 hr target was hit.
- Steps chart.
- Habit adherence chart (% yes across all logged days), computed correctly across
  both the legacy schema (boolean isabgol, array-shaped breakfast/dinner, `water:
  "completed"`) and the new live schema (isabgol count, `{time, items}` meals) — see
  the `norm*()` helpers in `index.html`.
- Stat cards: latest weight, distance to goal, days logged, 16 hr fasting rate.

Tested with 28 scripted-browser checks (mocked GitHub Contents API): fresh daily log
entry end-to-end, custom tracker add, fasting calc, WhatsApp generation, cloud
connect/sync, dashboard render, theme toggle, date-change confirmation, reload
persistence — plus a second pass loading the real 171-day migrated history through
the app to confirm legacy-shaped records render correctly with no JS errors and the
dashboard aggregates across the full history without crashing.

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
