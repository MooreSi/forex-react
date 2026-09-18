# React frontend port — plan pack

**Status:** in progress
**Owner decision:** 2026-09-18 — the owner authorised the port and chose **big-bang replace**.
**Repo:** this pack lives in `MooreSi/forex-react`, a separate repository. `MooreSi/forex`
(the NiceGUI app that trades live) is **not touched by any task here**.

## Why this exists, given the decision on record

`docs/system/domains/frontend/010-the-react-decision.md` records that a React/Next.js/shadcn
rewrite was **proposed and rejected on 2026-08-06**, on cost: a Node runtime added to a
Python-only Windows installer, a second process to supervise on the VPS, and ~18,000 lines
rewritten against an HTTP API that did not exist.

That rejection named one prerequisite and one trigger:

- **Prerequisite:** the `frontend-reaches-the-backend-through-controllers` contract at zero.
  It reached zero on 2026-09-02 (`PROGRESS.md` of the restructure pack). **Met.**
- **Trigger:** "only if the UI's limitations actually bite — then it becomes a frontend-only
  project, decidable on evidence."

The owner exercised that trigger on 2026-09-18. The stated reason is presentation and
customisability, not a defect in NiceGUI. Two of the three original cost objections are
answered by the decisions below; the third (lines rewritten) is real and is what this pack
sequences.

## Decisions locked with the owner (2026-09-18)

| Decision | Choice |
|---|---|
| Does the port happen | **Yes**, in `forex-react`. `forex` is untouched. |
| Coexistence with NiceGUI | **No.** Big-bang replace — React is the only UI. No `/app` route, no dual shell. |
| Node at runtime | **No.** Node is a developer dependency. The compiled bundle is committed and served by FastAPI, so the installer stays Python-only. |
| First review point | Foundation + the Trading and Chart tabs wired end to end. |

## The shape of the work

NiceGUI put the UI in the same process as the backend, so a page called a controller function
directly. React cannot. The port is therefore two builds, not one:

```
   before                              after
   ──────                              ─────
   browser                             browser
     │ socket.io                         │ HTTP/JSON
   NiceGUI page (Python)               React app  (frontend/src, built to frontend/dist)
     │ direct call                       │
   controllers/                        backend/src/api/  ← NEW: routers, one per domain
     │                                   │ direct call
   services/                           controllers/
     │                                   │
   db/                                 services/ → db/
```

`backend/src/api/` is the new top layer and inherits the frontend's rules verbatim: it may
import **only** `backend.src.controllers`, it never touches `backend.src.db`, and it never
decides whether an order is allowed. The two import contracts that used to scan `frontend/`
are retargeted to it — see task 010. A contract left pointing at a directory with no Python in
it would pass every run while protecting nothing, which is the exact failure this repo was
rebuilt after.

## What must NOT change

- **The backend.** No service, repo, engine or controller behaviour changes in this pack. If a
  router needs something a controller does not expose, the answer is a flat forwarding function
  on the controller, never logic in the router.
- **The close path.** `close_trade`, `record_close`, `_make_close_trade_ctx`,
  `partial_close_trade` are frozen. A React button that closes a trade calls the same
  controller the NiceGUI button called, with the same arguments.
- **Money safety in the UI.** Every rule in the NiceGUI frontend-conventions §8 carries over:
  explicit confirmation naming instrument, direction and size; the backend decides whether an
  order is allowed; rejections surfaced verbatim; demo vs live unmistakable.
- **Semantic colours.** green = profit, red = loss, yellow/amber = warning, blue = remote/VPS,
  gray = neutral. Dark only. These move into CSS tokens, they do not get remapped.
- **No test is deleted to make a change pass.** NiceGUI render tests are deleted only when
  their subject is deleted, in the same commit, each one named — see task 070.

## Roadmap

| # | Task | Money | Depends on |
|---|---|---|---|
| 010 | [The API layer and its contracts](010-api-layer.md) | no | — |
| 020 | [Auth: session cookie and the login gate](020-auth.md) | no | 010 |
| 030 | [The React build and how it is served](030-build-and-serve.md) | no | 010 |
| 040 | [App shell: header, tabs, theme, one poll](040-app-shell.md) | no | 030 |
| 050 | [Chart tab](050-chart-tab.md) | no | 040 |
| 060 | [Trading tab](060-trading-tab.md) | **YES** | 040 |
| 070 | [Remove NiceGUI](070-remove-nicegui.md) | no | 050, 060 |
| 080 | [The remaining eight tabs](080-remaining-tabs.md) | mixed | 070 |
| 090 | [The licence screens are still NiceGUI](090-licence-screens.md) | no | 070 |

Tasks 050 and 060 are independent of each other once 040 lands. 060 is money-touching: it
carries the manual-entry and close controls, and it ships alone.

## Files

| File | What it is |
|---|---|
| [PROGRESS.md](PROGRESS.md) | Live status log. Claim a row before starting it. |
| [QUESTIONS.md](QUESTIONS.md) | Open owner decisions, answerable inline. |
