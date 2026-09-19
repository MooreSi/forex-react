# 090 — The licence screens

**Status:** done (2026-09-18) **Money:** no **Licence surface: yes**

`nicegui` is no longer a dependency of this project. `no-nicegui-in-the-backend`
is enforced at zero.

## What was here

`backend/src/config/licence/guard.py` rendered two screens with NiceGUI, both
running **before** the app starts, in their own `ui.run()` on the app port:
a licence error page, and a ~280-line activation screen (machine ID, nickname,
email, a registration request that polled for delivery, a manual activation
code, a 1s poll that redirected once a licence landed, and the remote-agent
startup that lets an already-known machine receive a re-signed key on its own).

## What replaced it

Two modules, split along the line that made the original untestable:

- **`config/licence/activation.py`** — every decision, and no web framework:
  validate what was typed, verify a key against this machine, describe what the
  remote client is actually doing. 19 tests.
- **`config/licence/activation_server.py`** — one HTML document and the routes
  around it. No build step, no bundle, no dependency on `frontend/dist`
  existing. 20 tests.

**Plain HTML rather than React, deliberately.** These run before the app and are
the only way back into a stranded install. A licence screen that needs the
dashboard to have been compiled is a licence screen that cannot rescue a broken
install — so `test_the_form_renders_with_no_bundle_and_no_database` is the first
test in the file.

## What the tests hold

- **Nothing is stored unless the Ed25519 signature verifies against this
  machine**, and it is the same check `enforce()` makes rather than a second
  opinion. `test_no_route_here_can_grant_a_licence` drives every route with
  verification refusing and asserts nothing was written.
- **The screen does not claim a request was sent when it was only queued.** On
  2026-08-07 a user was told their registration was "awaiting approval" while
  the admin server was down, so nothing had been transmitted and nothing ever
  arrived. `describe_delivery` reports what the client actually did.
- **The admin machine starts its server and not its client**, because it does
  not connect to itself; a known client connects so a pushed licence can land;
  a first install starts neither.
- **A failing agent does not take the screen down.** It is the only way back in.

## One bug the port found

The wait page polls a probe path and redirects to the app the moment it 404s —
that 404 IS the signal that the restart happened. The React app's SPA fallback
answered **200** for it, because the path is not under `/api/`. The operator
would have sat on a "restarting" page for ever. `ACTIVATION_PROBE` is now in the
fallback's exclusion list, defined in `config/` and imported by `api/` (that
direction: `config/` is the bottom of the import stack), and
`test_the_probe_is_not_registered_by_the_real_app` asserts both halves.
