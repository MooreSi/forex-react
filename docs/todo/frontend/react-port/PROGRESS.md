# React port — PROGRESS

**Shared status log.** Claim a row (name + date under Owner) before starting it, flip its
Status as you go, leave a one-line Note — commit, blocker or decision. A task reported Done
that is not is the exact failure this repo's rules exist to prevent.

## Status key
`not started` · `in progress` · `blocked` (say why) · `done` (date + commit)

## The numbers this pack moves

| Metric | Command | At start (2026-09-18) | Now (2026-09-18) |
|---|---|---|---|
| NiceGUI Python lines under `frontend/` | `find frontend -name '*.py' -not -path '*/node_modules/*' \| xargs wc -l` | 21,434 | **0** |
| Tabs served by React | — | 0 / 10 | **10 / 10** |
| Top-layer import contracts | `python -m tools.refactor_audit.import_contracts --check` | 2, at zero, scanning `frontend/` | 2, at zero, scanning `backend/src/api/` |
| `no-nicegui-in-the-backend` | same | 2, baselined | **0, enforced at zero** — `nicegui` is no longer a dependency |
| Modules orphaned by the port | `python -m tools.refactor_audit.orphan_modules --check` | 0 | **1** (was 32) |
| Controller operations with no caller | `pytest tests/refactor/test_controller_operations_have_callers.py` | 0 | **18** (was 47) |

Update this block when a task lands. It is the pack's only honest progress metric.

## Tasks

| # | Task | Money | Status | Owner | Notes |
|---|---|---|---|---|---|
| 010 | API layer and contracts | no | done (2026-09-18) | Claude | `backend/src/api/` with routers for system, auth, chart, trading, orders. Both top-layer contracts retargeted from `frontend` to `backend/src/api` and renamed; `server.py` is the single exemption. `tests/api/` — 75 tests. |
| 020 | Auth and the login gate | no | done (2026-09-18) | Claude | Signed-cookie session over the same per-install secret. 401 JSON for API paths, redirect for page paths. Default-off auto-login, unreadable-config-stays-shut and first-run-setup all pinned. |
| 030 | React build and serving | no | done (2026-09-18) | Claude | Vite + React 19 + TS + Tailwind 4. `run.py` calls `uvicorn.Server` instead of `ui.run`. SPA fallback is middleware, not a catch-all — see the note in the domain README. |
| 040 | App shell | no | done (2026-09-18) | Claude | Header, ten tabs in the documented order, Chart default, one shared poll, colour tokens, `NotPortedPanel` for the other eight. |
| 050 | Chart tab | no | done (2026-09-18) | Claude | Candles, EMA 9/21/50, RSI, FVG zones, bid/ask lines, open-position markers and a trades panel. lightweight-charts. |
| 060 | Trading tab | **YES** | **code complete, NOT signed off** | Claude | Positions with close, signals list, manual market order with a two-step confirmation. **No demo session has been run. The order and close paths have never executed against a broker through this UI.** See "Sign-off owed" below. |
| 070 | Remove NiceGUI | no | done (2026-09-18) | Claude | 21,434 lines and 57 test files deleted. Three tests kept and relocated; 32 backend modules allowlisted as orphaned-by-the-port. |
| 080 | The remaining eight tabs | mixed | done (2026-09-18) | Claude | All ten tabs are React. Three are narrower than their originals for boundary reasons, named in the task file. |
| 090 | Licence screens | no | done (2026-09-18) | Claude | Ported to plain server-rendered HTML; `nicegui` removed from the project |
| 100 | The rest of the Trading tab | **YES** | done (2026-09-18) | Claude | Limit order, schedule, EA templates, pending-signal editor |
| 110 | Node & updates | no | done (2026-09-18) | Claude | Pairing, autostart, restart, applying a release — a Settings tab, as it never was a top-level one |
| 090 | Licence screens | no | not started | — | still NiceGUI; the reason `nicegui` is still a dependency |

## Coverage, after the port

The three floors the port knocked down were restored by writing tests, not by
moving the floors, and all three now sit **above** where they started:

| Area | Before the port | Floor was | After | Floor now |
|---|---|---|---|---|
| `backend/src/controllers` | 79.3% | 79.3 | **100%** | 99.2 |
| `backend/src/services/analytics` | 66.0% | 66.0 | **79.1%** | 79.1 |
| `backend/src/services/cluster` | 86.9% | 86.9 | **96.0%** | 96.0 |
| `backend/src/api` | — | — | **88.4%** | 88.4 (new) |

`python -m tools.checks all` is green, 11 of 11.

## The one thing still missing, and why it needs you

**Analysis has no deal-level trade table.** Everything else is ported.

The NiceGUI page built that table from `bridge.get_deal_history()`, reached
through `engine._bridge` — past the controller boundary, which the React layer
may not do. There are exactly three ways to give it a legal route, and all
three are decisions rather than details:

1. **Add `get_deal_history` to `TradingRuntime`.** This is the natural home
   (`compute_mt5_performance` already works this way) and it is blocked by the
   facade gate, whose rule is one-way by design: *"its public surface is
   exactly the curated facade allowlist — names may be removed from the
   allowlist, never added."* Adding one is a baseline change, and CLAUDE.md
   says to stop and ask.
2. **A service function taking the engine**, like `engine_reads.open_trades`.
   It would have to read `engine._bridge`, and
   `test_no_production_code_reaches_into_a_runtime_private` derives its leak
   set from the runtime's own members, so it would fail the day it landed.
3. **Live without it.** The tab already reports the account's headline numbers,
   the hourly P&L grid, the channel scorecard and ladder reach — all from the
   local database. The trade table is the per-deal detail on top.

Option 1 is one line plus an allowlist entry and is what I would do. It needs
your word, because "a ratchet baseline would have to rise" is on the stop-and-ask
list.

## Sign-off owed

**Task 060 is not done.** Its code is written and its tests are green, and
neither of those is sign-off for a money path. What is missing is a demo
session: the owner watching a trade open and close **through the React UI**,
against the demo account, with the result recorded in this file. Until that
happens, treat the Trading tab's Market order and Close controls as unproven —
every test behind them uses a sentinel engine that records calls and returns a
canned dict, which proves the arguments are forwarded unchanged and proves
nothing about what a broker does with them.

## Deleted tests, and their replacements

`tests/frontend/` (54 files), `tests/ui/test_history_comment_attribution.py`,
`tests/ui/test_history_session_attribution.py` and
`tests/core/test_ui_theme.py` were deleted in the same change as their subject.
Three were kept because they are repo-wide rules that merely lived in a
frontend directory:

| Was | Now |
|---|---|
| `tests/frontend/test_server_bind.py` | `tests/api/test_server_bind.py` — unchanged; it was always about `run.py` |
| `tests/frontend/test_no_silent_excepts.py` | `tests/api/test_no_silent_excepts.py` — retargeted from `frontend/` to `backend/src/api/`, with a new fail-closed test so it cannot scan an empty tree |
| `tests/frontend/test_install_guide_matches_the_code.py` | `tests/refactor/test_install_guide_matches_the_code.py` — unchanged |

The behaviours the deleted login-gate tests pinned have named twins in
`tests/api/test_auth_gate.py`: default-off auto-login, unreadable config
staying shut, first-run setup instead of an impossible login form, a wrong
password setting no session, and the referrer round-trip.

**The rest have no twin yet, and that is the honest position.** The history
attribution helpers and the theme presets were page-level code in tabs that are
not ported. Task 080 must re-establish those behaviours and their tests when it
ports the Analysis tab; they are not covered by anything today.
