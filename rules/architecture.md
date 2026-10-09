---
domain: architecture
reviewed: 2026-08-13
---

# Architecture rules

## ARC1 — Follow the established pattern, or agree a new one
New code follows the patterns already established in the area it lands in.
Introducing a different approach to a job the codebase already has an approach
for — state management, error handling, module structure, data access — is
proposed with its reason before it is written. Where the existing pattern is
genuinely wrong, it is changed across the area deliberately, not supplemented
with a second pattern beside it.

**Greenfield:** where no pattern exists yet, the agent does not choose
unilaterally. It presents the architecture it would suggest, the alternatives
it considered, and what each would cost, and discusses it with the user before
writing. Agreement precedes implementation; the agent does not proceed on
silence.
**Why:** two patterns for one job cost more than either alone — and the first
implementation becomes the precedent whether or not anyone decided it should.
→ *discussion + review; the greenfield clause is a hard stop*

## ARC2 — Dependencies flow one way
Dependencies flow in the direction the project declares. Lower layers do not
import from higher ones, and modules do not acquire circular dependencies.
Where a change appears to require reversing a declared dependency, that is
evidence the boundary is wrong or the code is in the wrong place — it is
raised, not worked around with an interface extracted solely to launder the
direction.
**Why:** once two modules point at each other they are one module with extra
steps, and neither can be tested, replaced, or reasoned about alone.
→ *review; import linters can enforce this where the ecosystem has one*
