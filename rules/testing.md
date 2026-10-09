---
domain: testing
reviewed: 2026-08-14
---

# Testing rules

When tests run is VER1. Not deleting or skipping them to reach green is VER3.
These two cover what a test must be.

## TST1 — A test is seen failing before it is made to pass
Every new test is observed failing, for the expected reason, before the code
that makes it pass is written or changed. For a bug fix, that failing test
reproduces the bug, and the fix is not written until the reproduction fails.
**Why:** a test that has never failed proves nothing — it may assert on the
wrong object, pass vacuously, or not execute at all, and all three look
identical to a passing test.
→ *advisory; the evidence is the failure output being shown before the fix
(see VER1)*

## TST2 — Test what can break, not what exists
Tests target behavior that can plausibly break: logic, branching, boundaries,
invariants, integration points, anything with a rule encoded in it. Code
containing no decisions — trivial accessors, straight delegation, pure data
declarations, generated code — is not tested for its own sake; it is exercised
incidentally by the tests covering the behavior that uses it. Coverage
percentage is not a goal and is not evidence of anything.
**Why:** every test is code to maintain, and tests over trivia cost on every
run, break on every refactor, and train people to skim failures instead of
reading them. The question is "if this broke, would I want to know before
shipping" — never "is this line covered".
→ *review*
