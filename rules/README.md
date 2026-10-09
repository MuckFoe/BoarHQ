# The ruleset

Twenty-three rules across nine domains, carried over from a real project and
written to be portable. They are the **baseline**: they work as shipped, and
`/project-startup` adapts them to your project rather than starting from zero.

## The split that makes this reusable

**Rules are portable. Bindings are not.**

| Layer | Holds | Portable |
|---|---|---|
| `rules/*.md` | The policy. What must be true. | yes |
| `CLAUDE.md` | The bindings. Which commands, which paths, which classifications. | no |

Every file here carries policy only — no paths, no commands, no file or symbol
names, no framework specifics. That is what lets the whole directory be lifted
into the next repository unchanged. A single rule containing `go build ./...`
would make the whole file project-specific.

The test before adding a rule here: *would this fire in a different project
with a different stack?* If it only bites given this codebase's particular
design, it is a **finding** — it goes in `_progress.md`, not into a rule.

## Always-on versus triggered

Rules are classified by **when they apply**, not by topic. The test: *would
loading this rule late cause harm?*

- **Always-on** — governs behavior that happens before an agent would think to
  go read a rule file. A destructive-operation rule loaded after the command
  ran is worthless. These are cited one line each in `CLAUDE.md`.
- **Triggered** — loading it at the moment the area is touched is timely. These
  live only here and are reached through the routing table in `CLAUDE.md`.

`CLAUDE.md` **cites, never copies**. An always-on entry there is the rule id, a
one-line statement, and a path. Copying full rule text into the router creates
a second copy that will drift.

## The files

| File | Domain | Rules | Always-on |
|---|---|---|---|
| `verification.md` | how "done" is decided | VER1–5 | VER1, VER3, VER4 |
| `security.md` | the non-negotiables | SEC1–4 | SEC4 |
| `conduct.md` | the agent's control surface | CON1–3 | all three |
| `data.md` | persistence and schema | DAT1–2 | — |
| `architecture.md` | boundaries and placement | ARC1–2 | ARC1 |
| `api.md` | contract rules | API1–2 | — |
| `git.md` | repository etiquette | GIT1 | GIT1 |
| `testing.md` | what must be covered and how | TST1–2 | — |
| `style.md` | deviations from the defaults only | STY1–2 | — |
| `_decisions.md` | settled questions — read before proposing | — | — |
| `_progress.md` | the index, findings, follow-ups | — | — |

Nine of the twenty-three are always-on. Keep that set under about ten; past
that, the classification has gone soft.

## Written form

Statement, one-line **why**, enforcement tier. Nothing else.

**Keep the one-line why.** It is the highest-value-per-token content in the
file: it lets a model extend a rule to mechanisms the statement did not
enumerate, and — more importantly — it tells the model when the rule does *not*
apply. A statement alone is a blocklist; a statement with a reason is a
principle.

**Do not keep the long rationale.** The paragraph-length argument and the "if
wrong" analysis are decision support for you at accept time. Once a rule is
accepted they are cost with no reader.

Numbered ids exist so rules can be cited in review and in commit messages —
"violates VER3" is a complete review comment. Do not renumber on edit; retire
an id rather than reusing it.

## Enforcement tiers

Every rule declares one. This is the difference between a rule and a wish.

| Tier | Meaning |
|---|---|
| `advisory` | Stated in a rule file. Followed most of the time. Fine for preferences. |
| `rule-file` | Stated and cited by the router so it loads when relevant. |
| `hook` | A script enforces it deterministically. Cannot be skipped. |
| `CI` | Enforced outside the agent entirely. The only tier that always holds. |

A `hook`-tier rule with no hook behind it is mislabeled, and mislabeling is
worse than being advisory honestly. `_progress.md` tracks which enforcement is
still unbuilt.

## Changing a rule

Propose the change as its own rule with the old text shown alongside, and
record the outcome in `_decisions.md`. Never silently rewrite an approved rule —
the decisions log is what stops a future run re-litigating a settled question.
