# Specs

Every spec in this project, one row each. A spec is one file:
`specs/<NNN-slug>/spec.md`. Its `plan.md`, `tasks.md` and `notes.md` are added
beside it as the feature moves through the pipeline in `docs/sdd.md`.

Specs are where requirements live. `REQUIREMENTS.md` was retired on 2026-10-09;
a commit names the spec folder its change serves (GIT1).

## Viewer

`specs/index.html` shows everything here in one place: this index, each spec's
artifacts, requirements paired with their acceptance criteria, task progress.
It holds no copy; it reads the markdown each time it loads. From the root of
the repository:

```
python specs/serve.py
```

then open <http://127.0.0.1:8777/>. A different port goes after the file name.

**Progress.** The *Progress* view (`#progress`) lists every spec with every
task, above three charts: tasks done and open per spec, how many specs have
each artifact, and tasks per wave. One row of filters — text, status, artifact,
open or done, only `[P]` — narrows the charts and the list together; clicking a
bar in the artifact chart sets that filter. All of it is counted from the
markdown on each load. A task's number, wave, `[P]` flag and requirement ids
are read from `tasks.md`: the checklist line, a table with *Wave* and *Tasks*
columns, and the `**Spec:**` line under the heading that starts with the task's
number.

**Editing.** Every file the viewer shows has an **Edit** button: this index,
and each spec's `spec.md`, `plan.md`, `tasks.md` and `notes.md`. It opens the
raw markdown; **Save** (or Ctrl+S) writes the whole file back. In a `tasks.md`
the checkboxes can be ticked directly. What to know before relying on it:

- A save writes straight to the working tree. Nothing is committed; `git diff`
  shows what changed, and git is the undo.
- If the file changed on disk after the page read it — an agent or an editor
  got there first — the save is refused and nothing is written. Reload, then
  apply the change again.
- It edits files that exist. It does not create, rename or delete one; the
  `/sdd-*` skills create the artifacts.
- It does not know the pipeline. Changing a spec does not mark its plan or
  tasks stale, and changing a status here is the human moving it.
- Task checkboxes are list items — `- [ ] **1** — …` — not headings.

`specs/serve.py` is what saves: Python standard library, bound to `127.0.0.1`,
and it accepts writes only to those markdown files and only from this page.
Under any plain static server (`python -m http.server`, run inside `specs/`)
the page still works, read-only, and says so in its header.

The page loads its markdown renderer from cdnjs, so it needs network access.

## Index

| # | Spec | What it requires | Status | Open questions |
|---|---|---|---|---|
| 001 | [public-access](001-public-access/spec.md) | Reachable from the public internet at one hostname, over HTTPS; nothing published on the host; everything named `boarhq`, no trace of the inherited project | draft | 0 |
| 002 | [single-hostname-routing](002-single-hostname-routing/spec.md) | Every service under one hostname by path; adding one disturbs none | draft | 0 |
| 003 | [authentication-gate](003-authentication-gate/spec.md) | Every request authenticated at the entry point; the API re-checks a signed proof | draft | 0 |
| 004 | [delegated-identity](004-delegated-identity/spec.md) | The application never handles a credential; a dedicated identity provider does; self-registration with administrator approval; configured from a file in the repo | draft | 0 |
| 005 | [persistence](005-persistence/spec.md) | State survives stop, restart and rebuild; application and identity data are separately resettable; backed up daily on the same machine, kept 14 days | draft | 0 |
| 006 | [secrets-and-configuration](006-secrets-and-configuration/spec.md) | Secrets come from deploy-time configuration only; a missing one stops the start | draft | 0 |
| 007 | [startup-and-recovery](007-startup-and-recovery/spec.md) | Services start in a working order and restart after a crash | draft | 0 |

**Status** is one of `draft` (written, not approved), `approved` (every
`[NEEDS CLARIFICATION]` resolved and the spec signed off), `planned`, `in
progress`, `validated`. Only the human moves a spec out of `draft`.

Adding a spec means adding its row here. Numbers are never reused.

## Where these seven came from

Specs 001–007 were written on 2026-10-09 **after** the thing they describe. The
infrastructure was designed for CDM and merged into boarHQ without a spec, so
these record what the inherited files are meant to do, read back out of
`compose.yaml`, `deploy/` and `docs/compose.md`. Two consequences:

- Every "why" traces to a CDM decision, not to a boarHQ one. Each spec's open
  questions ask whether boarHQ wants the same thing.
- None has a `plan.md`. The existing files are the de facto plan and were never
  put through `/sdd-plan`.

Nothing here has been run. The stack does not start — the two application
services it builds do not exist — so no acceptance criterion has been checked.

## What already exists for each

Not part of any spec; a spec states what must hold, not how.

| # | Existing files | Gap against the spec |
|---|---|---|
| 001 | `compose.yaml` (ingress sidecar, no `ports:`), `deploy/tailscale/cdm.json` | **FR1 is not met.** The existing files publish to the private network only; `CLAUDE.md`-era docs (`docs/infrastructure.md`, `docs/compose.md`) still say "never public" |
| 002 | `deploy/caddy/Caddyfile` | The two services it routes to do not exist. A Caddyfile comment and `docs/compose.md` §10 still advise a separate hostname for services that cannot run under a path; the spec rules that out |
| 003 | `deploy/caddy/Caddyfile`, `compose.yaml` (`authproxy`) | FR7 is a requirement on an API that does not exist |
| 004 | `compose.yaml` (`auth`, `db-auth`) | **FR4–FR8 are not met.** No identity-provider configuration file exists; registration, approval and session length are configured nowhere |
| 005 | `compose.yaml` (named volumes, two databases) | **FR5–FR7 are not met.** No backup or restore exists |
| 006 | `.env.example`, `.gitignore`, `compose.yaml` (`:?` variables) | See findings below |
| 007 | `compose.yaml` (`depends_on`, `restart`, health checks) | — |

## Findings while writing these

Reported, not fixed.

- **`.env.example` contradicts the authentication design.** Its comment says the
  session-checking component is "only used by FUTURE services" and that web and
  API "authenticate directly" against the identity provider. `compose.yaml`,
  the Caddyfile and `docs/compose.md` all say it gates every request. The
  comment is stale.
- ~~**CDM application settings remain** in `.env.example` and the API's
  environment block.~~ Removed 2026-10-09 (spec 006).
- **Not specified at all:** the web frontend and the API themselves. They are
  application features, not infrastructure, and nothing says what boarHQ's are.
