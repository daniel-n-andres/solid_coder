# Clean Code basics

## Naming
- Names reveal intent: `elapsedDays` not `d`, `isEligibleForRefund` not `check`.
- Functions are verbs (`calculateTotal`), types and values are nouns (`Invoice`), booleans read as questions (`hasAccess`, `isEmpty`).
- One word per concept across the codebase (don't mix `fetch`, `get`, `retrieve` for the same thing).
- Avoid encodings, abbreviations and noise words (`data`, `info`, `obj`, `tmp`, `Manager`) unless they are domain terms.
- Name length grows with scope: short names for tiny scopes, descriptive names for wide ones.

## Functions
- Small and doing one thing at one level of abstraction; extract when a block needs a comment to explain what it does.
- Few parameters (aim for ≤ 3); group related ones into a parameter object; avoid boolean flag parameters that switch behaviour — split into two functions.
- No hidden side effects; separate commands (change state) from queries (return data).
- Return early with guard clauses instead of deep nesting (aim for ≤ 2–3 levels).

```ts
// Before
function pay(user, amount) {
  if (user) {
    if (user.active) {
      if (amount > 0) { /* ... */ }
    }
  }
}

// After
function pay(user: User, amount: Money) {
  if (!user.active) return;
  if (amount.isZeroOrLess()) throw new InvalidAmount(amount);
  /* ... */
}
```

## Values and types
- Replace magic numbers and strings with named constants or enums.
- Use domain types / value objects (`Money`, `EmailAddress`) instead of primitive obsession where rules attach to the value.
- Prefer immutability; limit the scope and lifetime of mutable state.

## Error handling
- Fail fast on invalid input at boundaries; keep the core free of defensive clutter.
- Use the language's error mechanism consistently (exceptions, `Result`, error returns); do not return magic error codes or swallow errors silently.
- Don't return `null` for "no result" when an empty collection, an `Optional`, or a Null Object expresses it better.
- Error messages state what failed and with which input.

## Comments and structure
- Code explains *what* and *how*; comments explain *why* (decisions, constraints, links to specs).
- Delete commented-out code and stale comments.
- Keep related code close, order functions top-down (caller above callee), and keep files focused on one concept.
- Format consistently with the project's formatter and conventions.

## Tests
- Readable, fast, independent, and deterministic; one behaviour per test with a descriptive name.
- Testability is a design signal: code that is hard to test usually violates SRP or DIP.
