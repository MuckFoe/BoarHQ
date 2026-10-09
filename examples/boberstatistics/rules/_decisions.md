# Ruleset decisions

Settled questions. Read before proposing new rules — do not re-litigate these.

## Rejected

| Proposed rule | Reason | Date |
|---|---|---|
| `cmd/1` — use the declared command rather than an inferred invocation | Redundant with the CLAUDE.md command table itself. A rule telling the agent to read a table it already loads is noise. The residual value ("ask when a command is missing") generalised into CON2 instead. | 2026-08-13 |
| `git/1` (original) — one logical change per commit | Commit *timing* does not need a rule. VER1 already defines when a feature is done, and commit timing follows from it: feature done → commit → PR if needed. Git gets exactly one rule, and it is about message quality. | 2026-08-14 |

Deliberately not proposed in **git**, and available to reopen: "don't commit or
push unless asked" (already default agent behavior), "don't rewrite published
history" (covered by CON1), "never commit secrets" (covered by SEC4).

Deliberately skipped domains: performance, accessibility, observability,
release, docs. For a project at this stage they produce generic rules, and
generic rules dilute the real ones.

## Deferred

None.

## Edited during elicitation

The user rewrote these substantially. Originals kept so the intent behind each
edit stays visible.

| ID | As proposed | As accepted |
|---|---|---|
| VER1 | One gate: build + type check on every change | Two gates — fast check per change set, full check including tests per completed feature. Test clause is aspirational until runners exist. |
| VER3 | Suppression allowed with a same-line comment stating why | Suppression requires the user's sign-off; the agent does not self-authorize and annotate |
| SEC1 | Auth required on endpoints that *mutate* data | Auth required on all CRUD; health probes exempt |
| SEC3 | Uniform explicit-shape validation at every trust boundary | Two graded levels — FULL for human-originated input and destructive operations, BASIC for routine programmatic calls |
| DAT1 | Every persisted entity has a declared shape | Opaque storage is itself a valid declaration; the violation is *reading into* an undeclared structure |
| ARC1 | Greenfield patterns are named and proposed | Greenfield architecture is *discussed and agreed* with the user before writing — a hard stop, not a notification |
| API1 | Breaking changes are identified and handled deliberately | Breaking changes are fine — they ship as a new version |

## Structural decisions

1. **Rules are portable.** No paths, commands, or file names in `rules/*.md`.
   A rule that only bites given this codebase's design is a finding, not a
   rule. Project specifics live in `CLAUDE.md` as bindings.
2. **Compact written form.** Statement + one-line why + enforcement tier. The
   one-line why earns its tokens — it lets the model generalise a rule beyond
   the mechanisms listed, and tells it when the rule does *not* apply. Long
   rationale is decision support at accept time and stays in the transcript.
3. **Split by when a rule applies, not by topic.** Always-on rules are cited
   one line each in `CLAUDE.md`; triggered rules are routed on demand. Test:
   *would loading this late cause harm?*
4. **A fifth verdict, `record`.** Real problems that are not portable rules go
   to the run's findings list rather than into the ruleset.
