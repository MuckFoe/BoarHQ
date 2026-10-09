# Ruleset progress

Run started 2026-08-13 via `/project-startup`, completed 2026-08-14.
**Elicitation closed.** All 10 domains resolved · 23 rules accepted · 2 rejected
· 1 recorded as a finding · 9 substantially rewritten by the user.

Remaining work is implementation, not elicitation.

## Domains

| # | Domain | Rules |
|---|---|---|
| 1 | verification | VER1–5 |
| 2 | security | SEC1–4 |
| 3 | agent-conduct | CON1–3 |
| 4 | data | DAT1–2 |
| 5 | commands | none — bindings only |
| 6 | architecture | ARC1–2 |
| 7 | api | API1–2 |
| 8 | git | GIT1 |
| 9 | testing | TST1–2 |
| 10 | code-style | STY1–2 |

Skipped by decision: performance, accessibility, observability, release, docs.

## The ruleset

| ID | Rule | Always-on |
|---|---|---|
| VER1 | "Done" is scoped — fast check per change, full check per feature | ● |
| VER2 | Features record the concepts they introduce | |
| VER3 | Never make the check pass by weakening it | ● |
| VER4 | One failed fix, then stop | ● |
| VER5 | Name what the check cannot cover | |
| SEC1 | No unauthenticated CRUD beyond localhost | |
| SEC2 | Internal error detail stays server-side | |
| SEC3 | Validation depth follows source and criticality | |
| SEC4 | Secrets never enter the repository or the transcript | ● |
| CON1 | Destructive data operations are confirmed first | ● |
| CON2 | Gaps in project context are recorded, not worked around | ● |
| CON3 | Change what the task requires, nothing else | ● |
| DAT1 | Stored shapes are declared, not implied (opaque counts) | |
| DAT2 | Derived data is never the source of truth | |
| ARC1 | Follow the established pattern, or agree a new one | ● |
| ARC2 | Dependencies flow one way | |
| API1 | Breaking changes ship as a new version | |
| API2 | Every capability exists through the API | |
| GIT1 | Commits state intent and name their requirement | ● |
| TST1 | A test is seen failing before it is made to pass | |
| TST2 | Test what can break, not what exists | |
| STY1 | One term per concept | |
| STY2 | Comments carry what the code cannot | |

Rejections, the user's edits with originals, and structural decisions:
`_decisions.md`.

## Project findings — not rules

Real issues in this codebase. Too situational for the portable ruleset.
Backlog, not policy.

- **`SyncFromClient` infers deletion from absence.** Deletes every action in
  the database and reinserts from the request body, then deletes any team or
  game whose id is missing from the payload. A partial, stale, or
  half-serialized client payload is therefore a silent mass delete that reports
  success. Written to mirror the v36 browser `db` object, now declared dead
  weight — the semantics serve a client that no longer exists.
- **Stale legacy comments.** `store.go` describes `Snapshot` as matching "the
  legacy HTML export/import shape" and `SyncFromClient` as mirroring the
  browser `db` object. Both are now misleading.
- **`bober_statistics_v36.html` (6147 lines) and `boberstatistics_fixed.json`
  (175 KB)** sit in the repo root with no live dependency. Archive candidates.
- **Every handler returns `err.Error()` to the client.** SEC2 makes this a real
  refactor of `handlers.go`, not a free policy adoption.
- **List endpoints are unbounded.** Actions, teams and games return everything.

## Open follow-ups

- [ ] **`git init`** — repository still not under version control. Highest
      value item here; 23 rules about changing code carefully, and no undo.
- [ ] **SEC1 known exposure is unresolved** — deployed with no auth on any
      endpoint, including two that delete data wholesale. SEC1 requires a
      stated reason and a remediation before an exposure counts as accepted.
      Neither given. Recorded in `CLAUDE.md` as open, not accepted.
- [ ] **Test runners do not exist** (Go: 0 test files; frontend: no runner).
      VER1's feature gate is build-only and TST1–2 are dormant until they do.
- [ ] **Hooks to write** — VER1 Stop hook + pre-push, VER3 suppression-marker
      grep, CON1 destructive-command block, GIT1 commit-msg shape check.
- [x] ~~Update skill `references/templates.md`~~ — done 2026-08-14. Now
      documents the portability split, compact form with one-line why, and the
      always-on/triggered axis.
- [ ] `docs/concepts.md` is empty. The rules encoded in
      `VolleyballMatchModel.ts` are its highest-value first entries.
- [x] ~~**Two copies of the skill exist**~~ — resolved 2026-08-27 by restructuring
      this repository into a single starter tree. The nested `project-starter/`
      copy is gone and `.claude/skills/project-startup/` is the only copy. The
      23 rule files this run produced became the shipped baseline at `rules/`;
      these run artifacts moved to `examples/boberstatistics/`.
