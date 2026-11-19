# SOLID

Sketches use TypeScript syntax for readability; translate to the target language's idioms (protocols, traits, ABCs, duck typing, plain functions).

## S — Single Responsibility Principle

A unit (class, module, function) should have one reason to change, i.e. answer to one actor or concern.

**Violation signals**
- Name contains "And", "Manager", "Util", "Helper", "Processor" with mixed duties.
- One class does validation + persistence + formatting + notification.
- Changes for unrelated features keep touching the same file.
- Methods cluster into groups that use disjoint fields.

**Remedy**: split along the axes of change; keep a thin coordinator if needed.

```ts
// Before: one class, three reasons to change
class InvoiceService {
  total(inv: Invoice) { /* pricing rules */ }
  save(inv: Invoice) { /* SQL */ }
  emailPdf(inv: Invoice) { /* rendering + SMTP */ }
}

// After
class InvoicePricing { total(inv: Invoice): Money { /* ... */ } }
interface InvoiceRepository { save(inv: Invoice): void }
class InvoiceMailer { constructor(private renderer: PdfRenderer, private mail: MailGateway) {} send(inv: Invoice) { /* ... */ } }
```

## O — Open/Closed Principle

Open for extension, closed for modification: adding a new variant should mean adding code, not editing stable code.

**Violation signals**
- `switch (type)` / `if (kind === ...)` chains that grow with every new variant, often duplicated in several places.
- A "registry" that must be hand-edited in many places for each new case.

**Remedy**: polymorphism, Strategy, a lookup map of handlers, plugin/registration points.

```ts
// Before
function shippingCost(order: Order) {
  if (order.method === "standard") return 5;
  if (order.method === "express") return 15;
  if (order.method === "pickup") return 0;
}

// After
interface ShippingMethod { cost(order: Order): Money }
const methods: Record<string, ShippingMethod> = { standard: new Standard(), express: new Express(), pickup: new Pickup() };
const shippingCost = (order: Order) => methods[order.method].cost(order);
```

Don't pre-emptively abstract: one `if` with two branches that never grows is fine (KISS/YAGNI). Apply OCP at the second or third variant, or where variants are known to come.

## L — Liskov Substitution Principle

Subtypes must be usable wherever the base type is expected without callers noticing: same or weaker preconditions, same or stronger postconditions, invariants kept, no new exceptions for base-type behaviour.

**Violation signals**
- Overrides that throw `NotImplemented`/`UnsupportedOperation`.
- Callers checking `instanceof` / type before calling.
- Subclass that ignores or silently changes base behaviour (classic Square extends Rectangle with independent `setWidth`).

**Remedy**: redesign the hierarchy around actual shared behaviour, split interfaces, or use composition.

```ts
// Before: not every bird can fly
class Bird { fly() {} }
class Penguin extends Bird { fly() { throw new Error("can't fly"); } }

// After
interface Bird { move(): void }
interface FlyingBird extends Bird { fly(): void }
```

## I — Interface Segregation Principle

Clients should not be forced to depend on methods they do not use. Prefer several small, role-based interfaces over one wide one.

**Violation signals**
- Implementations with empty or throwing methods to satisfy an interface.
- An interface with many methods where each caller uses a few.
- Test doubles that must stub large unused surfaces.

```ts
// Before
interface Machine { print(d: Doc): void; scan(d: Doc): void; fax(d: Doc): void }

// After
interface Printer { print(d: Doc): void }
interface Scanner { scan(d: Doc): void }
class SimplePrinter implements Printer { print(d: Doc) { /* ... */ } }
```

## D — Dependency Inversion Principle

High-level policy should not depend on low-level details; both depend on abstractions owned by the policy side. Wire concrete implementations at the edge (composition root).

**Violation signals**
- Business logic that calls `new DatabaseClient()`, reads env vars, touches the file system, network, clock or random directly.
- Domain modules importing framework or vendor SDK types.
- Code that can only be tested with real infrastructure.

```ts
// Before
class SignupService {
  register(u: User) { new PostgresUsers().insert(u); new SmtpMailer().send(u.email, "Welcome"); }
}

// After
interface Users { insert(u: User): void }
interface Mailer { send(to: Email, msg: string): void }
class SignupService {
  constructor(private users: Users, private mailer: Mailer) {}
  register(u: User) { this.users.insert(u); this.mailer.send(u.email, "Welcome"); }
}
// composition root: new SignupService(new PostgresUsers(db), new SmtpMailer(cfg))
```

Single-implementation interfaces are justified at I/O and framework boundaries; inside pure domain code, depend on concrete value types freely.
