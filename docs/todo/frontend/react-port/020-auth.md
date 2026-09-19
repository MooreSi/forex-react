# 020 — Auth: session cookie and the login gate

**Money:** no **Depends on:** 010 **Layer:** `backend/src/api/`

## Problem

`frontend/auth_gate.py` is the only thing between someone at the keyboard and the trading
controls. It works through `app.storage.user` — a NiceGUI signed session cookie — and a Starlette
middleware that redirects unauthenticated requests to a NiceGUI `/login` page. Both halves die
with NiceGUI.

## Decision

Port it, do not redesign it. Same three behaviours, same default:

1. **`auto_login_enabled` defaults to False.** An install that never touched the setting keeps
   asking. Unreadable config must not open the door — the `except: return False` in
   `_may_pass` is load-bearing and stays.
2. **Open prefixes stay open.** `/login`, `/static`, `/favicon`, and now the bundle's own assets
   — the login page cannot render otherwise. `/_nicegui` goes.
3. **First-run setup.** `auth_controller.needs_setup()` still short-circuits to the one-time
   password setup instead of a login form that could never succeed.

The signed cookie is now itsdangerous over the same per-install `storage_secret` from
`run.py:_dashboard_storage_secret`, so an existing install's secret file keeps working.

The React side gets a `/login` route and an `AuthContext`; a 401 from any endpoint sends the user
there. The API returns **401 JSON**, never a redirect — a redirect to an HTML login page inside
an XHR is how a session expiry becomes "the dashboard renders the login form inside the trading
panel".

## Tests first (TDD)

- `tests/api/test_auth_gate.py`
  - `test_an_unauthenticated_request_to_a_trading_endpoint_is_401_not_a_redirect`
  - `test_the_login_route_is_reachable_without_a_session`
  - `test_auto_login_off_is_the_default_for_an_install_that_never_set_it`
  - `test_unreadable_config_does_not_open_the_door` — config raises; still gated.
  - `test_a_correct_password_sets_a_session_that_the_next_request_accepts`
  - `test_a_wrong_password_sets_no_session` — negative control.
  - `test_first_run_offers_setup_instead_of_a_login_that_cannot_succeed`

`tests/frontend/test_auth_gate.py` and `test_auto_login_option.py` cover the NiceGUI versions of
these. They stay green until 070 deletes their subject, and each assertion above has a twin
there — check it before deleting.
