# DRY, KISS, YAGNI

## DRY — Don't Repeat Yourself

Every piece of *knowledge* (a business rule, a constant, an algorithm, a schema) should have one authoritative representation.

**Violation signals**
- The same validation rule, formula, regex or magic value written in two or more places.
- Copy-pasted blocks that differ only in a variable name or literal.
- Parallel structures that must be edited together (an enum plus a matching `switch` plus a matching label map).

**Remedy**: extract a function, constant, type or table-driven definition; derive secondary representations from the primary one (generate, map, or compute).

**Not DRY violations** — leave them alone:
- Code that looks alike but represents different knowledge that will change for different reasons (coincidental duplication). Merging it couples unrelated features.
- Small repetition in tests where explicitness aids readability.

Rule of three: tolerate a second copy, extract on the third, unless the duplicated knowledge is clearly the same rule.

## KISS — Keep It Simple

Prefer the simplest construct that correctly solves the current problem.

**Violation signals**
- Abstraction layers, generics or patterns with a single concrete use and no boundary to protect.
- Clever one-liners, deep nesting, or metaprogramming where a plain loop or function reads better.
- Custom implementations of things the standard library or an existing dependency already provides.
- Configuration options nobody sets.

**Remedy**: inline the needless indirection, use standard library facilities, flatten control flow with guard clauses, name intermediate values.

## YAGNI — You Aren't Gonna Need It

Do not build for requirements that do not exist yet.

**Violation signals**
- Parameters, flags, hooks or extension points with no current caller.
- "Future-proof" plugin systems, generic frameworks or multi-backend support for a single backend.
- Unused methods, dead branches, commented-out code kept "just in case".

**Remedy**: delete it; keep the design easy to change instead (small units, tests, clear seams). Version control remembers removed code.

## Balancing them with patterns and SOLID

- KISS and YAGNI are the default; SOLID and patterns are applied in response to actual pressure (a second variant, a test that needs a seam, an I/O boundary).
- An abstraction should pay for itself now: by removing duplication, isolating a volatile dependency, or making code testable.
- When unsure, write the straightforward version first and refactor when the smell appears.
