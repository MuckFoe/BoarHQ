---
domain: api
reviewed: 2026-08-13
---

# API rules

## API1 — Breaking changes ship as a new version
Interfaces that callers depend on are versioned. Breaking changes are permitted
— removing, renaming, retyping, or redefining something callers rely on — but
they ship as a new version rather than being made in place, and the previous
version keeps working until its callers have moved off it. Additive changes are
not breaking and need no new version. Identifying which kind a change is comes
first: you cannot version a break you did not notice was one.
**Why:** versioning turns a breaking change from a coordination problem into a
migration; without it, "breaking" means every caller must update at the same
instant.
→ *review*

## API2 — Every capability exists through the API
Every operation the system performs on its data is available through its API.
Clients call the API rather than implementing operations it cannot perform, and
nothing reaches storage by a path that bypasses it. Local-first computation is
permitted where the API can perform the same operation — compute locally, sync
through the API; what is not permitted is a capability existing only inside a
client. Purely presentational behavior — layout, local UI state, formatting —
is not an operation and is out of scope.
**Why:** an operation living only in a client cannot be scripted, tested
independently, or reused, and it makes the API an untruthful description of
what the system can do.
→ *review*
