---
name: project-startup
description: Interactively adapt this project's agent ruleset — a lean CLAUDE.md router plus portable domain rule files — to what this project actually is. Reviews the shipped baseline ruleset rule by rule and elicits what is missing, getting an explicit verdict on every rule before anything is written. Can ingest existing knowledge files (style guides, ADRs, READMEs, notes) and convert them into candidate rules.
disable-model-invocation: true
---

# Project Startup

Adapt the ruleset that governs agent work in this repository. Nothing is written
to disk until the user has approved it, rule by rule.

## You are not starting from a blank page

This repository ships with a **baseline**: twenty-three rules across nine
domains in `rules/`, a router at `CLAUDE.md` with its Bindings section left as
placeholders, a `REQUIREMENTS.md`, and an empty `docs/concepts.md`. Read
`rules/README.md` and `rules/_decisions.md` before proposing anything.

Your job is three things, in this order:

1. **Fill in the bindings.** Several baseline rules name a classification that
   lives in `CLAUDE.md`, not in the rule — VER5's uncoverable areas, SEC3's
   validation levels, DAT1's declared shapes, ARC1's load-bearing files, SEC1's
   known exposures. Until those are filled in, those rules are inert. This is
   the highest-value work in the run and it is the part most often skipped.
2. **Review each baseline rule against this project.** One at a time, same five
   verdicts. A baseline rule is a *proposal*, not a fait accompli — keeping one
   requires a verdict exactly as much as adding one does.
3. **Elicit what the baseline does not cover.** The project's own rules, from
   `input/`, from the code, and from the user.

Do not rebuild the general-purpose ruleset from scratch. It exists, the reasons
behind its current shape are in `_decisions.md`, and re-deriving it wastes the
run.

## Two hard constraints

### 1. Every rule is approved individually

**No rule reaches a file without an explicit verdict from the user on that
specific rule.** Not a batch. Not an implied yes. Not "sounds good, continue"
covering five proposals. One rule, one verdict. **This applies to baseline rules
too** — an unreviewed baseline is exactly the ruleset-you-did-not-author problem
this skill exists to prevent, just pre-packaged.

This is not a formality — it is the entire point of the skill. A ruleset the
user did not author is a ruleset they will not trust, enforce, or maintain. If
the user says "just keep them all," stop and say that defeats the purpose, then
offer to speed up by narrowing domains instead.

### 2. Nothing about the project is inferred

**Never state as fact anything the inputs do not state.** Not the domain rules,
not the business logic, not the workflows, not what a term means, not what the
edge cases are. No threshold of obviousness suspends this.

An inferred fact and a stated fact are indistinguishable once written into a
rule file. From then on the inference is trusted, implemented, and defended —
and nothing marks it as invented. A wrong business rule is worse than a missing
one, because a missing one gets asked about.

Where something is not stated: ask, or record it as unknown. **"I could not
determine this" is always an acceptable answer and frequently the correct one.**
Full protocol in `references/ingestion.md`.

## Ground rules to state before starting

Open the session by telling the user, in your own words:

- The repository already has a baseline ruleset; the run adapts it rather than
  inventing one, and every baseline rule still needs their verdict to stay.
- Rules are handled one at a time, each with evidence and a stated cost.
- Five verdicts: **accept** / **edit** (they rewrite it) / **reject** / **defer** /
  **record** (it is a real problem, but not a portable rule — see below).
- Rejections are recorded with reasons so future runs don't re-propose them.
- Nothing is written until a domain is fully resolved.
- They can stop at any point and keep what's been approved.

## Process

### Phase 1 — Input analysis

Read everything in `input/` first, completely, before looking at code and
before proposing anything. Follow `references/ingestion.md` exactly — the
classification table and the citation requirement are what make constraint 2
checkable rather than a promise.

Produce `input/_analysis.md` and walk the user through it, **questions first**.
Those questions are the decisions only they can make; every one closed by
inference instead is a defect that cannot be found later.

If `input/` is empty or absent, say so and continue. An empty input folder
raises the importance of the questions, it does not lower it.

### Phase 2 — Repository survey

Then read the repository. Skip if there is no code yet. Gather:

- Language, framework, build system, test runner, package manifests
- Directory structure and where the real boundaries are
- Existing config: linters, formatters, CI, docker, editorconfig

Produce a **findings summary**, not rules. Include what you could *not*
determine — those gaps are where rules are most needed. Keep facts observed in
the code separate from claims made in the input documents, and say plainly
where the two disagree.

Answer specifically what the baseline's open follow-ups ask: whether a test
runner exists, whether the repository is under version control, and which areas
have no automated check behind them. Those answers feed Phase 3b.

### Phase 3 — Business background

Walk the user through the analysis, **Questions section first**, and resolve
what can be resolved. Anything still open at the end of this phase stays open —
it does not get closed by inference (constraint 2).

Then confirm the project profile: what it is, who runs it, what stage it is at
(prototype / in production / handed off), where it runs, and what breaking it
would cost. The answers change which rules matter. A weekend prototype and a
system holding customer data deserve different rulesets, and applying the same
baseline unchanged to both is the most common way this goes wrong.

This phase ends when the user agrees you have understood the subject matter.
Do not proceed on the basis that you have read the documents — reading them is
not the same as having understood what they leave out.

### Phase 3b — Bindings

Fill in `CLAUDE.md`: identity paragraph, stack, command table, and the
**Bindings** section. Take the bindings one at a time, the same as rules — each
is a claim about this project that you must not invent.

- **Commands** — exact invocations including working directory. Ask; do not
  reconstruct from a package manifest and hope.
- **Cannot be automatically verified** (VER5) — the areas where a green check
  proves nothing. If the answer is "nothing", say so explicitly; a blank
  heading reads as unfinished, not as empty.
- **Validation levels** (SEC3) — which entry points are FULL, which are BASIC.
- **Declared shapes** (DAT1) — where each persisted entity's authoritative
  definition lives, and the mechanism.
- **Load-bearing** (ARC1) — what should not be refactored casually, and why.
- **Known exposures** (SEC1) — SEC1 requires a stated reason *and* a
  remediation before an exposure counts as accepted. An exposure with neither
  is an open finding; record it as one and say so plainly rather than letting
  it read as accepted.

A rule whose binding is missing is inert. Say which rules you have just
activated when this phase closes.

### Phase 4 — Requirements

Build `REQUIREMENTS.md` with the user, following `references/requirements.md`.

The bar is that **a requirement is written only when it is unambiguous**, and
where it is not, you say *precisely* what is unclear — naming the requirement,
quoting the ambiguous part, and stating the competing readings you can see.
"Let me know if anything is unclear" is not asking; it reliably produces
agreement and leaves the ambiguity in place.

One requirement at a time. A requirement containing an assumption is
indistinguishable from one the user specified the moment it is written down.

### Phase 5 — Domain selection

Two lists, presented together:

**The nine baseline domains.** For each, state in one line what it governs and
what you found in Phases 1–2 that makes it apply — or fail to apply — *here*. A
baseline domain with no evidence behind it in this project is a candidate for
rejection, and saying so is the useful contribution.

**Candidate additional domains** from `references/rule-catalog.md`. Check
`_decisions.md` first: performance, accessibility, observability, release and
docs were left out of the baseline deliberately, with reasons. Propose one only
where you found evidence it bites here, and say what changed relative to that
reasoning.

The user picks which domains to work through and in what order. Fewer, sharper
domains beat complete coverage.

### Phase 6 — Rule elicitation, one at a time

For each selected domain, handle the baseline rules first, then new proposals.

**For a baseline rule**, present it in this format:

```
BASELINE  <ID>
-----------------------------------------------
Statement     <the rule as it currently stands, quoted>
Applies here  <what in this project it would bite on — file:line where you
              have it, or "no evidence found in this codebase">
Binding       <the binding it needs, and whether Phase 3b supplied one>
Cost here     <what following it costs on this project specifically>

keep / edit / reject / defer ?
```

**For a new rule**, propose individually in this format:

```
RULE  <domain>/<n>
-----------------------------------------------
Statement   <the rule, imperative, one or two lines>
Why         <what goes wrong without it>
Evidence    <file:line, knowledge-file citation, or "no evidence — convention">
Tier        <advisory | rule-file | hook | CI> — see Enforcement tiers below
If wrong    <cost of this rule being a mistake — be honest>

accept / edit / reject / defer ?
```

Then stop and wait. Do not queue the next rule in the same message.

**Quality bar.** Before proposing, check the rule is:

1. **Actionable** — names a behavior, not a sentiment. "Write clean code" fails.
2. **Falsifiable** — you could point at a diff and say whether it was violated.
3. **Non-inferable** — an agent reading the code wouldn't already do this. If it
   would, the rule is noise and dilutes the rules that matter.
4. **Tiered** — you can name where it's enforced.
5. **Portable** — it would fire in a different project with a different stack.
   A rule that only bites given this codebase's particular design is a finding,
   not a rule. Ask: *would I want this in the next project?* If no, propose it
   as a **record**, not a rule.
6. **Not already covered** — check the baseline before proposing. A new rule
   overlapping an existing one is the fastest route to a self-contradicting
   ruleset. If it sharpens an existing rule, propose it as an **edit** to that
   rule with the old text alongside, not as an addition.

**The `record` disposition.** Elicitation surfaces real problems that are not
policy — a specific broken behavior, a stale artifact, a design decision worth
revisiting. These are worth capturing and worth *not* putting in the ruleset,
because a portable ruleset diluted with situational entries stops being
portable. Write them to `rules/_progress.md` under project findings, and say
plainly that you are recording rather than proposing.

If a candidate fails any of these, don't propose it. Say what you dropped and
why when you finish the domain — the user may disagree and want it back.

**Enforcement tiers.** Every rule declares one. This is the difference between a
rule and a wish:

| Tier | Meaning |
|---|---|
| `advisory` | Stated in a rule file. Followed most of the time. Fine for preferences. |
| `rule-file` | Stated and cited by the router so it's loaded when relevant. |
| `hook` | A script enforces it deterministically. Cannot be skipped. |
| `CI` | Enforced outside the agent entirely. The only tier that always holds. |

When a user accepts a rule at `hook` or `CI` tier, note it — after the ruleset is
written, offer to implement the enforcement. A `hook`-tier rule with no hook
behind it is mislabeled, and mislabeling is worse than being advisory honestly.
The baseline ships with none of its hooks built; `_progress.md` lists them.

### Phase 7 — Write

Only after a domain's rules are all resolved:

1. Show the exact file content you intend to write, as a diff against what is
   already there. `rules/` is populated — never present a rewrite of an
   existing file as if it were a new one.
2. Get one final confirmation.
3. Write it, using the templates in `references/templates.md`.

`CLAUDE.md` is a **router**, not a container. It holds only what applies to every
session — identity, commands, and a table pointing at domain rule files with the
trigger for reading each. Domain knowledge lives in `rules/<domain>.md` and loads
on demand.

Do not use `@path` import syntax for domain rules. Imports load eagerly at
session start, which reintroduces exactly the context bloat the split avoids.
Reference the path in prose so it is read only when its trigger fires.

Keep `CLAUDE.md` under 200 lines. If it grows past that, something belongs in a
rule file.

**Rules stay portable.** No paths, commands, or file names in `rules/*.md` — if
a rule needs one, either the rule is wrong or the specific belongs in Bindings.
This is what lets the ruleset be lifted into the next repository whole, and it
is the constraint that erodes first.

### Phase 8 — Record decisions

Update `rules/_decisions.md`: every rejected and deferred rule with the user's
reason, and every baseline rule the user rewrote, with the original alongside.
Preserve the entries marked *(baseline)* — they are why the shipped rules have
the shape they do, and a future run that loses them will re-propose the forms
that were already tried and rejected.

Update `rules/_progress.md`: the status line, the domain table, the ruleset
index, findings, and which follow-ups are now closed.

## Resuming a paused run

Long runs get paused. Before anything else, read `rules/_progress.md` — it is
the authoritative resume point and its **Resume here** section names the exact
rule awaiting a verdict. Then `_decisions.md`, then the rule files. Do not
re-survey the repository from scratch and do not re-propose anything settled.

Keep `_progress.md` current as each domain closes: it is what makes the run
survivable across a context reset, and a run that cannot survive one will not
finish.

## Re-running on an adapted ruleset

If `rules/_progress.md`'s status line no longer says the baseline is unadapted,
a run has already happened here. Read it and `_decisions.md` first, then work
only on gaps and on rules the user flags as stale. Never silently rewrite an
approved rule — propose the change as its own rule with the old text alongside.

## References

- `references/ingestion.md` — reading `input/` without inferring anything (Phase 1)
- `references/requirements.md` — the clarity bar and how to ask about what is unclear (Phase 4)
- `references/rule-catalog.md` — candidate domains and the questions that surface real rules (Phase 5)
- `references/templates.md` — exact file shapes for CLAUDE.md, rule files, and the logs (Phase 7)
- `rules/README.md` (repository root) — how the shipped ruleset is structured: the
  portability split, the always-on axis, the written form, the enforcement tiers
