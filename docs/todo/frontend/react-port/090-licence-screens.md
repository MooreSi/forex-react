# 090 — The licence screens are still NiceGUI

**Money:** no **Licence surface: yes** **Depends on:** 070 **Status:** open

## What is left

`backend/src/config/licence/guard.py` renders two screens with NiceGUI:

| Function | What it is |
|---|---|
| `_show_error_and_exit` | A licence error page, then `sys.exit(1)`. ~15 lines of UI. |
| `_show_registration_page` | The activation screen: machine ID with copy-to-clipboard, nickname and email, a registration request that polls for delivery on a 2s timer, a manual activation code path, a 1s poll that redirects once the licence lands, and the remote-agent startup that lets an already-known machine receive a re-signed key without the user doing anything. ~280 lines. |

Both run **before** the main app, in their own `ui.run()` on the app port. They
are the reason `nicegui` is still in `requirements.txt` and the reason the
`no-nicegui-in-the-backend` contract still carries 2 baselined violations.

## Why this was not done with the rest

It is a licence surface, it is the longest interactive flow left in the repo,
and the failure mode of getting it half right is that somebody who has paid for
the app cannot get into it. CLAUDE.md forbids adding a licence bypass "even for
testing", which rules out the shortcut of stubbing the screen and letting the
app boot. Rushing it at the end of the port session would have been the wrong
trade, so it was left whole and named here rather than left whole and unmentioned.

## Decision (proposed, not yet taken)

Serve both screens from a **small dedicated FastAPI app with server-rendered
HTML** — not React. Reasons:

- They run before the dashboard exists and must work on an install whose
  `frontend/dist` may be absent or stale.
- They are two forms and two pollers. A React bundle for that is a build step
  in front of the thing that decides whether the app may run at all.
- `_activation_probe` and `/licence-activated` are already plain FastAPI routes
  registered on `_ng_app` — only the page rendering is NiceGUI.

## Tests first (TDD)

- `tests/config/licence/test_activation_screen.py`
  - `test_the_screen_shows_this_machines_fingerprint`
  - `test_a_manual_activation_code_that_does_not_verify_is_rejected_with_its_reason`
  - `test_a_verifying_code_is_stored_and_the_app_is_released`
  - `test_the_screen_cannot_release_the_app_without_a_licence` — **the negative
    control that matters.** Prove the release path is reachable only through a
    verified licence, not through any request the screen can be made to send.
- `tests/refactor/test_no_nicegui_in_the_backend.py`
  - `test_the_contract_is_at_zero` — flip `no-nicegui-in-the-backend` to
    `enforced_at_zero=True` as the last step, and delete `nicegui` from
    `requirements.txt` and `pyproject.toml` in the same commit.

## Until then

`nicegui` stays in `requirements.txt`, with the comment there saying exactly
why. An install that never hits the licence path never loads it.
