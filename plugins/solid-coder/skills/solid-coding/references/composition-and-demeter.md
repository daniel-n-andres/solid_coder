# Composition over Inheritance, Law of Demeter

## Composition over inheritance

Build behaviour by combining small objects (has-a) rather than by extending base classes (is-a), unless the relationship is a true, stable subtype relationship that satisfies Liskov.

**Inheritance smells**
- Subclassing only to reuse a few helper methods.
- Hierarchies deeper than two levels, or "fragile base class" changes breaking subclasses.
- Subclass explosion from combining independent features (`LoggedCachedHttpClient`, `CachedHttpClient`, `LoggedHttpClient`, ...).
- `protected` state shared across the hierarchy and mutated from many levels.

**Remedy**: inject collaborators, use Strategy/Decorator/Bridge, delegate, or use mixins/traits/protocol extensions where the language encourages it.

```ts
// Before
class Logger { log(m: string) {} }
class OrderService extends Logger { place(o: Order) { this.log("placing"); /* ... */ } }

// After
class OrderService {
  constructor(private log: Logger) {}
  place(o: Order) { this.log.log("placing"); /* ... */ }
}
```

Inheritance is fine for: framework extension points that require it, sealed/algebraic type hierarchies, and genuine is-a relationships with substitutable behaviour.

## Law of Demeter (principle of least knowledge)

A method should only talk to its immediate collaborators: itself, its parameters, objects it creates, and its own fields. Do not reach through one object to manipulate another.

**Violation signals**
- Train-wreck chains: `order.getCustomer().getAddress().getCity().toUpperCase()` used to make a decision.
- Callers that pull data out of an object to compute something the object could compute itself ("ask, don't tell").
- Tests that need deep mock chains.

**Remedy**: "Tell, don't ask" — move the behaviour to the object that owns the data, or add an intention-revealing method.

```ts
// Before
if (order.getCustomer().getAddress().getCountry() === "DE") applyVat(order);

// After
if (order.shipsTo("DE")) applyVat(order);
```

**Not violations**: fluent builders and query DSLs returning the same object type, chaining over plain data structures/DTOs, and standard-library pipelines (`list.map(...).filter(...)`).
