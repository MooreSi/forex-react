# 080 — The remaining eight tabs

**Money:** mixed **Depends on:** 070

Eight tabs, in the order their absence costs most. Each is the same three steps as 050: a router
that forwards, a domain folder under `components/`, tests first including a negative control.

| Tab | NiceGUI source | Lines | Money | Notes |
|---|---|---|---|---|
| Settings | `pages/settings/` | 3,306 | some | Twelve domains in one package. Split into `SettingsTab` files from the start; do not port it as one panel. Risk and MT5 sections touch money. |
| Analysis (History) | `pages/history/` | 2,034 | no | Equity curve, calendar, heatmap, trade table. The 10MB WebSocket buffer existed for `days=3650`; paginate instead of pushing it. |
| AI Analysis | `pages/ai_trade_analysis/`, `pages/ai_summary.py` | 1,852 | no | |
| Signal Generator | `pages/test_panel/`, engine panels | 1,700+ | no | The three engine panels are near-identical and deliberately not collapsed — see frontend-conventions §10. Porting them is not permission to merge them. |
| Backtest | `pages/backtest/` | 811 | no | |
| Parsing (Telegram) | `pages/telegram/` | 752 | no | |
| News | `pages/news.py` | 313 | no | |
| About | `app/_about.py` | 475 | no | ~530 lines of it is Glossary content. Content, not code — move it to a data file, not a component. |

Not a tab, but in this task: `pages/update_panel.py`, `pages/remote_node.py`,
`pages/dpm_analysis.py`, `pages/expert_tunables.py`. Expert Tunables is rendered generically from
the tunables registry and stays generic — a hand-written React form per tunable defeats
`/add-tunable`.

## Behaviours the port dropped, that a tab must bring back

Each of these was pinned by a test whose subject the big-bang replace deleted.
They are **not covered by anything today**. The test named beside each one is
what has to exist again before its tab can be called done.

| Behaviour | Tab | What the new test must prove |
|---|---|---|
| Editing one pending signal must not write another row's values | Trading (pending signals editor) | Render several rows, edit one, assert the others are untouched. The NiceGUI version was a loop-closure capture bug; React's version is a handler reading state from an earlier render. Was `tests/refactor/test_late_binding.py::TestThePendingSignalsEditorSpecifically`. |
| History deal attribution — which session and which comment a deal belongs to | Analysis | Was `tests/ui/test_history_session_attribution.py` and `test_history_comment_attribution.py`. The helpers lived in the page; when the tab is ported they belong in `history_controller` or a service, with the tests moving to match. |
| The four theme presets re-skin only the neutral scale | any | Was `tests/core/test_ui_theme.py`. The React equivalent is the token set in `frontend/src/index.css`: a test that a preset changes `surface-*`/`ink-*` and leaves `profit`, `loss`, `warning` and `remote` alone. |
| The HTF-bias Asian exemption switch is reachable | Signal Generator | Already guarded: `tests/risk/test_htf_bias_gate_asian_exemption.py::TestTheSwitchIsReachable` goes red the moment the tab is marked ported without the switch. |
| Every parsing settings row reaches the screen | Parsing | The NiceGUI version shipped with its settings body in a function nothing called, so `immediate_market_entry` could not be turned on and a bare "Buy Now" signal was missed. Pin every row of the category list, not just the most eye-catching card. |
