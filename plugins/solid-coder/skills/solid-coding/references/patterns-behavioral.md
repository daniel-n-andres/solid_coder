# Behavioral patterns

Each entry: intent · use when · avoid when · sketch. Sketches use TypeScript syntax; adapt to the target language.

## Chain of Responsibility
- **Intent**: pass a request along a chain of handlers until one handles it.
- **Use when**: several independent handlers may process a request and the set or order varies (middleware, validation pipelines, approval levels).
- **Avoid when**: there is a fixed, short sequence — call the steps directly.
```ts
type Handler = (req: Req, next: () => Res) => Res;
const pipeline = [auth, rateLimit, handle];
```

## Command
- **Intent**: turn a request into an object so it can be queued, logged, retried or undone.
- **Use when**: undo/redo, job queues, macro recording, transactional operations.
- **Avoid when**: a plain function call or callback is enough.
```ts
interface Command { execute(): void; undo(): void }
class RenameFile implements Command { /* execute renames, undo restores */ }
```

## Iterator
- **Intent**: traverse a collection without exposing its internal representation.
- **Use when**: custom data structures need sequential access.
- **Avoid when**: the language's built-in iteration protocol (generators, `for ... of`, `Sequence`, `Iterable`) already covers it — implement that protocol rather than a bespoke API.
```ts
class Tree { *[Symbol.iterator]() { /* yield nodes in order */ } }
```

## Mediator
- **Intent**: centralise communication between objects so they don't reference each other directly.
- **Use when**: many components interact in tangled ways (form fields, chat rooms, UI dialogs).
- **Avoid when**: it grows into a god object; split per use case.
```ts
class FormMediator { notify(sender: Field, event: string) { if (sender === country) postcode.setFormat(country.value); } }
```

## Memento
- **Intent**: capture and restore an object's state without exposing its internals.
- **Use when**: undo, snapshots, checkpoints.
- **Avoid when**: state is huge and changes often — consider Command-based undo or Event Sourcing.
```ts
const snap = editor.save(); /* ... */ editor.restore(snap);
```

## Observer
- **Intent**: notify dependents automatically when a subject changes.
- **Use when**: one change must trigger reactions in several decoupled places (UI bindings, domain events).
- **Avoid when**: there is exactly one fixed listener (call it directly), or ordering/error handling between listeners is critical. Always provide unsubscribe to prevent leaks.
```ts
orders.on("placed", e => mailer.confirm(e.order));
```

## State
- **Intent**: let an object change behaviour when its internal state changes, by delegating to state objects.
- **Use when**: many methods `switch` on a status field; explicit state machines (orders, connections, documents).
- **Avoid when**: there are two states and little behaviour difference.
```ts
interface OrderState { pay(o: Order): void; ship(o: Order): void }
class Paid implements OrderState { pay() { throw new AlreadyPaid(); } ship(o: Order) { o.state = new Shipped(); } }
```

## Strategy
- **Intent**: define interchangeable algorithms behind one interface, selected at runtime.
- **Use when**: `if`/`switch` selects among algorithms (pricing, sorting, compression, routing).
- **Avoid when**: there is one algorithm. In languages with first-class functions, a function parameter is often the simplest Strategy.
```ts
type Discount = (total: Money) => Money;
const checkout = (cart: Cart, discount: Discount) => discount(cart.total);
```

## Template Method
- **Intent**: define an algorithm's skeleton in a base, letting subclasses fill in specific steps.
- **Use when**: several algorithms share structure and differ in a few steps, and inheritance is already appropriate.
- **Avoid when**: composition works — prefer passing the varying steps as Strategies (composition over inheritance).
```ts
abstract class Importer { run() { const rows = this.parse(this.read()); this.save(rows); } protected abstract parse(raw: string): Row[] }
```

## Visitor
- **Intent**: add operations to a stable object structure without modifying its classes.
- **Use when**: the element types are stable but new operations are added often (AST processing, export formats).
- **Avoid when**: element types change often. With sum types and exhaustive pattern matching (Rust, Swift, Kotlin, TS discriminated unions), a `match`/`switch` over the union is usually simpler.
```ts
interface Visitor<R> { visitNumber(n: Num): R; visitAdd(a: Add): R }
```

## Null Object
- **Intent**: provide a do-nothing implementation of an interface instead of `null`, so callers need no null checks.
- **Use when**: absence has a sensible neutral behaviour (no-op logger, zero discount, guest user, empty collection).
- **Avoid when**: absence is an error or a meaningful case the caller must handle — then use `Optional`/`Result` or fail fast. Do not hide bugs behind silent no-ops.
```ts
interface Logger { log(m: string): void }
const NullLogger: Logger = { log() {} };

// Before: if (this.logger) this.logger.log("x");  (repeated everywhere)
// After:  constructor(private logger: Logger = NullLogger) {}  …  this.logger.log("x");
```
