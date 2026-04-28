# Architecture Glossary

Use these terms exactly when proposing architecture changes during `/wf:plan`. Consistent language is the point — don't drift into "component," "service," "API," or "boundary."

## Terms

- **Module** — anything with an interface and an implementation (function, class, package, slice).
- **Interface** — everything a caller must know to use the module: types, invariants, error modes, ordering, config. Not just the type signature.
- **Implementation** — the code inside.
- **Depth** — leverage at the interface: a lot of behaviour behind a small interface. **Deep** = high leverage. **Shallow** = interface nearly as complex as the implementation.
- **Seam** — where an interface lives; a place behaviour can be altered without editing in place. Use this, not "boundary."
- **Adapter** — a concrete thing satisfying an interface at a seam.
- **Leverage** — what callers get from depth.
- **Locality** — what maintainers get from depth: change, bugs, knowledge concentrated in one place.

## Key Principles

### Deletion test

Imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.

### The interface is the test surface

If you cannot drive a module's behaviour through its interface in tests, the interface is wrong. Do not extract pure functions just to make them testable — fix the interface instead.

### One adapter = hypothetical seam. Two adapters = real seam.

A single implementation behind an interface is just deferred coupling. The interface earns its keep when at least two adapters need it. Beware "future-proof" interfaces with one adapter.

## When These Apply in /wf:plan

- **Surfacing friction** during the codebase survey — apply the deletion test to suspicious modules.
- **Comparing architecture approaches** — describe trade-offs in terms of leverage and locality, not vague "cleanness."
- **ADR rationale** — when capturing a decision, name the depth / locality / leverage trade-off explicitly.

## Vocabulary Source of Truth

This skill is *informed by* the project's domain model. The domain language gives names to good seams; existing ADRs record decisions the skill should not re-litigate.

Use `CONTEXT.md` (or equivalent project glossary) vocabulary for the domain, and the terms above for architecture. If `CONTEXT.md` defines "Order," talk about "the Order intake module" — not "the FooBarHandler," and not "the Order service."
