# Ruleset decisions

Settled questions. Read before proposing new rules — do not re-litigate these.

Entries marked **(baseline)** came with the starter and explain why the shipped
rules have the shape they do. Entries you add during `/project-startup` or later
sit alongside them. Keep both: the baseline reasoning is what stops a future run
"simplifying" a rule back into the form it was deliberately moved away from.

## Rejected

| Proposed rule | Reason | Date |
|---|---|---|
| *(baseline)* `cmd/1` — use the declared command rather than an inferred invocation | Redundant with the `CLAUDE.md` command table itself. A rule telling the agent to read a table it already loads is noise. The residual value ("ask when a command is missing") generalised into CON2 instead. | 2026-08-13 |
| *(baseline)* `git/1` — one logical change per commit | Commit *timing* does not need a rule. VER1 already defines when a feature is done, and commit timing follows from it: feature done → commit → PR if needed. Git gets exactly one rule, and it is about message quality. | 2026-08-14 |

**Deliberately not proposed in `git`** *(baseline)*, and available to reopen:
"don't commit or push unless asked" (already default agent behavior), "don't
rewrite published history" (covered by CON1), "never commit secrets" (covered
by SEC4).

**Domains not covered by the baseline** *(baseline)*: `performance`,
`accessibility`, `observability`, `release`, `docs`. Left out because for most
projects they produce generic rules, and generic rules dilute the real ones.
Propose them only where there is evidence they bite — a domain with two real
rules is worth more than a domain with eight generic ones. `project` and
`commands` are covered by `CLAUDE.md` rather than by rule files.

## Deferred

| Proposed rule | Revisit when | Date |
|---|---|---|

## Edited during elicitation

Rules the baseline's author rewrote substantially before accepting. Originals
kept so the intent behind each edit stays visible — each of these is a place
where the obvious version of the rule was tried and found wanting.

| ID | As first proposed | As accepted |
|---|---|---|
| VER1 | One gate: build + type check on every change | Two gates — fast check per change set, full check including tests per completed feature |
| VER3 | Suppression allowed with a same-line comment stating why | Suppression requires the user's sign-off; the agent does not self-authorize and annotate |
| SEC1 | Auth required on endpoints that *mutate* data | Auth required on all CRUD; health probes exempt |
| SEC3 | Uniform explicit-shape validation at every trust boundary | Two graded levels — FULL for human-originated input and destructive operations, BASIC for routine programmatic calls |
| DAT1 | Every persisted entity has a declared shape | Opaque storage is itself a valid declaration; the violation is *reading into* an undeclared structure |
| ARC1 | Greenfield patterns are named and proposed | Greenfield architecture is *discussed and agreed* with the user before writing — a hard stop, not a notification |
| API1 | Breaking changes are identified and handled deliberately | Breaking changes are fine — they ship as a new version |

## Structural decisions

Decisions about how the ruleset itself is built. These outlive any individual
rule and are the most expensive things to reverse.

1. **Rules are portable.** *(baseline)* No paths, commands, or file names in
   `rules/*.md`. A rule that only bites given this codebase's design is a
   finding, not a rule. Project specifics live in `CLAUDE.md` as bindings.
2. **Compact written form.** *(baseline)* Statement + one-line why + enforcement
   tier. The one-line why earns its tokens — it lets the model generalise a rule
   beyond the mechanisms listed, and tells it when the rule does *not* apply.
   Long rationale is decision support at accept time and stays in the transcript.
3. **Split by when a rule applies, not by topic.** *(baseline)* Always-on rules
   are cited one line each in `CLAUDE.md`; triggered rules are routed on demand.
   Test: *would loading this late cause harm?*
4. **A fifth verdict, `record`.** *(baseline)* Real problems that are not
   portable rules go to the findings list in `_progress.md` rather than into the
   ruleset.
5. **The ruleset ships as a baseline, not a blank page.** *(baseline)* The nine
   domain files arrive populated and working. `/project-startup` reviews them
   against the project one at a time and elicits what is missing, rather than
   rebuilding a general-purpose ruleset from scratch in every repository. The
   one-verdict-per-rule constraint is unchanged — it now applies to keeping a
   baseline rule as much as to adding a new one.
6. **Requirements live in specs; `REQUIREMENTS.md` is retired.** (user,
   2026-10-09) The project runs the SDD pipeline in `docs/sdd.md`, and a
   per-feature spec is the requirement. GIT1's text is **unchanged** — it says
   "requirement" and names no file, so only its binding moved.
   *Binding as shipped:* "**Requirements** live in `REQUIREMENTS.md` (GIT1)."
   *Binding now:* requirements live in `specs/<NNN-slug>/spec.md`, indexed in
   `specs/README.md`; a commit references the spec folder name.
   Left open by this decision: GIT1 says a one-sentence requirement written
   first is enough for any change, while the SDD text says not to apply the
   pipeline to a trivial one-file fix. What a small change references is
   undecided — see `_progress.md`.
7. **The ruleset is the SDD constitution.** (user, 2026-10-09) `rules/*.md` plus
   the Bindings in `CLAUDE.md` are what a plan's compliance check walks.
   `/sdd-constitution` is not run, because it would write a second set of
   principles beside the first. `memory/constitution.md` exists only as a
   pointer so the `sdd-*` skills find something at the path they read.
