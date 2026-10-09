---
domain: data
reviewed: 2026-08-13
---

# Data rules

## DAT1 — Stored shapes are declared, not implied
Every persisted entity has one declared shape that is the authoritative
definition of what it stores — one definition per entity, not one per endpoint
that touches it. Fields the system reads or acts on are part of that
declaration. Data the system stores but never interprets **may be declared
opaque**: naming who owns its interior and stating that the system does not
read it is a declaration and satisfies this rule. What is not permitted is
reading into a structure that was never declared.
**Why:** unstructured storage is not the hazard — unstructured *interpretation*
is; a blob parsed by several call sites has as many meanings as it has readers.
→ *review; mechanism is language-specific and lives in CLAUDE.md*

## DAT2 — Derived data is never the source of truth
Data computable from other stored data is derived, and the derivation is the
authority. Where a derived value is stored for performance it is recomputed or
invalidated from its sources — never edited directly, never treated as correct
when it disagrees with them. Where a stored value is intended as a historical
snapshot rather than a cache, that intent is stated.
**Why:** a stored derived value drifts the first time a source changes through
a path that forgot to update it, and from then on the system holds two answers
with nothing to say which is stale.
→ *review*
