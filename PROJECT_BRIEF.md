# PROJECT BRIEF — iHealth

*Written: Sat, 11th July 2026 · Status: awaiting Deep's approval*

## What this is

iHealth is Deep's personal health app. It merges two projects — **iFood-Assist**
(daily food/dietician logging) and **iActive** (activity tracking, never built) —
into one app that Deep uses on his iPhone every day to log food, activities,
weight and habits, with a dashboard that always shows the current picture:
weight trend toward the **65.0 kg goal**, food habits, fasting hours, steps
and workouts.

## What we learned in the interview

- **User:** Deep, non-technical, logs primarily on his **iPhone**, often via
  voice-to-text. Follows the Sathwaa program with a dietitian.
- **Problem:** logging today needs a Claude session, and July logs sit
  unsynced; iActive never got built. Deep wants one app, one dashboard.
- **Must keep everything tracked today:** weight, steps, workout, fasting
  (16-hr target, auto-calculated from prior dinner end → breakfast start),
  breakfast/snack/dinner with times, power nap, routine habits (water,
  morning detox, deep breathing, Isabgol 2x, multivitamin). **No Flonase**
  (discontinued 23 Jun 2026).
- **Flexibility:** Deep can add his own new trackers anytime (e.g., knee
  physio exercises) without a rebuild.
- **WhatsApp stays:** after logging, the app produces the exact formatted
  dietitian message (current template) for Deep to paste into WhatsApp with
  his meal photos.
- **Data home:** a **private GitHub repo** is the master copy. Health data is
  never placed in any public repo. The app's code lives in a public repo
  with zero personal data in it.
- **History:** everything since Dec 2025 is loaded in from day one,
  including recovering the July 1–10 logs from earlier sessions.
- **Not required for "done":** automatic write-back to the OneDrive folder.
  The OneDrive iFood-Assist folder stays as-is; we can sync it in Cowork
  sessions when Deep asks.

## Architecture (plain English)

iHealth is a website that installs on the iPhone home screen like a real app.
Its code is hosted free on GitHub Pages from a public repo that contains no
personal information. When Deep finishes a day's log, the app saves it
directly into his private GitHub repo (using a one-time key stored only on
his phone) — that private repo is the single master copy of his health data.
The dashboard is part of the same app and reads from that private repo, so
it is always current the moment a log is saved, on any device Deep signs
into. No servers to pay for, nothing new to maintain.

Technical notes (for the record, Deep doesn't need to read this):
- Public repo: `borannavar1984/iFood-AssistApp` (rename/rebrand to iHealth)
  — static PWA on GitHub Pages. Vanilla HTML/JS, no build step.
- Private repo: `borannavar1984/iHealth-Data` — JSON per-day log files +
  generated Markdown mirror (Master_Daily_Log format preserved).
- Writes via GitHub Contents API with a fine-grained PAT scoped to the
  private repo only, stored in localStorage on Deep's phone; never in code.
- Log schema: fixed core fields + user-defined tracker definitions stored
  as data, so new trackers need no code change.
- History migration: parse `_Archive/Master_Daily_Log.md` + monthly
  reports; recover Jul 1–10 from prior session transcripts; no
  interpolation — explicitly stated values only (house rule).

## Build phases

- **A — Data foundation:** parse all history into structured data, recover
  July 1–10, verify every date and weight against the source files.
- **B — The app:** tap-based daily log (under 2 minutes), custom trackers,
  WhatsApp message generator using the exact current template.
- **C — The dashboard:** weight trend to 65 kg, habit adherence, fasting,
  steps/activity; readable on iPhone and desktop.
- **D — Hosting & handover:** GitHub Pages, private data repo, and ONE
  short click-by-click setup checklist for Deep (create private repo,
  create key, add app to home screen).

## Acceptance criteria (things Deep can personally verify)

1. iHealth opens from an icon on your iPhone home screen, like any app.
2. You complete a full daily log — every field you track today — by tapping,
   in under 2 minutes, and it saves by itself.
3. After logging, one tap gives you the formatted dietitian message, exactly
   in today's template, ready to paste into WhatsApp.
4. You open the dashboard on phone or computer and today's log is already
   there, with your full weight history since Dec 2025 charted toward 65 kg.
5. You add a brand-new tracker yourself (e.g., "knee exercises") in the app
   and it shows up in the next day's log — no help from me.
6. Searching your public GitHub repo shows zero personal or health data.

## House rules carried over

Factual logs only, no invented foods, fasting = prior dinner end → breakfast
start with a factual flag under 16 hrs, plain copyable WhatsApp output,
date confirmation at month boundaries, mis-tap recovery by re-asking.
