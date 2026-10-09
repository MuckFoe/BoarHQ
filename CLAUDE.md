# <Project name>

> **This is the baseline router that ships with the starter.** Everything in
> angle brackets is a placeholder. Run `/project-startup` to replace them, adapt
> the ruleset to this project, and fill in the Bindings section — or fill it in
> by hand. The rules below work as written; the bindings are what make them
> applicable here.

<One paragraph: what this is, who runs it, current stage. Enough that an agent
with no other context knows what it is working on and what breaking it costs.>

## Stack

- **<Component>** `<dir>/` — <language, framework, notable libraries>
- **Data** — <store, how it runs locally, ports and proxies>

## Commands

| Task | Command |
|---|---|
| Fast check (every change set) | `<build + type check>` |
| Feature check (before done) | `<fast check + full test suite + full build>` |
| Run <component> | `<...>` |

<Any command with a non-obvious prerequisite gets one line here, not a
paragraph. Flag anything the commands cannot yet do — a dormant test clause is
worth stating, e.g. "no test runner exists yet, so the feature check is
build-only and VER1's test clause is dormant.">

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

## Workflow — spec-driven

Features are built through the SDD pipeline; full text in `docs/sdd.md`.

`/sdd-specify → /sdd-plan → /sdd-tasks → /sdd-implement → /sdd-validate`

- One folder per feature: `specs/<NNN-slug>/{spec,plan,tasks,notes}.md`, and
  one index of all of them: `specs/README.md`.
- Spec is WHAT/WHY, plan is HOW. Ambiguity is marked
  `[NEEDS CLARIFICATION: ...]`, never guessed away.
- **A skill stops after writing its artifact and never invokes the next one.**
  The human runs the next `/sdd-*` command.
- State lives in files, not in conversation; each skill reads only its declared
  inputs.

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
| starting, planning, or implementing a feature | `docs/sdd.md` |
| touching `compose.yaml`, ingress, or deployment | `docs/infrastructure.md`, `docs/compose.md` |
| proposing a change to any rule | `rules/_decisions.md` |

## Bindings

Everything a rule needs to be applicable *here*. Rules are portable; bindings
are not. One heading per rule that has one, naming the rule id so the
connection is explicit. **This section is empty until you fill it in — several
rules above are inert without it.**

**Requirements** live in specs (GIT1): one file per spec at
`specs/<NNN-slug>/spec.md`, indexed in `specs/README.md`. The reference a commit
carries is the spec folder name. `REQUIREMENTS.md` is retired.

**Constitution** (SDD). `rules/*.md` and this Bindings section are the
constitution every plan is checked against. `memory/constitution.md` only points
here, and `/sdd-constitution` is not run.

**Cannot be automatically verified** (VER5). A green check says nothing about:
<list the areas — UI interactions, generated documents, rendering, anything
without an assertion behind it. If this list is empty, say so explicitly rather
than leaving the heading blank.>

**Validation levels** (SEC3). FULL: <entry points taking human input, and every
destructive or irreversible operation>. BASIC: <routine reversible programmatic
calls>.

**Declared shapes** (DAT1). <Where the authoritative definition of each
persisted entity lives, and the mechanism — schema file, type, migration.>

**Load-bearing** (ARC1). <The files that should not be refactored casually, and
why each is risky.>

**Known exposures** (SEC1). <Anything reachable beyond localhost without auth.
SEC1 requires a stated reason and a remediation before an exposure counts as
accepted — an exposure listed without both is an open finding, not an accepted
one. If there are none, say none.>

## Open

See `rules/_progress.md` for follow-ups and recorded findings, and
`rules/_decisions.md` for settled questions.
