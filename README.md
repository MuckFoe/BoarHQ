# Project starter

A complete agent setup for a new repository: a working ruleset, a router, a
requirements protocol, a concepts glossary, and an interview skill that adapts
all of it to the project in front of you.

Drop it into a repository, put whatever documents you have into `input/`, and
run `/project-startup`.

## What this is

Not a template to fill in and not a skill that generates one from nothing —
**a setup that works on arrival and is then tuned.** Twenty-three rules across
nine domains ship populated, carried over from a real project and written to be
portable. The interview reviews them against your project one at a time, fills
in the parts that cannot be portable, and elicits what is missing.

The distinction matters because the two halves fail differently. A blank ruleset
means every project re-derives the same nine domains badly. A ruleset you were
handed and never reviewed is one you will not enforce. This does neither: the
rules arrive written, and keeping each one still takes your verdict.

## Layout

```
CLAUDE.md                    the router — commands, always-on citations, routing, bindings
REQUIREMENTS.md              every change traces to a requirement; the clarity bar
.gitignore

.claude/
  settings.json              permissions that ship with the kit
  skills/project-startup/
    SKILL.md                 the interview
    references/
      ingestion.md           reading input/ without inferring anything
      requirements.md        the clarity bar, and how to ask about what is unclear
      rule-catalog.md        candidate domains and the questions that surface real rules
      templates.md           exact file shapes for every output

rules/                       the baseline ruleset — portable policy, no paths or commands
  README.md                  how the ruleset works: the splits, the form, the tiers
  verification.md            VER1–5   how "done" is decided
  security.md                SEC1–4   the non-negotiables
  conduct.md                 CON1–3   the agent's control surface
  data.md                    DAT1–2   persistence and schema
  architecture.md            ARC1–2   boundaries and placement
  api.md                     API1–2   contract rules
  git.md                     GIT1     repository etiquette
  testing.md                 TST1–2   what must be covered and how
  style.md                   STY1–2   deviations from the defaults only
  _decisions.md              settled questions — read before proposing
  _progress.md               the index, findings, follow-ups, resume point

docs/
  concepts.md                accumulating domain glossary (VER2)

input/
  README.md                  what to drop in before running

examples/
  boberstatistics/           one real run's output, kept as a worked example
```

## Install

Copy everything above into the root of the target repository, minus `examples/`
if you don't want it. Then run:

```
/project-startup
```

The skill is manual-invocation only. It writes files and asks a lot of
questions, so it runs when you say so and not because something looked relevant.

## What the run does

1. **Reads `input/`** completely and writes `input/_analysis.md` — what the
   documents actually state, with citations, and a list of everything that could
   not be determined. The questions come first.
2. **Surveys the repository** — stack, structure, existing config, and what it
   could not work out from the code. Skipped if there is no code yet.
3. **Works through the business background with you** — resolves what the
   documents left open, then confirms the project profile: stage, stakes, where
   it runs. Ends when you agree it has understood the subject matter.
4. **Fills in the bindings.** Several shipped rules name a classification that
   lives in `CLAUDE.md` rather than in the rule — which areas no check can
   cover, which entry points get full validation, what is load-bearing, what is
   exposed. **Until this is done those rules are inert**, and this is the step
   most worth not skipping.
5. **Builds `REQUIREMENTS.md` with you**, one at a time. A requirement is
   written only when it is unambiguous; where it isn't, it tells you *exactly*
   what is unclear and what the competing readings are.
6. **Reviews every baseline rule against your project**, one at a time, with
   what it would bite on here and what it costs here. Keep / edit / reject /
   defer on each. Then proposes what the baseline does not cover.
7. **Writes** — the adapted rule files, the decisions log, the progress file.

Expect to reject some. A ruleset where you rejected a third is better than one
where you kept everything.

## The two constraints it will not bend on

**Every rule gets its own verdict — the shipped ones included.** Not in batches,
not by implication. A pre-written baseline you never reviewed is the same
problem as a generated ruleset you never reviewed, just faster to produce.

**Nothing about your project is inferred.** Not the domain rules, not the
business logic, not what a term means, not the edge cases. If the documents
don't say it and you haven't said it, it goes in the questions list — never into
a rule or a binding. An inferred business rule is indistinguishable from a real
one once written down, and from then on it gets implemented with confidence.

If you ever catch it stating something about your domain that you never told it,
that is a bug in the run. Stop and say so.

## Rules are portable, bindings are not

`rules/*.md` carries policy that would fire in any project — no paths, no
commands, no file or symbol names, no framework specifics. `CLAUDE.md` carries
everything tying it to this one. That split is what lets a ruleset be lifted
into the next repository whole, and it is the thing that erodes first: one rule
containing `go build ./...` makes the whole file project-specific.

A rule that only bites given your particular design is a **finding**, not a
rule. It goes in `rules/_progress.md`.

Full explanation in [rules/README.md](rules/README.md).

## Resuming

Long runs get paused. `rules/_progress.md` records the exact rule awaiting a
verdict, quoted in full. Clear context, run `/project-startup` again, and it
picks up there without re-surveying or re-proposing anything settled.

## The example

`examples/boberstatistics/` is the complete non-portable output of one real run
— a filled-in router with every binding populated, the decisions log with the
rules that were rejected and the seven that were substantially rewritten before
being accepted, and a progress file with real findings in it.

The rule files themselves are not duplicated there; they are `rules/`, which is
what that run produced. Worth reading before your own run, mainly for what a
finished Bindings section looks like.
