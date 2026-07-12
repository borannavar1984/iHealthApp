# iHealth

Deep's personal health app. Merges food/dietician logging with activity tracking into
one daily log, with a dashboard tracking weight toward a 65.0 kg goal.

Static PWA, no build step. Hosted free on GitHub Pages. Installs on the iPhone home
screen like a real app (Safari → Share → Add to Home Screen).

## What it does

- **Daily log** (`index.html`, Today tab): weight, steps, workout, breakfast/snack/
  dinner with times, auto-calculated fasting (prior dinner end → today's breakfast
  start, 16 hr target), power nap, habits (water, morning detox, deep breathing,
  isabgol 2x, multivitamin), and any custom trackers Deep has added himself.
- **WhatsApp message generator**: one tap turns today's log into a formatted message
  ready to paste into WhatsApp for the dietitian, alongside meal photos.
- **Dashboard**: weight trend to goal, fasting adherence, steps, habit adherence —
  computed client-side from synced history.
- **Custom trackers**: tap "+ Add tracker" in the app, no code change needed. Saved to
  `trackers.json` in the private data repo.

## Data

All of Deep's actual health data (weight, food, fasting logs) lives in a **separate
private repo**, `borannavar1984/health-data` — never here. This repo is
public and contains zero personal data; searching it should always come up empty on
anything personal. See that repo's README for its layout.

The app talks to the private repo via the GitHub Contents API using a fine-grained
personal access token Deep generates once and stores only in his phone's local
storage — never in this code.

## Setup

See `SETUP.md` for the one-time click-by-click checklist (private repo, access token,
GitHub Pages, add to home screen).

## Status

See `STATUS.md` for what's built, what's a draft/placeholder (the WhatsApp message
format hasn't been confirmed against Deep's actual template yet), and what's still
open.
