# Spec-Driven Development (SDD)

Copied into boarHQ on 2026-10-09 from the `sdd` skill-pack, together with
`.claude/skills/sdd-*`. The pack is v1: unreviewed, and not yet dry-run on a
real feature.

Three things differ here from the text below (decided 2026-10-09,
`rules/_decisions.md` § Structural decisions 6–7):

- Specs replace the starter's `REQUIREMENTS.md`, which is retired.
- The constitution is `rules/*.md` plus the Bindings in `CLAUDE.md`.
  `memory/constitution.md` only points there and `/sdd-constitution` is not run.
- `specs/README.md` indexes every spec; `/sdd-specify` adds a row to it.

## Pipeline

`constitution → specify → plan → tasks → implement → validate`

| # | Skill | Reads | Writes | Human gate |
|---|---|---|---|---|
| 0 | `/sdd-constitution` | (interview) | `memory/constitution.md` | approve principles — run once, rarely revisited |
| 1 | `/sdd-specify` | constitution | `specs/<slug>/spec.md` | resolve every `[NEEDS CLARIFICATION]`, approve |
| 2 | `/sdd-plan` | spec.md, constitution | `specs/<slug>/plan.md` | approve architecture |
| 3 | `/sdd-tasks` | plan.md, spec.md | `specs/<slug>/tasks.md` | approve/reorder task list |
| 4 | `/sdd-implement` | one task | code + tests, task ticked off | approve diff, per task |
| 5 | `/sdd-validate` | spec.md, tasks.md, test run | `specs/<slug>/notes.md` | sign off feature |

**Hard rule:** a skill stops after writing its artifact. It never invokes the next skill itself. No exceptions, no "since it looks obviously right." The human runs the next `/sdd-*` command.

## Why this order

- Spec = WHAT/WHY. Plan = HOW. Never mix them — implementation detail in a spec is the single most common SDD failure mode across every source reviewed.
- Ambiguity gets marked (`[NEEDS CLARIFICATION: ...]`), never guessed away.
- Tasks carry acceptance criteria traceable back to the spec's Given/When/Then, not invented ad hoc.
- Tests are written from acceptance criteria before implementation (TDD), not after.
- The spec is the durable record. Code and tests are regenerable from it; when behavior changes, the spec changes first.

## Repo layout

```
memory/constitution.md          # project-wide, one-time, rarely edited
specs/<NNN-feature-slug>/
  spec.md                       # WHAT/WHY — stable
  plan.md                       # HOW — architecture, data model, contracts
  tasks.md                      # execution delta — per-task checklist
  notes.md                      # validation log, drift/learnings, appended each cycle
```

One folder per feature. Never one monolithic spec for the whole product.

## Context management — non-negotiable

State lives in files, not in conversation. This is what makes the gates real: a human can `/clear` or close the session between every phase with zero loss, because the next skill reconstructs everything it needs from disk.

- **Each skill reads only its declared inputs** — its upstream artifact(s) + constitution. Never the whole `specs/` tree, never unrelated features, never "the rest of the chat" for facts that belong on disk.
- **Isolate features.** Working feature B must not load feature A's spec/plan/tasks into context.
- **Size tasks to a context window.** A task that touches more than ~3 files or mixes concerns is a planning bug — split it in `/sdd-tasks`, don't let `/sdd-implement` discover the overflow mid-task.
- **One task (or one `[P]` wave) per implementation pass.** Prefer a fresh Agent/session per task. Never carry task N's exploration noise into task N+1 — the handoff is `tasks.md` + `spec.md` on disk, not memory.
- **Split specs before they grow unbounded.** If a spec needs a table of contents, it should have been two features.

## Do / Don't (cross-source synthesis)

**Do:** write specs collaboratively; use EARS (WHEN/THEN) or Given/When/Then for acceptance criteria; keep plan/tasks separate from spec; calibrate spec detail to risk; keep everything under version control; require human approval at every gate; run a second, fresh-context review pass before calling a feature done.

**Don't:** put tech stack or file structure in the spec; resolve ambiguity by guessing; over-specify into pseudo-code or under-specify into "should work well"; implement a whole feature in one pass; let agent self-review replace human review; skip re-validating the spec after implementation surfaces new facts; apply this ceremony to a trivial one-file fix.
