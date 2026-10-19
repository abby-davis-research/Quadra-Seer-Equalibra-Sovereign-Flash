# Debug pass (parity port) — October 18, 2026 (Melody, for Anthony via Abby)

This change ports, file-for-file, the fixes from
quadra-seer-equalibra-sovereign PR #7 (merged there the same day) to this
repo. The four files below were verified byte-identical to that repo's
pre-fix versions before the fixed versions were copied over — no other
differences exist between the two codebases in these files.

## Test results (this repo)
- Python suite (pytest, 33 files, run from `app/`):
  - As received: **259 passed, 0 failed, 17 warnings.**
  - After this port: **259 passed, 0 failed, 0 warnings.**
- Not run here: frontend suite, Docker image builds, and the backend boot
  smoke test (all verified on the sovereign repo in PR #7; this repo's
  Python suite is the same 259 tests).

## Fixes applied (4 files touched, no behavior changes intended)
1. `atlantean_core/identity.py`, `atlantean_core/sync.py` (x2),
   `atlantean_quadra_bridge.py` — replaced deprecated `datetime.utcnow()`
   (scheduled for removal from Python) with a timezone-aware computation that
   produces byte-identical naive-UTC strings, so stored formats and the sync
   protocol are unchanged. The bridge call site also had an eager-default bug
   (`.get('timestamp', <now>)` computed the fallback on every call); it now
   only computes the timestamp when one is actually needed.
2. `atlantean_backend.py` — the no-Redis session fallback used flask-session's
   deprecated built-in 'filesystem' interface (removal announced). Migrated to
   the supported CacheLib backend (`SESSION_TYPE='cachelib'` +
   `SESSION_CACHELIB=FileSystemCache(...'flask_session_cache')`). Note: the
   fallback now stores session files in `flask_session_cache/` beside the
   backend file (was `./flask_session`); existing fallback sessions do not
   carry over (they are ephemeral).

## Net effect
Test-suite warnings went from 17 to 0, matching the sovereign repo.
Keeping these two repos in lockstep matters: Flash is a copy of the same
codebase, so fixes landed in one should be ported to the other.
