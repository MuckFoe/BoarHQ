---
domain: verification
reviewed: 2026-08-13
---

# Verification rules

## VER1 — "Done" is scoped
Two gates. **Per change set:** the project's fast check (build + type check)
runs and its output is shown. **Per completed feature:** the fast check plus
the full test suite and full build. A feature is not done on the change-set
gate alone.
**Why:** fast feedback on every edit, real proof at the boundary that matters —
a check that is too slow to run becomes a check that is skipped.
→ *Stop hook for the fast gate; pre-push for the feature gate*

## VER2 — Features record the concepts they introduce
Any domain concept a feature introduces or changes is recorded in the project's
concepts document — one accumulating file, never one per feature. An entry is
the term, what it means in this system, and any invariant that must hold. A
feature that introduced no new concept adds nothing; that is a valid outcome.
**Why:** domain rules encoded only as code are unreadable to everyone including
their author six months later; per-feature files are written once and never
opened again.
→ *review*

## VER3 — Never make the check pass by weakening it
Fix the cause, not the signal. No compiler-directive ignores, linter opt-outs,
silenced warnings, discarded errors, or deleted and skipped tests. Where
suppression is genuinely correct, the agent stops and requests sign-off stating
what and why — it does not suppress and annotate.
**Why:** a gate reporting green while the defect stands is worse than no gate,
because you stop looking exactly when you should be looking.
→ *hook flags suppression markers in diffs; sign-off is a human decision*

## VER4 — One failed fix, then stop
When a fix does not resolve the failure it targeted, the agent stops and
requests review rather than attempting a second fix for the same failure. The
report states the original failure, what was changed and why that was expected
to work, what happened instead, and the current best explanation. A *different*
failure surfacing is not a failed fix — that is progress, and work continues.
**Why:** a failed fix means the model of the problem is wrong, and a second
attempt built on the same wrong model compounds it while polluting context.
→ *advisory; optional Stop hook counting consecutive failing checks*

## VER5 — Name what the check cannot cover
Areas whose correctness the automated check cannot establish are listed in
CLAUDE.md. When a change touches one, the agent states that it cannot verify
its own work there and names what a human must look at. A green check is never
reported as validation of an area the check does not cover.
**Why:** the dangerous case is not a failing check but a passing one that never
examined what changed — green then reads as validated.
→ *review; the list of uncovered areas lives in CLAUDE.md*
