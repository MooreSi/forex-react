# React port — open questions for the owner

Answer inline under each question. An unanswered question is not a blocker unless it says it
is: the provisional default is stated so work can continue, and the answer changes it.

## Q1 — Do the eight unported tabs stay visible while they are unported?

Big-bang replace means NiceGUI is deleted before all ten tabs exist in React. The tab strip can
either show all ten, with the unported eight rendering an honest "not ported yet" panel, or show
only the tabs that work.

**Provisional default: show all ten, with a placeholder panel naming the task that will fill
it.** A missing tab reads as a lost feature; a placeholder reads as work in progress.

**Answer:**

## Q2 — Does the React dashboard need to work on a phone?

The NiceGUI dashboard was only ever used on a desktop browser. Responsive layout is cheap if it
is designed in from the first component and expensive to retrofit.

**Provisional default: desktop-first, and the layout does not break below 1024px — but no phone
layout is designed, and nothing is tested at phone width.**

**Answer:**

## Q3 — Light mode?

The NiceGUI app is dark-only, deliberately: a genuine light mode needed Quasar's internal
component styling overridden everywhere. In React with CSS tokens that objection disappears —
light mode becomes a second set of token values.

**Provisional default: dark only, but every colour goes through a token from day one, so light
mode stays a decision rather than a rewrite.**

**Answer:**

## Q4 — How does a release rebuild the bundle?

Committing `frontend/dist/` means a release can ship a stale UI if somebody forgets to rebuild.
Options: a pre-commit hook, a CI check that rebuilds and fails on a diff, or discipline.

**Provisional default: a CI check that rebuilds the bundle and fails if it differs from the
committed one.** Discipline is what the stale-guardrail incident in CLAUDE.md is about.

**Implemented on that default (2026-09-18):** `.github/workflows/checks.yml` now installs Node,
type-checks and tests the dashboard, then rebuilds it and fails on any diff under `frontend/dist`.
Asset filenames are content-hashed, so any real source change moves them. Say if you would rather
it were a pre-commit hook instead.

**Answer:**

## Q5 — Three coverage floors that the port knocked down

`python -m tools.checks all` is 10 of 11 green on this branch. The coverage ratchet is the one that
fails, and it is the only thing between this work and a clean run:

```
backend/src/controllers          69.1%  < floor 79.3%   (226 statements)
backend/src/services/analytics   57.8%  < floor 66.0%   (377 statements)
backend/src/services/cluster     84.7%  < floor 86.9%   (38 statements)
```

Nothing got worse in the code. Those lines were executed because the NiceGUI page render tests
imported the pages, which called the controllers, which reached the repos. Eight of those pages are
gone, so the callers are gone with them.

**The baseline was deliberately not lowered.** "Weaken a ratchet baseline to make CI pass" is on
CLAUDE.md's never list, and the floor is the record of what was once covered. It would also be easy
to cheat: a sweep that calls every controller forwarder under a mock would restore the number and
verify nothing, which is precisely what this repo's own warning about coverage says not to trust.

Three honest ways out, and this one is yours:

1. **Port the tabs** (task 080). The real callers come back and the floors are met the way they were
   met before. Correct, and the slowest.
2. **Write genuine tests for what is now uncovered.** Most of the analytics gap is three repos
   (`ai_analysis_repo`, `read_repo`, `trade_history_repo`) — real SQL that deserves tests on its own
   merits and never had them; the coverage was incidental. This is worth doing regardless of the
   port, and `/coverage-gap` is the skill for it.
3. **Record a dated one-time adjustment**, with the port named as the reason, and let the floors
   climb back as tabs land. Honest if it is written down; a quiet `--update-baseline` is not.

**Answer:**
