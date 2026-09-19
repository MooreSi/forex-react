# 030 — The React build and how it is served

**Money:** no **Depends on:** 010 **Layer:** `frontend/`, `run.py`

## Decision

**Vite + React 19 + TypeScript + Tailwind + shadcn/ui.** Not Next.js: SSR, CDN delivery and
code-splitting are the benefits the 2026-08-06 rejection correctly said do not apply to a
single-user localhost dashboard, and Next.js costs a Node server process to get them. Vite
compiles to static files that the FastAPI process already running serves.

```
frontend/
  package.json  vite.config.ts  tsconfig.json  tailwind.config.ts  index.html
  src/          api/ components/ contexts/ hooks/ pages/ lib/
  dist/         committed build output — see QUESTIONS Q4
```

`frontend/src/**` follows `.claude/skills/frontend-conventions/` (the React one): the eight file
suffixes, `components/<domain>/`, `internal/` and `hooks/` only once earned, the 150/250/400 LOC
tiers, `DialogShell` / `PanelShell` / `format.ts` created once.

**Serving.** `run.py` stops calling `ui.run()` and calls `uvicorn.run(build_app(), ...)`. Same
port, same host resolution, same `_claim_port` call immediately before binding — that ordering is
load-bearing and its comment explains why. `ws_ping_*` and `reconnect_timeout` go: they tuned
socket.io, and there is no socket.io any more. The polling cadence that replaces them is task 040.

`build_app()` mounts routers **first** and the bundle **last**, with an SPA fallback that returns
`index.html` for any unmatched non-`/api` path so client-side routes survive a refresh. A
fallback mounted before the routers swallows `/api/*` and every endpoint returns HTML — verify it,
do not assume it.

## Tests first (TDD)

- `tests/api/test_static_bundle_is_served.py`
  - `test_the_index_is_served_at_the_root`
  - `test_an_unknown_client_route_falls_back_to_the_index`
  - `test_an_unknown_api_path_is_404_json_not_the_index` — the mount-order bug, pinned.
  - `test_the_app_still_builds_when_the_bundle_is_missing` — a source checkout with no `dist/`
    must start and say so, not crash on mount.
- `tests/refactor/test_the_committed_bundle_is_not_stale.py`
  - `test_dist_exists_and_names_the_entry_the_index_loads` — cheap staleness smoke. The real
    check is CI rebuilding it (Q4).
