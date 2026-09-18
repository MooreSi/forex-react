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
| Modules orphaned by the port | `python -m tools.refactor_audit.orphan_modules --check` | 0 | **0** (was 32) |
| Controller operations with no caller | `pytest tests/refactor/test_controller_operations_have_callers.py` | 0 | **12** (was 47) |

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
| 120 | Connections, Remote Node, and the trading-control handover | **YES** | **code complete, NOT signed off** | Claude | Four surfaces the port had dropped or mis-wired; see below. The handover is a money-path control and needs a demo session. |

## Coverage, after the port

The three floors the port knocked down were restored by writing tests, not by
moving the floors, and all three now sit **above** where they started:

| Area | Before the port | Floor was | After | Floor now |
|---|---|---|---|---|
| `backend/src/controllers` | 79.3% | 79.3 | **100%** | 99.2 |
| `backend/src/services/analytics` | 66.0% | 66.0 | **79.1%** | 79.1 |
| `backend/src/services/cluster` | 86.9% | 86.9 | **96.0%** | 96.0 |
| `backend/src/api` | — | — | **94.6%** | 94.6 (new) |
| `backend/src/config` | 42.7% | 42.7 | **77.6%** | 77.6 |
| `backend/src/services/risk` | 88.8% | 88.8 | **93.5%** | 93.5 |
| `backend/src/services/engines` | — | — | **100%** | 100 (new) |

Floors were raised for the six areas this work moved and left alone everywhere
else. Several other areas now sit well above their floors; that slack is not
this change's doing, and raising a floor somebody else earned is how a ratchet
starts failing for reasons nobody can explain.

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

---

# Task 120 — what a second pass over the port found (2026-09-18)

The port was reported complete. It was not, and the way it was not is worth
recording: **every gap below was invisible to a green suite**, because the
tests that should have caught them replaced the thing being tested with a
recorder. A recorder accepts any signature, any key and any order.

## The one that could have cost money

**`PUT /api/node/active-trader` was a flag write.**

Two paired nodes point at the same MT5 account, and `active_trader` decides
which of them may open new positions. The NiceGUI header ran a handshake:
remote → local asked the VPS to stand down and waited for its acknowledgement
*before* starting this node's engines; local → remote stopped this node's
engines *before* asking the VPS to resume. The React port kept the endpoint and
dropped the sequence. Setting `local` therefore left the VPS believing it still
owned the account while this node was marked active — two sets of engines, one
balance, and nothing on either screen saying so.

It also had no caller: no React control ever reached it. That is the only
reason this is a near miss rather than an incident.

Now: `services/cluster/handover.py` holds the sequence and its failure
behaviour, ordered so that a peer which does not answer leaves the account with
**no** active trader rather than two. `tests/services/cluster/test_handover.py`
asserts the order through one shared list — a peer with its own call list can
say "stand-down happened" and "the flag was set" but not which came first, and
which came first is the entire property. Five mutations planted, four caught;
the survivor is recorded in the test that should have caught it, because the
guard it removed is genuinely redundant for the current engine set.

The header now has the control back (`ActiveTraderControl`), with a
confirmation that states the order before it happens and shows the backend's
own note afterwards.

**This has not been through a demo session.** Like task 060, its tests are
green and that is not sign-off.

## Settings > Connections wrote to columns that do not exist

Three separate defects in one tab, all of which a recorder hid:

* `recipient` was sent to `email_config`, whose column is `to_addr`. An
  operator who typed an address got a 500 and no saved address.
* The Telegram section offered `api_id`, `api_hash` and `phone` against
  `telegram_config`, which has `bot_token_enc`, `chat_id` and `enabled`. Those
  three are the **Telethon reader's** credentials and live in `config.yaml` —
  a different store for a different thing. All three writes would have failed.
* `PUT /api/settings/telegram` handed the whole request body to
  `save_telegram_config(bot_token, chat_id, enabled)`. Every save raised.

Also dropped: the provider picker, the schedule (daily/weekly/ORB), the Resend
key, `from_addr`, `use_tls`, and all four **test-send buttons**. A settings form
for an SMTP account with no way to send a test is a form that cannot be
diagnosed, which is most of what it is for.

`tests/api/test_settings_writes_reach_the_store.py` reads the tab's own field
list and checks each name against the schema and the real save signatures, so
the next one goes red in CI rather than at the keyboard.

## Settings > Remote Node did not exist

`frontend/pages/remote_node.py` (270 lines) was deleted with the rest and
nothing replaced it. Unreachable from the dashboard since the port: starting
and stopping the sync server, connecting out to a VPS, headless mode,
centralized signal generation, and the one-off model-snapshot copy.

Now `backend/src/api/routers/remote.py` + `RemoteTab`. A failed server start is
recorded as **off**, because a stored "enabled" with no listener has the next
restart claim the VPS is accepting connections when it is not.

## A trading window that is not a time

`PUT /api/schedule/schedule` stored whatever it was sent. The schedule is text,
parsed when the engines ask whether they may trade, and `_find_active_block`
swallows a parse failure — so a broken window never matches, the engines stop
trading or keep trading past a stop, and nothing says why.

Range matters as much as shape and was the part missing: `_parse_hm` is
`int(h) * 60 + int(m)`, so `"25:00"` parses happily to 1500 minutes. The check
went into `set_trading_schedule` rather than the router, because a paired node
forwards its grid straight there.

## Housekeeping done in the same pass

* `services/engines/registry.py` — one table of which engines exist and their
  bulk lifecycle. The handover had grown a second copy.
* `api/redaction.py` — one copy of "never echo a credential", now that two
  routers answer with stored configuration.
* `notifications_controller` was the last `awaiting-react-port` orphan; wiring
  the test sends cleared it. **That allowlist class is now empty.**
* Controller operations with no caller: 18 → 12. Three were deleted rather than
  re-wired (`parse_hm`, `start_stopped_engines`, `stop_running_engines`) — their
  behaviour moved to services where it protects every caller, and a forwarder no
  router calls is a route to nowhere.
* `test_ui_shutdown_helper.py` asserted `nicegui imports <= 2` against a
  contract enforced at zero — a test that could not fail. It asserts zero now.

## What the remaining 12 are waiting for

Six (`history_controller.ticket_*_map`) and `system_controller.local_today`
belong to the Analysis deal-level trade table and its calendar. They are
blocked on the same owner decision as that table, below, and are not dead.

The other five are surfaces this pass did not reach:
`sync_controller.is_remote_active`, `is_centralized_remote_mode` and
`note_remote_setting` (remote-awareness on the engine panels and the signals
card), `settings_controller.switch_environment_db` (the demo/live environment
switcher that lived in the app shell) and `get_app_config_async`. Each is one
question for whoever next touches that surface: wire it or delete it.

## Mutation testing, this pass

29 planted, 28 caught. The one survivor is the `_NOT_BULK_STARTED` guard, and
it is recorded in `test_handover.py` with the reason it cannot fail today.

---

# Task 130 — catching up with `MooreSi/forex` (2026-09-18)

Six commits landed upstream after this branch's clone point (`1d594cb`..
`0622ea7`, 2026-09-16 to 2026-09-18): the signal decision log, the lot ceiling,
CME futures context, instant-entry reporting, and two backfill fixes. About
5,600 lines.

**The merge itself was nearly clean.** Five conflicts, four of them NiceGUI
files this branch had deleted and which stay deleted; one real content conflict
in `telegram_controller.__all__`, where both sides had added an export. Out of
8,463 tests, four failed and one file would not collect — all five for the same
reason: an upstream test reads a NiceGUI page that no longer exists.

That is the port's recurring shape and it is worth naming. Upstream writes
"the switch is reachable" tests that read the page source, which is exactly
right — `docs/todo/refactor` records a guardrail that scanned a deleted
directory and printed "all good" for months. Every one of those tests needs its
subject re-pointed at the React tab, and the assertions themselves carry over
unchanged.

## What was ported

**CME futures context switch** → the Signal Generator tab's capability list.
The wording is the feature: there is no CME feed in this build, and turning the
switch on records an intent and changes nothing the engine decides. An owner
who turned it on, saw no change and concluded the engine was broken would be
the failure; believing a later decision was informed by CME data would be
worse. `test_cme_context_switch.py` reads that text and fails if it stops
admitting it.

**Signal Decision Log** → a new sub-tab on Parsing, plus
`api/routers/decision_log.py`. Two readouts, because they become useful at
different times: the summary is worth reading from the first decision, and
champion-vs-challenger needs closed trades so it says nothing for days. Neither
polls — this sits behind a live trading page, and a card that queries a
database every few seconds for a number that moves twice a day is a cost with
no benefit. The recording toggle is a 13th parsing switch under a new RESEARCH
badge, default off.

## The one thing this found

`test_the_card_admits_it_is_not_connected` reads forward from the FIRST
occurrence of the key in the file. The comment I wrote above the capability
quoted both phrases the test looks for, so the assertion passed on my own
comment and the description underneath could have said anything. Two planted
mutations survived, which is how it was caught; the comment no longer repeats
them and both mutations now fail.

That is the same class as everything in task 120: a test that cannot fail is
worse than no test, because it reports a guarantee it is not providing.

## Evidence

```
python -m tools.checks all   ->  11 of 11, green   (8,463 tests)
npm test                     ->  239 passed
```

10 mutations planted against the ported surfaces; 10 caught, after the two that
survived were made to fail.

## Not merged

Nothing. The branch is level with `upstream/main` as of `0622ea7`.

**`MooreSi/forex` is still untouched by this work** and is free to keep moving.
Each future catch-up is this same shape, and the cost is proportional to how
many UI-reachability tests the upstream work brought with it — not to how many
lines it changed.

---

# Task 140 — the Signal Generator was driving the wrong machine (2026-09-18)

The same defect as the Local/Remote handover in task 120, one screen along, and
found by the same question: *what does this control do when the other node is
the one trading?*

**When the remote node is the active trader, this machine's sub-engines are
stood down.** Pressing Stop on the Signal Generator tab therefore stopped an
engine that was not running, on a node that is not trading, while the peer's
copy kept generating signals — and the screen said "stopped". The sync server's
own `_handle_engine_control` names it exactly: those buttons *"would otherwise
act on the Mac's own stood-down engine instance, which does nothing useful
while looking like it worked."* The NiceGUI panels routed around it. The React
port did not, so every control on that tab was local-only between 2026-09-18
and this change.

Three separate things had to be right, and each had cost a real evening before:

1. **Where a control lands.** Not "am I in Remote mode": under centralized
   signal generation the engines moved HERE, so the local ones are the live
   ones even though the peer trades. `remote_stats_facade` already knew that
   and `services/cluster/remote_control.py` defers to it rather than
   re-deriving it.

2. **What the panel shows.** In Remote mode the settings the engines obey are
   the peer's, so they are overlaid on the local row — overlaid, not replacing
   it, because the snapshot carries a handful of keys and a wholesale swap
   would blank every setting the broadcast never sends.

3. **Where "current" is read before a toggle inverts it.** From the local row
   while writing to the peer, a toggle recomputes the same current on every
   click and re-sends the same target state for ever. That is the live-confirmed
   bug the old `note_remote_setting` existed to close: **Bounce stuck OFF,
   Breakout stuck ON.**

The tab now says which machine it is driving, in three states rather than two.
"Centralized" is the one an operator would otherwise misread: the header says
REMOTE and these engines are still the live ones.

**A limit, stated rather than hidden.** The sync protocol carries exactly one
risk setting between nodes — the AI-evaluation flag, which has its own
endpoint. Every other tunable has no remote route, so in Remote mode saving one
writes a row the trading node will not read. The banner says so.

## Debt

Controller operations with no caller: 12 → **9**. Three more were deleted
rather than re-wired (`is_remote_active`, `is_centralized_remote_mode`,
`note_remote_setting`): a browser cannot make the snapshot write, and the only
code that knows a write went to the peer at all is the routing, so all three
moved into the service with it.

Of the 9 that remain, **seven are one feature**: the six `ticket_*_map`
builders and `local_today` belong to the Analysis deal-level trade table and
its calendar, both blocked on the facade decision below. The other two are the
demo/live environment switcher that lived in the app shell.

## Evidence

```
python -m tools.checks all   ->  11 of 11, green
npm test                     ->  244 passed
```

16 mutations planted across the routing, the router and the banner; 16 caught.
Floors raised for `backend/src/api` (94.6 → 94.7) and
`backend/src/services/cluster` (96.5 → 97.1).

**Not signed off.** Starting an engine lets it generate signals again, which is
the same authority this panel has always had — but it now does so on a machine
the operator is not sitting at. It belongs with the handover and task 060 in
the demo session that is still owed.
