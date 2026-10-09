---
domain: git
reviewed: 2026-08-14
---

# Git rules

## GIT1 — Commits state intent and name their requirement
Every change is driven by a requirement, and the commit names it.

- **Subject** — imperative, stands alone. Readable in a log without opening the
  diff, and it names the change, not the files touched.
- **Reference** — the requirement the change serves.
- **Body** — the why, whenever the subject does not carry it.

Where a change has no requirement, one is written first. A single sentence is
enough; the bar is that it exists and came first, not that it is formal. A
message that restates the diff — "update handlers", "fix bug", "changes" —
does not satisfy this rule.

**Why:** the diff already says what changed, so the only thing a message can add
is why it was done at all — and requiring a requirement makes that answerable
by construction.
→ *review; a commit-msg hook can enforce shape and the presence of a reference,
not whether the requirement is real*
