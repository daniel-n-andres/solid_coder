# Architectural and application patterns

Each entry: intent · use when · avoid when · sketch. Sketches use TypeScript syntax; adapt to the target language and framework conventions.

## Dependency Injection
- **Intent**: supply an object's dependencies from outside instead of having it create or look them up.
- **Use when**: always at I/O and framework boundaries; it is the practical implementation of DIP and the main enabler of testing.
- **Avoid when**: injecting pure helpers or value types adds nothing. Prefer constructor injection; use a DI container only when the framework expects one or wiring is genuinely large. Avoid service locators (hidden dependencies).
```ts
class ReportJob { constructor(private repo: SalesRepo, private clock: Clock) {} }
// composition root: new ReportJob(new SqlSalesRepo(db), systemClock)
```

## Repository
- **Intent**: present persistence as a collection-like interface of domain objects, hiding the data source.
- **Use when**: business logic needs to load/save aggregates and should not know SQL/ORM/HTTP details.
- **Avoid when**: the app is a thin CRUD layer over the ORM and the repository would only forward calls (KISS).
```ts
interface OrderRepository { byId(id: OrderId): Promise<Order | undefined>; save(o: Order): Promise<void> }
```

## Service Layer
- **Intent**: define the application's use cases as a layer of operations that coordinates domain objects, transactions and infrastructure.
- **Use when**: several entry points (HTTP, CLI, jobs) need the same use cases; controllers are accumulating business logic.
- **Avoid when**: it becomes an anemic pass-through or a dumping ground; keep business rules in the domain objects.
```ts
class PlaceOrder { constructor(private orders: OrderRepository, private payments: PaymentGateway) {} async run(cmd: PlaceOrderCmd) { /* ... */ } }
```

## MVC (Model–View–Controller)
- **Intent**: separate data/business logic (Model), presentation (View) and input handling (Controller).
- **Use when**: server-rendered web apps and UI frameworks built around it.
- **Avoid when**: "fat controllers" — move business logic into the model or a service layer.

## MVVM (Model–View–ViewModel)
- **Intent**: a ViewModel exposes view state and commands; the View binds to it declaratively; the Model holds domain data and logic.
- **Use when**: data-binding UI frameworks (SwiftUI, Jetpack Compose, WPF, Angular, Vue).
- **Avoid when**: ViewModels grow into god objects or contain networking/persistence — inject services instead. Keep Views free of business logic.
```ts
class ProfileViewModel { name = ""; isSaving = false; constructor(private users: UserService) {} async save() { /* ... */ } }
```

## Event-Driven Architecture
- **Intent**: components communicate by publishing and reacting to events instead of calling each other directly.
- **Use when**: decoupled modules or services, many reactions to one fact, asynchronous workflows.
- **Avoid when**: a simple synchronous call is clearer; consider debugging, ordering, idempotency and delivery guarantees before adopting it.
```ts
bus.publish(new OrderPlaced(order.id)); // inventory, mailing, analytics subscribe independently
```

## CQRS (Command Query Responsibility Segregation)
- **Intent**: separate the model that changes state (commands) from the model that reads state (queries).
- **Use when**: read and write workloads differ strongly in shape, scale or performance needs.
- **Avoid when**: ordinary CRUD — it adds significant complexity. A lightweight form (separate query functions/DTOs for reads) is often enough.

## Event Sourcing
- **Intent**: store state as an append-only sequence of events and derive current state by replaying them.
- **Use when**: full audit history, temporal queries, or rebuilding projections are core requirements.
- **Avoid when**: the domain does not need history; schema evolution of events, projections and eventual consistency carry real cost.
```ts
const account = events.reduce(apply, Account.empty()); // apply(state, Deposited | Withdrawn)
```
