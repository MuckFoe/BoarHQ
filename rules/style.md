---
domain: code-style
reviewed: 2026-08-14
---

# Code style rules

Formatting belongs to the formatter, lint to the linter, and consistency with
surrounding code to ARC1. These two cover what neither tool can.

## STY1 — One term per concept
A concept has one name, used consistently across code, data, and interfaces.
Where the domain's vocabulary is not in the language of the code, the domain
term is kept rather than translated — a translated term is a second name for
the same thing. Renaming a concept is legitimate, done everywhere at once; two
names coexisting is not.
**Why:** naming is the only documentation that cannot go stale, because it is
the code — and a concept with two names cannot be searched for reliably.
→ *review; the canonical term for each concept is recorded in the concepts
document (VER2)*

## STY2 — Comments carry what the code cannot
Comments carry the three things code cannot express about itself:

- **What** — the bigger picture, kept minimal. A line or two that orients a
  reader: what this block amounts to in domain terms. Not a walkthrough of its
  steps.
- **How** — when the mechanism is not self-evident: the approach taken, and why
  that one.
- **Why** — the reason for the choice, what was rejected, what constraint
  forced it.

Excluded: restating the line below. A comment echoing its code is the one that
goes stale silently, because nothing about changing the code draws attention
to it.

**Why:** code states its mechanism exactly and its meaning not at all — and one
line of orientation above a dense block is often the most valuable line in the
file. Keeping it to one line is what separates it from the step-by-step
narration that "why, not what" was reacting against.
→ *review*
