# Creational patterns

Each entry: intent · use when · avoid when · sketch. Sketches use TypeScript syntax; adapt to the target language.

## Factory Method
- **Intent**: let a creator defer which concrete product to instantiate to a subclass or a supplied function.
- **Use when**: callers should not know the concrete type; the product type varies by context or configuration.
- **Avoid when**: there is one product and no foreseeable variation — call the constructor.
```ts
interface Notifier { send(msg: string): void }
function createNotifier(channel: "email" | "sms"): Notifier {
  return channel === "email" ? new EmailNotifier() : new SmsNotifier();
}
```

## Abstract Factory
- **Intent**: create families of related objects through one interface, without naming their concrete classes.
- **Use when**: products must be used together consistently (UI widgets per theme, clients per cloud vendor).
- **Avoid when**: only one family exists; a single Factory Method suffices.
```ts
interface StorageFactory { blobs(): BlobStore; queue(): Queue }
class AwsFactory implements StorageFactory { blobs() { return new S3(); } queue() { return new Sqs(); } }
```

## Builder
- **Intent**: construct a complex object step by step; the same process can produce different representations.
- **Use when**: many optional parameters, telescoping constructors, or validation that spans several fields.
- **Avoid when**: the language has named/default arguments or record literals that already express it clearly.
```ts
const req = new HttpRequestBuilder("GET", url).header("Accept", "json").timeout(5_000).build();
```

## Prototype
- **Intent**: create new objects by copying a configured existing instance.
- **Use when**: construction is expensive or configuration-heavy, and variations differ little from a template.
- **Avoid when**: objects hold resources or deep graphs where copy semantics are unclear.
```ts
const draft = templateInvoice.clone(); draft.customer = c;
```

## Singleton
- **Intent**: ensure exactly one instance with a global access point.
- **Use when**: rarely — truly process-wide, stateless or read-only resources where the platform requires it.
- **Avoid when**: almost always. It is hidden global state, hurts testability and violates DIP. Prefer creating one instance in the composition root and injecting it.
```ts
// Prefer: const config = loadConfig(); new App(config)  — one instance, passed explicitly
```

## Object Pool
- **Intent**: reuse a set of pre-created, expensive objects instead of creating and destroying them.
- **Use when**: creation is costly and usage is short and frequent (DB connections, threads, large buffers).
- **Avoid when**: objects are cheap; the runtime or library already pools (most DB drivers, HTTP clients). Ensure objects are reset before reuse.
```ts
const conn = await pool.acquire(); try { await conn.query(q); } finally { pool.release(conn); }
```

## Lazy Initialization
- **Intent**: delay creating a value until it is first needed.
- **Use when**: creation is expensive and the value is often unused.
- **Avoid when**: the cost is trivial, or first-use latency/thread-safety matters more than startup cost. Use the language's built-in lazy constructs where available.
```ts
class Report { private _data?: Data; get data() { return (this._data ??= loadData()); } }
```
