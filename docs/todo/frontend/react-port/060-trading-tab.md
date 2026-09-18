# 060 — Trading tab

**Money: YES.** **Depends on:** 040 **Ships alone.**

## Read first

`.claude/skills/safe-change/SKILL.md`, `docs/system/rules/20-trading-safety.md`, and
frontend-conventions §8. This tab carries manual entry, position close, partial close and
strategy overrides. It is the button.

## Scope

`frontend/pages/trading/` is 3,678 lines across ten modules: strategy cards, active trades,
manual entry, pending signals, Telegram signals, the schedule, EA templates, the signals card.
It reaches `trading_controller`, `sync_controller`, `broker_controller` and `schedule_controller`.

## Decision

- **No order logic moves into the API layer.** A router endpoint is one controller call. The
  backend decides whether an order is allowed; the router forwards the answer, including the
  rejection text, verbatim.
- **Confirmation is a React dialog that names instrument, direction and size** — not "Are you
  sure?", and never defaulting to yes. Same bar as the NiceGUI dialog it replaces.
- **The disabled reason is rendered.** A greyed Execute button with no explanation is
  indistinguishable from a broken one: *trading paused*, *bridge disconnected*, *stood down as
  Remote*, *out of hours*.
- **Demo vs live is unmistakable** in the header badge before this tab ships.
- Read endpoints and write endpoints are separate routers. A GET that can place an order is one
  browser prefetch away from an order nobody asked for.

## Tests first (TDD)

- `tests/api/routers/test_trading_reads.py` — forwarding + shapes, as 050.
- `tests/api/routers/test_trading_orders.py` — **no test in this file may reach a broker.**
  Module docstring says so and a guard test proves it, in the style of
  `test_bridge_process_relocation.py::test_no_test_in_this_file_can_spawn_a_process`.
  - `test_placing_an_order_forwards_the_exact_arguments_the_controller_was_given`
  - `test_a_rejection_reaches_the_client_verbatim`
  - `test_an_order_endpoint_is_not_reachable_by_GET`
  - `test_an_unauthenticated_order_request_is_401`
- `PlaceOrderDialog.test.tsx`
  - `test_the_confirmation_names_instrument_direction_and_size`
  - `test_confirm_does_not_default_to_yes`
  - `test_execute_is_disabled_with_the_reason_shown_when_the_backend_says_no`

## Sign-off

Green tests are not sign-off for this task. It needs the owner plus a demo session watching a
trade open and close through the React UI, recorded in PROGRESS.md before the row flips to done.
