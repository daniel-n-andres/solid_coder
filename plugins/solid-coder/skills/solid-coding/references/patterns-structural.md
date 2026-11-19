# Structural patterns

Each entry: intent · use when · avoid when · sketch. Sketches use TypeScript syntax; adapt to the target language.

## Adapter
- **Intent**: convert one interface into the interface clients expect.
- **Use when**: integrating a third-party or legacy API whose shape should not leak into domain code.
- **Avoid when**: you own both sides — change one of them instead.
```ts
class StripeGateway implements PaymentGateway {
  constructor(private stripe: Stripe) {}
  charge(amount: Money, card: CardToken) { return this.stripe.charges.create({ amount: amount.cents, source: card.id }); }
}
```

## Bridge
- **Intent**: separate an abstraction from its implementation so both can vary independently.
- **Use when**: two independent dimensions would otherwise multiply subclasses (shape × renderer, report × output format).
- **Avoid when**: only one dimension varies.
```ts
abstract class Report { constructor(protected out: Formatter) {} abstract render(): string }
class SalesReport extends Report { render() { return this.out.table(this.rows()); } }
```

## Composite
- **Intent**: treat individual objects and groups of objects uniformly via a tree.
- **Use when**: part–whole hierarchies (file systems, UI trees, org charts, nested price items).
- **Avoid when**: the structure is flat or leaves and groups need very different operations.
```ts
interface Item { price(): Money }
class Bundle implements Item { constructor(private items: Item[]) {} price() { return sum(this.items.map(i => i.price())); } }
```

## Decorator
- **Intent**: attach behaviour to an object dynamically by wrapping it with the same interface.
- **Use when**: optional, stackable cross-cutting concerns (caching, logging, retry, metrics, authorization).
- **Avoid when**: the order of wrappers matters in fragile ways or there is a single fixed combination.
```ts
class CachedRepo implements Repo { constructor(private inner: Repo, private cache: Cache) {} get(id: Id) { return this.cache.getOr(id, () => this.inner.get(id)); } }
```

## Facade
- **Intent**: provide one simple entry point to a complex subsystem.
- **Use when**: callers need a common use case of a subsystem with many parts.
- **Avoid when**: it turns into a god object; keep it thin and task-focused.
```ts
class CheckoutFacade { placeOrder(cart: Cart) { this.stock.reserve(cart); this.payments.charge(cart.total); this.shipping.schedule(cart); } }
```

## Flyweight
- **Intent**: share common intrinsic state among many fine-grained objects to save memory.
- **Use when**: very large numbers of similar objects (glyphs, map tiles, particles) with measured memory pressure.
- **Avoid when**: object counts are modest — it adds complexity for no gain.
```ts
const glyph = glyphCache.get(char); glyph.draw(ctx, x, y); // position is extrinsic, shape is shared
```

## Proxy
- **Intent**: stand in for another object to control access to it (lazy loading, remote access, access control, rate limiting).
- **Use when**: the real object is expensive, remote or needs guarding, and callers should not change.
- **Avoid when**: a Decorator (adding behaviour) or a plain function call expresses the intent better.
```ts
class LazyImage implements Image { private real?: RealImage; constructor(private path: string) {} draw() { (this.real ??= new RealImage(this.path)).draw(); } }
```
