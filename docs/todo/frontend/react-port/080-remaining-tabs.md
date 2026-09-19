# 080 — The remaining eight tabs

**Status:** done (2026-09-18) **Money:** mixed **Depended on:** 070

All ten tabs are React. `frontend/src/components/shell/tabs.ts` carries no
`notPorted` entry, and `AppShell.test.tsx` asserts that — it fails the day
somebody adds a tab and forgets to build it.

| Tab | Router | Components | Tests |
|---|---|---|---|
| AI Analysis | `api/routers/ai.py` | `components/ai/` | 14 API + 11 dashboard |
| Parsing | `api/routers/parsing.py` | `components/parsing/` | 15 + 15 |
| Signal Generator | `api/routers/engines.py` | `components/engines/` | 16 + 11 |
| Backtest | `api/routers/backtest.py` | `components/backtest/` | 15 + 12 |
| Analysis | `api/routers/history.py` | `components/history/` | 13 + 15 |
| Settings | `api/routers/settings.py` | `components/settings/` | 20 + 17 |
| News | `api/routers/news.py` | `components/news/` | 8 + 14 |
| About | `api/routers/system.py` (releases) | `components/about/` | 1 + 12 |

## What each tab is built on, and what it is not

Three tabs are narrower than their NiceGUI originals, for reasons that are
about the boundary rather than about effort. Saying so here is the point.

**Analysis has no deal-level trade table.** The NiceGUI page built one from
`bridge.get_deal_history()`, reached through `engine._bridge` — past the
controller boundary, which the React layer may not do. `TradingRuntime` has no
`get_deal_history` of its own and the runtime-facade gate is shrink-only, so
adding one is a decision, not a detail. The tab is built on what the boundary
does expose: `compute_mt5_performance`, the hourly P&L grid, the channel
scorecard and ladder reach. **Restoring the table needs a controller function
that does not exist yet.**

**Analysis also inherits the bugs/030 lesson structurally.** The old page had
three panels each polling the bridge independently: 4.3 calls a minute at idle
and 388 round-trips in 25 seconds on one page load. The fix there was a 60s
cache; here one endpoint assembles every panel and one shared poll reads it, so
a full render is one broker call — asserted by
`test_a_full_render_costs_one_trip_into_the_broker`.

**Settings is five tabs, not twelve panels.** Risk, MT5, Connections, Expert
tunables and Diagnostics, each owning one endpoint. The original reached 3,112
lines in one module because everything needing a setting was added to the same
surface; one component per domain is what stops that returning. Secrets are
write-only through the API: a denylist redacts anything whose key looks like a
credential, so a service that grows a new one is excluded by default.

**AI Analysis separates the free half from the billable one.** `/evidence` calls
no model and says `billable: false`; `/analyse` is a POST and says
`billable: true`. A page that could only show the numbers by asking a model
would make every glance cost money.

## Still not ported

Named rather than left to be discovered:

| Surface | Where it was | Why it is not here |
|---|---|---|
| ~~Trading: limit order, EA templates, schedule, pending-signal editor~~ | `frontend/pages/trading/` | **Done** (task 100). Strategy cards were folded into the channel-strategy recommendations rather than rebuilt as cards. |
| Analysis: the closed-trade table | `frontend/pages/history/_trade_table.py` | Needs deal history through a controller. See above. |
| ~~Remote node page, Update panel~~ | `pages/remote_node.py`, `pages/update_panel.py` | **Done** (task 110), as Settings → Node & updates. They were never top-level tabs and are not one now. |
| Email ORB report attachment | `pages/settings/_email.py` | `notifications_controller.ORB_CHART_CID` is still awaiting a caller; the report itself is wired. |

`AWAITING_REACT_PORT` in `tests/refactor/test_controller_operations_have_callers.py`
went from **47 to 31** and names every one of these. The orphan allowlist lost
26 of its 32 `awaiting-react-port` entries.

## Behaviours the port dropped, that a tab must bring back

| Behaviour | Tab | What the new test must prove |
|---|---|---|
| Editing one pending signal must not write another row's values | Trading — **done**: the id is in the URL, the dialog is keyed on it, and `TradingPanel.test.tsx` proves a second Edit does not carry the first row's draft | Render several rows, edit one, assert the others are untouched. The NiceGUI version was a loop-closure capture bug; React's version is a handler reading state from an earlier render. Was `tests/refactor/test_late_binding.py::TestThePendingSignalsEditorSpecifically`. |
| History deal attribution — which session and which comment a deal belongs to | Analysis — **still open**; needs the deal table, which needs a controller function | Was `tests/ui/test_history_session_attribution.py` and `test_history_comment_attribution.py`. The helpers lived in the page; when the tab is ported they belong in `history_controller` or a service, with the tests moving to match. |
| The four theme presets re-skin only the neutral scale | any — **still open**; there is one theme today | Was `tests/core/test_ui_theme.py`. The React equivalent is the token set in `frontend/src/index.css`: a test that a preset changes `surface-*`/`ink-*` and leaves `profit`, `loss`, `warning` and `remote` alone. |
| The HTF-bias Asian exemption switch is reachable | Signal Generator — **now due**: the tab is marked ported, so the guard is live | Already guarded: `tests/risk/test_htf_bias_gate_asian_exemption.py::TestTheSwitchIsReachable` goes red the moment the tab is marked ported without the switch. |
| Every parsing settings row reaches the screen | Parsing — **done**: `content/settings.ts` holds all 12 switches and `ParsingPanel.test.tsx` asserts every one reaches the screen | The NiceGUI version shipped with its settings body in a function nothing called, so `immediate_market_entry` could not be turned on and a bare "Buy Now" signal was missed. Pin every row of the category list, not just the most eye-catching card. |
