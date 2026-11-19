# solid_coder

Claude Code plugin marketplace **daniel-andres-plugins** with two plugins that make Claude write well-structured code.

## What the plugins do

**[solid-coder](plugins/solid-coder)** changes how Claude writes code. Whenever Claude writes, designs or refactors code, it loads the [`solid-coding` skill](plugins/solid-coder/skills/solid-coding/SKILL.md): it plans where each responsibility belongs, applies the principles below while writing, picks a design pattern only when a concrete code smell calls for it (KISS and YAGNI come first), and re-checks its change against the checklist before reporting the work as done. The plugin also adds [`/solid-coder:review`](plugins/solid-coder/skills/review/SKILL.md), which reviews existing code (by default your uncommitted changes) with the read-only [`solid-reviewer` agent](plugins/solid-coder/agents/solid-reviewer.md), applies the fixes directly, runs your tests, and reports what changed.

**[solid-coder-guard](plugins/solid-coder-guard)** (optional) checks the result. After every `Write`/`Edit`/`MultiEdit`, a [hook](plugins/solid-coder-guard/hooks/hooks.json) sends the new code to a separate model call. If it finds a clear, significant violation, Claude receives a one-line finding (for example `solid-coder: order.ts: Open/Closed - shippingCost(); extract a ShippingMethod strategy map`) and fixes it right away. Non-code files, comment-only edits and refactorings are skipped. It costs one extra model call per edit.

| Plugin | Contains |
|---|---|
| [solid-coder](plugins/solid-coder) | `solid-coding` skill (automatic), `/solid-coder:review` command, `solid-reviewer` agent |
| [solid-coder-guard](plugins/solid-coder-guard) | `PostToolUse` edit hook |

## Principles included

Each entry links to its section in the skill's reference files, with violation signals, the remedy and a short code sketch. The skill's [checklist and smell-to-pattern table](plugins/solid-coder/skills/solid-coding/SKILL.md) tie them together.

| Principle | What Claude does |
|---|---|
| [Single Responsibility (S)](plugins/solid-coder/skills/solid-coding/references/solid.md#s--single-responsibility-principle) | Each class, module or function has one reason to change. |
| [Open/Closed (O)](plugins/solid-coder/skills/solid-coding/references/solid.md#o--openclosed-principle) | New variants are added as new code, not by editing growing `if`/`switch` chains. |
| [Liskov Substitution (L)](plugins/solid-coder/skills/solid-coding/references/solid.md#l--liskov-substitution-principle) | Subtypes can replace their base type without surprising callers. |
| [Interface Segregation (I)](plugins/solid-coder/skills/solid-coding/references/solid.md#i--interface-segregation-principle) | Small, role-based interfaces instead of wide ones clients only partly use. |
| [Dependency Inversion (D)](plugins/solid-coder/skills/solid-coding/references/solid.md#d--dependency-inversion-principle) | Business logic depends on abstractions; databases, HTTP, files and clocks are injected. |
| [DRY](plugins/solid-coder/skills/solid-coding/references/dry-kiss-yagni.md#dry--dont-repeat-yourself) | Each rule, constant or algorithm lives in one place; look-alike code with different reasons to change stays separate. |
| [KISS](plugins/solid-coder/skills/solid-coding/references/dry-kiss-yagni.md#kiss--keep-it-simple) | The simplest construct that solves today's problem; no needless layers or cleverness. |
| [YAGNI](plugins/solid-coder/skills/solid-coding/references/dry-kiss-yagni.md#yagni--you-arent-gonna-need-it) | No code for requirements that don't exist yet. |
| [Composition over inheritance](plugins/solid-coder/skills/solid-coding/references/composition-and-demeter.md#composition-over-inheritance) | Combine small objects instead of inheriting just to reuse code. |
| [Law of Demeter](plugins/solid-coder/skills/solid-coding/references/composition-and-demeter.md#law-of-demeter-principle-of-least-knowledge) | Talk only to direct collaborators; no `a.b().c().d()` reach-through ("tell, don't ask"). |
| [Null Object](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#null-object) | A do-nothing implementation instead of `null` where absence has a neutral meaning. |
| [Clean Code](plugins/solid-coder/skills/solid-coding/references/clean-code.md) | [Naming](plugins/solid-coder/skills/solid-coding/references/clean-code.md#naming) · [Functions](plugins/solid-coder/skills/solid-coding/references/clean-code.md#functions) · [Values and types](plugins/solid-coder/skills/solid-coding/references/clean-code.md#values-and-types) · [Error handling](plugins/solid-coder/skills/solid-coding/references/clean-code.md#error-handling) · [Comments and structure](plugins/solid-coder/skills/solid-coding/references/clean-code.md#comments-and-structure) · [Tests](plugins/solid-coder/skills/solid-coding/references/clean-code.md#tests) |

## Design patterns included

Each pattern has its intent, when to use it, when to avoid it, and a code sketch.

- **[Creational](plugins/solid-coder/skills/solid-coding/references/patterns-creational.md)**: [Factory Method](plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#factory-method) · [Abstract Factory](plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#abstract-factory) · [Builder](plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#builder) · [Prototype](plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#prototype) · [Singleton](plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#singleton) · [Object Pool](plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#object-pool) · [Lazy Initialization](plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#lazy-initialization)
- **[Structural](plugins/solid-coder/skills/solid-coding/references/patterns-structural.md)**: [Adapter](plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#adapter) · [Bridge](plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#bridge) · [Composite](plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#composite) · [Decorator](plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#decorator) · [Facade](plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#facade) · [Flyweight](plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#flyweight) · [Proxy](plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#proxy)
- **[Behavioral](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md)**: [Chain of Responsibility](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#chain-of-responsibility) · [Command](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#command) · [Iterator](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#iterator) · [Mediator](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#mediator) · [Memento](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#memento) · [Observer](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#observer) · [State](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#state) · [Strategy](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#strategy) · [Template Method](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#template-method) · [Visitor](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#visitor) · [Null Object](plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#null-object)
- **[Architectural](plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md)**: [Dependency Injection](plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#dependency-injection) · [Repository](plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#repository) · [Service Layer](plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#service-layer) · [MVC](plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#mvc-modelviewcontroller) · [MVVM](plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#mvvm-modelviewviewmodel) · [Event-Driven Architecture](plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#event-driven-architecture) · [CQRS](plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#cqrs-command-query-responsibility-segregation) · [Event Sourcing](plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#event-sourcing)

## Install

### 1. Add the marketplace (once)

In a Claude Code session:

```
/plugin marketplace add daniel-n-andres/solid_coder
```

Or from a terminal:

```bash
claude plugin marketplace add daniel-n-andres/solid_coder
```

### 2. Install the plugin (skill, review command, reviewer agent)

In a Claude Code session:

```
/plugin install solid-coder@daniel-andres-plugins
```

Or from a terminal:

```bash
claude plugin install solid-coder@daniel-andres-plugins
```

### 3. Install the edit hook (optional)

`solid-coder-guard` is available only from this repository's marketplace. Anthropic's plugin directory accepts hooks that can steer Claude only from Verified organizations, so the directory version of `solid-coder` ships without the edit hook. I'm working on getting verified so the edit hook can be listed in the directory as well.

The per-edit check is a separate plugin, `solid-coder-guard`. In a Claude Code session:

```
/plugin install solid-coder-guard@daniel-andres-plugins
```

Or from a terminal:

```bash
claude plugin install solid-coder-guard@daniel-andres-plugins
```

Start a new Claude Code session after installing so the skill, agent and hook load.

To pause the edit hook later:

```bash
claude plugin disable solid-coder-guard@daniel-andres-plugins
```

### Update

```bash
claude plugin marketplace update daniel-andres-plugins
```

## Support

Questions and feedback: [Daniel Andres on LinkedIn](https://www.linkedin.com/in/daniel-n-andres/) or [GitHub issues](https://github.com/daniel-n-andres/solid_coder/issues).

## License

MIT
