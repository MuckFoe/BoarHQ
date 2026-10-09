---
domain: agent-conduct
reviewed: 2026-08-13
---

# Agent conduct rules

## CON1 — Destructive data operations are confirmed first
Deleting, overwriting, or irreversibly transforming persistent data is
confirmed before execution — bulk deletes, drops, truncates, resets and
re-seeds, data-discarding migrations, overwriting data files, history rewrites.
Applies in development and testing identically; "it is only local data" is not
a reason to skip confirmation. The confirmation states what is destroyed, how
much, and the recovery path — where there is none, that is said first.
Authorization is per operation and does not carry to the next one.
**Why:** destruction is the one class of action where speed has no upside and
error has no remedy, and the value of data is routinely misjudged until it is
gone.
→ *PreToolUse hook blocks on shape; the confirmation is a human decision*

## CON2 — Gaps in project context are recorded, not worked around
When the agent needs something the project's declared context does not contain
— a command, a convention, a constraint, a piece of domain knowledge — it
surfaces the gap rather than silently inferring past it. Once resolved, the
answer is added to the project's context files.
**Why:** this is why context files stagnate — each session hits the same gaps,
works around them, and ends, leaving the knowledge only in transcripts nobody
reads.
→ *advisory; nothing can detect a silent inference*

## CON3 — Change what the task requires, nothing else
The agent changes what the task requires and nothing more. Problems noticed in
passing — dead code, a bug outside the task, a stale comment, an obvious
refactor — are reported, not fixed. Where a fix genuinely is required to
complete the task, it is named as part of the work rather than folded in
silently.
**Why:** unrequested changes turn a diff that can be checked at a glance into
one that can only be skimmed, and destroy revert granularity along the way.
→ *review*
