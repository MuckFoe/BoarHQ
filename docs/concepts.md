# Concepts

Domain vocabulary and the invariants that hold in this system. **One
accumulating file — never one per feature** (VER2). This is also where the
canonical term for each concept is fixed, which is what makes STY1 checkable.

An entry is: the term (in the domain's own language and in English where they
differ), what it means *in this system*, and any invariant that must hold.

Add a concept when a feature introduces or changes one. A feature that
introduced no new concept adds nothing here, and that is a valid outcome.

## Format

```markdown
## <Term> (<other-language term, if the domain has one>)
<What it means in this system. Not the general-industry meaning — the one that
holds here.>

**Invariants:** <what must always be true of it, or "none stated">
**Source:** <citation from input/, or "user, <date>">
```

Where the domain's vocabulary is not in the language of the code, **keep the
domain term** rather than translating it (STY1). A translated term is a second
name for the same thing, and a concept with two names cannot be searched for
reliably.

A term used in the codebase but never defined is **undefined**, not
self-evident. Record it here as undefined rather than supplying a plausible
meaning — an invented definition is indistinguishable from a real one once it
is written down.

---

*Empty. The domain rules already encoded in the codebase's most load-bearing
file are the highest-value entries this file could have — start there.*
