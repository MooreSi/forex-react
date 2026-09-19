# 010 — The API layer and its contracts

**Money:** no **Depends on:** — **Layer:** `backend/src/api/`

## Problem

NiceGUI pages called controllers in-process. React cannot. Something has to turn a named
controller question into an HTTP response, and the obvious wrong answer is to let each router
reach for whatever it needs — which recreates, in a new directory, the boundary the restructure
pack spent two months closing.

## Decision

`backend/src/api/` is a new **top** layer with the frontend's rules verbatim:

- imports **only** `backend.src.controllers` (plus `backend.src.utils.models` for shapes);
- never imports `backend.src.db`, never writes SQL;
- never loops, merges, formats or decides — a router is `async def` around one controller call
  plus a Pydantic response model;
- 200-line ceiling per router, same as a controller. If a router would exceed it, the *controller*
  needs one coarser function.

Shape:

```
backend/src/api/
  __init__.py
  server.py          build_app() -> FastAPI; CORS off, routers mounted, bundle mounted last
  errors.py          one exception handler; a controller exception becomes a JSON problem body
  schemas/           Pydantic response models, one module per domain
  routers/
    system.py  auth.py  trading.py  chart.py  ...   one per domain, named for the controller
```

## The contracts

`tools/refactor_audit/import_contracts.py` carries two contracts whose `source_packages` is
`("frontend",)`:

- `frontend-never-imports-the-database`
- `frontend-reaches-the-backend-through-controllers`

After task 070 there is no Python under `frontend/`. `violations_for` only raises when the
directory is *missing*; a directory that exists and contains no `.py` scans zero files and
reports zero violations — green forever, protecting nothing. That is the `delegation_checker.py`
failure this repo was rebuilt after.

**So both contracts are retargeted to `backend/src/api` in this task**, before any router is
written, and renamed to say what they now guard:

- `the-api-layer-never-imports-the-database`
- `the-api-layer-reaches-the-backend-through-controllers`

Both enforced at zero, no baseline, no exemptions — there is no legacy here to absorb.

## Tests first (TDD)

Write these, watch them fail, then write the code.

- `tests/api/test_import_contracts_cover_the_api_layer.py`
  - `test_a_contract_names_the_api_layer_as_its_source` — the retargeted contracts exist and
    point at `backend/src/api`.
  - `test_no_contract_still_scans_a_directory_with_no_python_in_it` — **the negative control for
    the whole retarget**: for every contract, at least one source package contains `.py` files.
    Plant a contract scanning an empty dir and watch this go red.
- `tests/api/test_api_layer_boundary.py`
  - `test_no_router_imports_a_service_or_the_database` — AST scan over `backend/src/api/`.
  - `test_the_boundary_scan_can_see_a_violation` — negative control, against a source string.
- `tests/api/test_errors.py`
  - `test_a_controller_exception_becomes_a_json_body_not_a_stack_trace`
  - `test_the_handler_does_not_leak_the_exception_text_for_an_unexpected_error` — a broker
    rejection is surfaced verbatim (money rule); an unexpected `Exception` is not.

## What must NOT change

- No controller gains logic. A missing capability is a flat forwarding function or nothing.
- No service is touched.
