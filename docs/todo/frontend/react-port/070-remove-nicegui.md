# 070 — Remove NiceGUI

**Money:** no **Depends on:** 050, 060

## Decision

One commit removes: `frontend/**/*.py` (21,434 lines), `nicegui` from `requirements.txt` and
`pyproject.toml`, the two framework patches in `frontend/app/__init__.py`, and the `ui.run()`
call path in `run.py`.

## The tests

~40 files under `tests/frontend/` exercise NiceGUI rendering. **Golden rule 4 forbids deleting a
test to make a change pass.** It does not forbid deleting a test whose subject no longer exists —
but the bar is high, and it is met only when all three hold:

1. The subject is deleted in the **same commit**.
2. Every behaviour the test pinned has a named twin in the new suite, and that twin has been run
   green. Not "is covered by" — named, file and test function.
3. The commit message lists each deleted file with its twin.

`tests/frontend/test_no_live_network.py`, `test_no_silent_excepts.py` and the structural scans are
**not** NiceGUI tests wearing a frontend directory name. They are repo-wide rules that happen to
live there. Retarget them; do not delete them.

## Ratchets and gates

- `structure_baseline.json` — remove the deleted files. Removing an entry shrinks the baseline,
  which is allowed. Nothing new may enter it.
- `orphan_modules` — every new `backend/src/api/` module that nothing imports yet needs its
  allowlist entry with a reason, in the same change.
- Coverage ratchet — deleting 21,434 lines of measured Python moves the denominator. Recompute
  and record the new floor **with the reason in the commit**, because a ratchet that drops
  without an explanation is indistinguishable from one that was lowered to get green.
- `python -m tools.checks all` green before the commit, not after.
