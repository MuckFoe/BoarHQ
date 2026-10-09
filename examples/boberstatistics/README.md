# Example — BoberStatistics

The non-portable output of one real `/project-startup` run: a volleyball
scouting and statistics application, Go + React, personal project, deployed.
Run started 2026-08-13, closed 2026-08-14. Ten domains, 23 rules accepted, 2
rejected, 1 recorded as a finding, 7 substantially rewritten before acceptance.

**This is the run that produced the baseline ruleset.** The rule files are not
copied here — they are `rules/` at the repository root, which is exactly what
this run wrote. Duplicating them would recreate the two-copies problem that
[`rules/_progress.md`](rules/_progress.md) logged as an open follow-up.

What is here is everything that could *not* be portable:

| File | Why it is worth reading |
|---|---|
| `CLAUDE.md` | A **filled-in Bindings section** — the thing the starter ships as placeholders. Five bindings, each naming its rule id. |
| `rules/_decisions.md` | Two rejections with reasons, and seven rules shown as-proposed beside as-accepted. |
| `rules/_progress.md` | Real findings that failed the portability test, and honest open follow-ups. |
| `REQUIREMENTS.md` | Empty — the run closed before requirements were written. Left as-is rather than backfilled. |
| `docs/concepts.md` | Also empty, and the progress file says why that is the most expensive gap in the project. |

Paths inside these files (`rules/verification.md`, `frontend/src/...`) are
written relative to that project's root, not to this folder.

## What to look at first

**The Bindings section of `CLAUDE.md`.** It is the shortest demonstration of
why the portability split works. Five headings, each naming a rule id, each
carrying the one project-specific fact that rule needs to bite:

- VER5 gets the list of things a green check says nothing about — PDF output,
  the live scouting UI, keyboard shortcuts, chart rendering.
- SEC3 gets the classification of which endpoints are FULL and which are BASIC.
- ARC1 gets the load-bearing file, with its line count and why it is risky.
- SEC1 gets a known exposure that is recorded as **open, not accepted** —
  because SEC1 requires a stated reason and a remediation, and neither was
  given. That is the rule refusing to be satisfied by being written down, which
  is the behavior you want from a security rule.

None of those sentences would survive being moved to another project, and none
of the rules they serve mention them.

**The edits table in `_decisions.md`.** Seven rules where the obvious version
was tried first and found wanting — uniform validation that would have been
abandoned wholesale, agent-self-authorized suppression, greenfield architecture
as a notification rather than a stop. The baseline ships with the second
version of each. The table is why.

## The honest parts

The progress file's open follow-ups are not decoration. At the time this run
closed, the project had no version control, no test runner in either half, no
hooks built behind any rule claiming `hook` tier, an empty concepts file, and a
live unauthenticated deployment with two endpoints that delete data wholesale.

A run that ends with a clean ruleset and a blank follow-up list has usually not
looked hard enough.
