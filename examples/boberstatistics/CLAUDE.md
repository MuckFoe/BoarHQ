# BoberStatistics

Volleyball scouting and match statistics. Teams → Games → Actions, where an
action is a typed event (`type`, `set`, `side`, `playerId`, `value`). Live
scouting UI, analytics aggregation, PDF match reports. Personal project,
deployed to a server. Domain vocabulary is German inside English code — keep
the German (STY1).

## Stack

- **Backend** `backend/` — Go 1.22, chi, MongoDB driver
- **Frontend** `frontend/` — React 18, TypeScript 5.3 (strict), Vite 4;
  zustand, react-query, recharts, jspdf
- **Data** — Mongo 7 via docker-compose. Vite proxies `/api` → `127.0.0.1:8080`

## Commands

| Task | Command |
|---|---|
| Fast check (every change set) | `cd backend && go build ./... && go vet ./...` then `cd frontend && npx tsc --noEmit` |
| Feature check (before done) | fast check + `go test ./...` + `npm run build` |
| Run backend | `cd backend && go run ./cmd/server` |
| Run frontend | `cd frontend && npm run dev` |
| Database | `docker compose up -d mongo` |

⚠ No test runner exists in either half yet. Until one does, the feature check
is build-only and VER1's test clause is dormant.

## Always

Apply on every turn. Full text in the linked file.

- **VER1** — done is scoped: fast check per change set, full check per feature → `rules/verification.md`
- **VER3** — never make the check pass by weakening it; suppression needs sign-off → `rules/verification.md`
- **VER4** — one failed fix, then stop and request review → `rules/verification.md`
- **CON1** — destructive data operations are confirmed first, locally included → `rules/conduct.md`
- **CON2** — gaps in project context are recorded, not worked around → `rules/conduct.md`
- **CON3** — change what the task requires, nothing else → `rules/conduct.md`
- **SEC4** — secrets never enter the repo, logs, or agent output → `rules/security.md`
- **GIT1** — every change is driven by a requirement; the commit names it → `rules/git.md`
- **ARC1** — greenfield architecture is discussed and agreed before writing → `rules/architecture.md`

## Rules

Read before working in the area. Not loaded automatically.

| When you are... | Read |
|---|---|
| completing a feature | `rules/verification.md` — VER2, VER5 |
| writing or changing tests | `rules/testing.md` |
| touching HTTP handlers or the API surface | `rules/security.md`, `rules/api.md` |
| changing what is persisted | `rules/data.md`, `rules/security.md` — SEC3 |
| adding a module or crossing a boundary | `rules/architecture.md` |
| naming or commenting | `rules/style.md` |
| looking up a domain term | `docs/concepts.md` |
| proposing a change to any rule | `rules/_decisions.md` |

## Bindings

**Requirements** live in `REQUIREMENTS.md` (GIT1).

**Cannot be automatically verified** (VER5). A green check says nothing about:
PDF report output (`frontend/src/reports/matchReportPdf.ts`), the live scouting
court UI and its interactions, keyboard shortcut behavior, chart rendering.

**Validation levels** (SEC3). FULL: `/api/import`, `/api/sync`, and all deletes.
BASIC: team and game upserts, action append.

**Load-bearing.** `frontend/src/scouting/VolleyballMatchModel.ts` (1254 lines)
— the rotation and rally engine, and the highest-risk file in the repository.

**Known exposure** (SEC1) — ⚠ **Unresolved, not accepted.** The service is
deployed with no authentication on any endpoint, including `/api/import` and
`/api/sync`, which both delete data wholesale. SEC1 requires a stated reason
and a remediation before an exposure counts as accepted; neither has been
given. Treat this as an open finding.

## Open

See `rules/_progress.md` for follow-ups and recorded findings, and
`rules/_decisions.md` for settled questions.
