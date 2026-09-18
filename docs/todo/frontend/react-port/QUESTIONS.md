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

**Answer:**
