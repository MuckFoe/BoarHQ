---
domain: security
reviewed: 2026-08-13
---

# Security rules

## SEC1 — No unauthenticated CRUD beyond localhost
Endpoints that read, create, modify, or delete data require authentication
before the service is reachable from anywhere other than the developer's own
machine. Endpoints exposing no data — liveness and health probes — are exempt.
Making an unauthenticated service reachable beyond localhost requires explicit
sign-off and is recorded as a known exposure, with the reason it was accepted
and what would remove it. The trigger is exposure, not existence.
**Why:** read exposure leaks data and mutation exposure destroys it; covering
both under one rule removes the per-endpoint severity judgment where this kind
of rule usually erodes.
→ *sign-off; not mechanically enforceable*

## SEC2 — Internal error detail stays server-side
Responses carry a stable, non-diagnostic message and a status code. Driver and
database errors, stack traces, query fragments, paths and version strings are
logged server-side, never placed in a response body. Where a caller must be
able to report a specific failure, the response carries a correlation
identifier that appears in the log beside the full detail.
**Why:** internal errors describe internals to anyone who can provoke one, and
passing them through turns error text into an API nobody meant to publish.
→ *review; project lint rule optional*

## SEC3 — Validation depth follows source and criticality
**FULL** — explicit shape; every field stored or acted on is declared and
checked; fields outside the shape are rejected or dropped by explicit decision.
Required for input originating from a person, and for any destructive or
irreversible operation regardless of caller.
**BASIC** — required fields present, types correct, identifiers well-formed,
sizes bounded. Sufficient for routine, reversible programmatic calls.
At every level: fields used for identity, ownership, or access control are
never taken from a request body unchecked.
**Why:** uniform strict validation on a moving model gets abandoned wholesale,
taking the critical paths down with it — and criticality overrides source
trust, since being trusted is not the same as being correct.
→ *review; boundary classification lives in CLAUDE.md*

## SEC4 — Secrets never enter the repository or the transcript
Credentials, tokens, keys and credential-bearing connection strings are loaded
at runtime from the environment or a secret store. Never in source, committed
config, logs, or error messages. Never echoed into agent output, summaries,
commit messages, or generated documents. Where a task genuinely requires a
secret's value, the agent states what it needs rather than retrieving and
displaying it.
**Why:** a committed secret is disclosed from the moment it lands and stays in
history until history is rewritten; agent transcripts are a second egress path
that gets overlooked.
→ *pre-commit scan + OS-level sandbox read-denial (permission deny rules do not
cover shell subprocesses)*
