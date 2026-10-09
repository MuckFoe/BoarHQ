# Ruleset progress

**Baseline installed, not yet adapted to this project.** Twenty-three rules
across nine domains arrived with the starter. None has been reviewed against
this codebase yet, and `CLAUDE.md`'s Bindings section is still placeholders —
several rules are inert until it is filled in.

Run `/project-startup` to adapt. Update the status line above when you do.

## Resume here

*Only while a run is paused.* The exact rule awaiting a verdict, quoted in full
so the next session does not need the transcript, then what comes after it.

*Nothing paused.*

## Domains

| # | Domain | Status | Rules |
|---|---|---|---|
| 1 | verification | baseline | VER1–5 |
| 2 | security | baseline | SEC1–4 |
| 3 | agent-conduct | baseline | CON1–3 |
| 4 | data | baseline | DAT1–2 |
| 5 | architecture | baseline | ARC1–2 |
| 6 | api | baseline | API1–2 |
| 7 | git | baseline | GIT1 |
| 8 | testing | baseline | TST1–2 |
| 9 | code-style | baseline | STY1–2 |
| — | commands | bindings only | — |
| — | project | bindings only | — |

Not covered by the baseline: performance, accessibility, observability,
release, docs. See `_decisions.md` before proposing any of them.

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

Rejections, edits with originals, and structural decisions: `_decisions.md`.

## Project findings — not rules

Real problems surfaced during elicitation that failed the portability test.
Backlog, not policy. This section is what keeps the ruleset clean — without
somewhere to put them, situational rules leak in.

*Empty — nothing elicited yet.*

## Open follow-ups

Carried in from the baseline. The first three apply to every fresh install.

- [ ] **Fill in `CLAUDE.md` Bindings.** VER5, SEC3, DAT1, ARC1 and SEC1 are
      inert without it — each names a classification that lives there, not in
      the rule.
- [ ] **Confirm the always-on set.** Nine rules load on every turn. If any does
      not earn that on this project, move it to triggered.
- [ ] **Hooks to write.** No rule here is enforced by anything but review yet:
      VER1 Stop hook + pre-push, VER3 suppression-marker grep, CON1
      destructive-command block, GIT1 commit-msg shape check. Until they exist,
      every tier line reading `hook` is aspirational — see `_decisions.md` §5.
- [ ] **`git init`** if this repository is not under version control. A ruleset
      about changing code carefully is worth much less with no undo.
- [ ] **Test runners.** If none exist, VER1's feature gate is build-only and
      TST1–2 are dormant. Say so in `CLAUDE.md`'s command table rather than
      leaving the gap silent.
- [ ] **`docs/concepts.md` is empty.** The rules already encoded in the
      codebase's most load-bearing file are its highest-value first entries.

Added 2026-10-09, when the SDD skill-pack (`docs/sdd.md`, `.claude/skills/sdd-*`)
was copied in beside the starter, and the overlaps between the two:

- [x] ~~**`REQUIREMENTS.md` vs `specs/<slug>/spec.md`.**~~ Decided 2026-10-09:
      specs are the requirements, `REQUIREMENTS.md` is retired, a commit names
      the spec folder. `_decisions.md` § Structural decisions 6.
- [x] ~~**`rules/` + Bindings vs `memory/constitution.md`.**~~ Decided
      2026-10-09: the ruleset is the constitution; `/sdd-constitution` is not
      run. `_decisions.md` § Structural decisions 7.
- [x] ~~**Inherited infrastructure has no spec.**~~ Specs 001–007 written
      2026-10-09, all `draft`. See `specs/README.md`.
- [ ] **`/project-startup` still writes `REQUIREMENTS.md`.** Its requirements
      phase and templates target the retired file. Its rule-review and Bindings
      phases are still wanted. Running it as shipped would re-create what
      decision 6 retired; the skill has not been changed.
- [ ] **What a small change references.** GIT1 accepts a one-sentence
      requirement for any change; `docs/sdd.md` says not to apply the pipeline
      to a trivial one-file fix. With `REQUIREMENTS.md` gone there is no place
      for the one sentence. Undecided.
- [ ] **Specs 001–007 await approval.** Every `[NEEDS CLARIFICATION]` marker
      was answered by the user on 2026-10-09 and none remains; the specs are
      still `draft` until signed off. None has a `plan.md`.
- [ ] **The inherited files no longer match the specs.** Answers on 2026-10-09
      moved boarHQ away from CDM's design: public instead of private-network
      only (001), self-registration with administrator approval, a one-week
      session and identity configuration kept as a file (004), daily backups
      kept 14 days (005). None of that is implemented, everything is still
      named `cdm`, and `docs/infrastructure.md` and `docs/compose.md` still say
      "never public" and "no known exposures". `specs/README.md` lists the gap
      per spec.
- [ ] **Future features named but not specified:** backups stored off the
      machine (005), operator notification over WhatsApp or Signal (007).
- [ ] **Spec terms are not in `docs/concepts.md`.** The specs use *member*,
      *operator*, *private network*, *identity provider*, *session* and
      *signed proof of identity* without an entry (VER2, STY1).
- [ ] **`README.md` and the `project-startup` references still describe
      `REQUIREMENTS.md` as live.** Left as shipped.
