# 040 — App shell: header, tabs, theme, one poll

**Money:** no **Depends on:** 030 **Layer:** `frontend/src/`

## Problem

`frontend/app/` is 1,989 lines across four modules: framework patches, lifecycle hooks, the
header, the About/Glossary content, and the ten-tab composition. Most of it does not survive the
port — the NiceGUI timer patch, the socket.io buffer patch and the lifecycle hooks are all
framework-specific. What must survive is the **anatomy**: what the header shows, in what order,
and what each tab is called.

## Decision

- `AppShell` composes a header and the tab strip. Ten tabs, same names and order as
  `frontend/app/__init__.py:411-420`: AI Analysis, Chart, Trading, Parsing, Signal Generator,
  Backtest, Analysis, Settings, News, About. Default tab is Chart, as today.
- **One poll, not ten.** The React conventions are explicit: one shared poll fetching a
  consolidated payload, paused when the tab is backgrounded, deduping in flight. That replaces
  ~20 `ui.timer()` calls. The 5s cadence for account/positions is the established one and is
  kept; a new "is X fresh" need becomes a field on the consolidated response, not a new interval.
- **Semantic colours become CSS tokens** — `--profit`, `--loss`, `--warning`, `--remote`,
  `--neutral` — mapped to the same greens/reds/ambers/blues. They are not remapped. The neutral
  scale is what a theme re-skins, exactly as `theme.py` did.
- Unported tabs render `NotPortedPanel`, naming the task that will fill them (QUESTIONS Q1).

## Tests first (TDD)

Vitest + Testing Library, `frontend/src/**/__tests__/`.

- `AppShell.test.tsx`
  - `test_all_ten_tabs_are_present_and_in_the_documented_order`
  - `test_chart_is_the_default_tab`
  - `test_an_unported_tab_says_so_instead_of_rendering_empty`
- `usePoll.test.ts`
  - `test_one_interval_serves_every_subscriber`
  - `test_a_second_subscriber_does_not_start_a_second_interval` — the thing this exists to prevent.
  - `test_polling_pauses_when_the_document_is_hidden`
  - `test_an_in_flight_request_is_not_duplicated_by_a_tick`
- `format.test.ts`
  - money, percent and the MT5 broker timestamp. The MT5 stamp is UTC+3 encoded as epoch;
    `_uk()` in `pages/trading/_shared.py` is the reference and must not be re-derived.
