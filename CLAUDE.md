# iHealth — Working Agreement

Single-file PWA (`index.html` — HTML/CSS/JS, no build step) for personal health
tracking. Two branches: `develop` (test at `/dev/`, separate `health-data-dev`
data repo) and `main` (production, `health-data` repo). See `ROADMAP.md` and
`STATUS.md` for what's shipped; see `SETUP.md` for hosting/cloud-sync setup.

Deep is the product owner. Claude (this session) is the orchestrator: it does
the actual development itself and delegates only specific, bounded jobs to the
subagents below.

## Standing rules

1. **Always start in plan mode.** For any non-trivial task — anything beyond a
   one-line fix — write a plan first: which files change, the approach, and
   the risks. Wait for Deep's explicit approval before writing code. Trivial
   changes (typo fixes, a copy tweak) don't need a plan.

2. **The main session does the development.** Subagents are for context-heavy
   or specialized work only — research, code review, test-writing — not for
   implementing features. Don't spawn a subagent for something small enough to
   just do directly.

3. **Keep the main thread lean.** Hand off large file reads, broad codebase
   search, and outside research to subagents so their bulk stays out of this
   session's context — pull back only their conclusions.

4. **After code changes, run `reviewer` and `qa` before calling a task done.**
   Both are read-mostly checks: `reviewer` for quality/security, `qa` for
   tests actually passing. Fix what they find, or explain to Deep why not.

5. **Commit in small, reviewable steps.** Prefer several focused commits over
   one large one; each should stand on its own.

## Subagents

Defined in `.claude/agents/`:

- **researcher** (haiku, read-only + web) — studies comparable apps/features
  on request and returns a prioritized suggestions list as a plan. Never
  touches code.
- **reviewer** (sonnet, read-only) — reviews a diff or file set for
  correctness, quality, and security; returns specific, actionable findings.
- **qa** (sonnet, read + Bash + Edit) — writes and runs tests against a
  change, reports pass/fail and coverage gaps.

## House rules carried over from prior sessions

- Never invent or interpolate health data — unset fields stay `null`.
- `develop`/`/dev/` first, always. `main`/production only after Deep tries the
  dev build and says to promote it.
- Real historical data in `health-data` (prod) is untouched unless Deep asks
  for a specific migration, and even then it's tested against
  `health-data-dev` first.
