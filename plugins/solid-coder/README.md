# solid-coder

A Claude Code plugin that holds Claude to established software-design principles while it writes code, and reviews and fixes existing code against them. Language-agnostic: the guidance is applied in the idioms of whatever language the project uses.

## What it does

**[solid-coder](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder)** changes how Claude writes code. Whenever Claude writes, designs or refactors code, it loads the [`solid-coding` skill](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/SKILL.md): it plans where each responsibility belongs, applies the principles below while writing, picks a design pattern only when a concrete code smell calls for it (KISS and YAGNI come first), and re-checks its change against the checklist before reporting the work as done. The plugin also adds [`/solid-coder:review`](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/review/SKILL.md), which reviews existing code (by default your uncommitted changes) with the read-only [`solid-reviewer` agent](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/agents/solid-reviewer.md), applies the fixes directly, runs your tests, and reports what changed.

**[solid-coder-guard](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder-guard)** (optional) checks the result. After every `Write`/`Edit`/`MultiEdit`, a [hook](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder-guard/hooks/hooks.json) sends the new code to a separate model call. If it finds a clear, significant violation, Claude receives a one-line finding (for example `solid-coder: order.ts: Open/Closed - shippingCost(); extract a ShippingMethod strategy map`) and fixes it right away. Non-code files, comment-only edits and refactorings are skipped. It costs one extra model call per edit.

| Plugin | Contains |
|---|---|
| [solid-coder](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder) | `solid-coding` skill (automatic), `/solid-coder:review` command, `solid-reviewer` agent |
| [solid-coder-guard](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder-guard) | `PostToolUse` edit hook |

## Principles included

Each entry links to its section in the skill's reference files, with violation signals, the remedy and a short code sketch. The skill's [checklist and smell-to-pattern table](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/SKILL.md) tie them together.

| Principle | What Claude does |
|---|---|
| [Single Responsibility (S)](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/solid.md#s--single-responsibility-principle) | Each class, module or function has one reason to change. |
| [Open/Closed (O)](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/solid.md#o--openclosed-principle) | New variants are added as new code, not by editing growing `if`/`switch` chains. |
| [Liskov Substitution (L)](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/solid.md#l--liskov-substitution-principle) | Subtypes can replace their base type without surprising callers. |
| [Interface Segregation (I)](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/solid.md#i--interface-segregation-principle) | Small, role-based interfaces instead of wide ones clients only partly use. |
| [Dependency Inversion (D)](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/solid.md#d--dependency-inversion-principle) | Business logic depends on abstractions; databases, HTTP, files and clocks are injected. |
| [DRY](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/dry-kiss-yagni.md#dry--dont-repeat-yourself) | Each rule, constant or algorithm lives in one place; look-alike code with different reasons to change stays separate. |
| [KISS](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/dry-kiss-yagni.md#kiss--keep-it-simple) | The simplest construct that solves today's problem; no needless layers or cleverness. |
| [YAGNI](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/dry-kiss-yagni.md#yagni--you-arent-gonna-need-it) | No code for requirements that don't exist yet. |
| [Composition over inheritance](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/composition-and-demeter.md#composition-over-inheritance) | Combine small objects instead of inheriting just to reuse code. |
| [Law of Demeter](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/composition-and-demeter.md#law-of-demeter-principle-of-least-knowledge) | Talk only to direct collaborators; no `a.b().c().d()` reach-through ("tell, don't ask"). |
| [Null Object](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#null-object) | A do-nothing implementation instead of `null` where absence has a neutral meaning. |
| [Clean Code](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/clean-code.md) | [Naming](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/clean-code.md#naming) · [Functions](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/clean-code.md#functions) · [Values and types](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/clean-code.md#values-and-types) · [Error handling](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/clean-code.md#error-handling) · [Comments and structure](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/clean-code.md#comments-and-structure) · [Tests](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/clean-code.md#tests) |

## Design patterns included

Each pattern has its intent, when to use it, when to avoid it, and a code sketch.

- **[Creational](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-creational.md)**: [Factory Method](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#factory-method) · [Abstract Factory](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#abstract-factory) · [Builder](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#builder) · [Prototype](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#prototype) · [Singleton](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#singleton) · [Object Pool](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#object-pool) · [Lazy Initialization](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-creational.md#lazy-initialization)
- **[Structural](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-structural.md)**: [Adapter](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#adapter) · [Bridge](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#bridge) · [Composite](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#composite) · [Decorator](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#decorator) · [Facade](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#facade) · [Flyweight](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#flyweight) · [Proxy](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-structural.md#proxy)
- **[Behavioral](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md)**: [Chain of Responsibility](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#chain-of-responsibility) · [Command](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#command) · [Iterator](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#iterator) · [Mediator](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#mediator) · [Memento](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#memento) · [Observer](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#observer) · [State](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#state) · [Strategy](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#strategy) · [Template Method](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#template-method) · [Visitor](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#visitor) · [Null Object](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-behavioral.md#null-object)
- **[Architectural](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md)**: [Dependency Injection](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#dependency-injection) · [Repository](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#repository) · [Service Layer](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#service-layer) · [MVC](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#mvc-modelviewcontroller) · [MVVM](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#mvvm-modelviewviewmodel) · [Event-Driven Architecture](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#event-driven-architecture) · [CQRS](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#cqrs-command-query-responsibility-segregation) · [Event Sourcing](https://github.com/daniel-n-andres/solid_coder/blob/main/plugins/solid-coder/skills/solid-coding/references/patterns-architectural.md#event-sourcing)

## Installation

In a Claude Code session:

```
/plugin marketplace add daniel-n-andres/solid_coder
/plugin install solid-coder@daniel-andres-plugins
```

Or from a terminal:

```bash
claude plugin marketplace add daniel-n-andres/solid_coder
claude plugin install solid-coder@daniel-andres-plugins
```

`solid-coder-guard` is available only from this repository's marketplace. Anthropic's plugin directory accepts hooks that can steer Claude only from Verified organizations, so the directory version of `solid-coder` ships without the edit hook. I'm working on getting verified so the edit hook can be listed in the directory as well.

Optional per-edit check (edit hook), after adding the marketplace:

```
/plugin install solid-coder-guard@daniel-andres-plugins
```

```bash
claude plugin install solid-coder-guard@daniel-andres-plugins
```

Start a new Claude Code session after installing.

For local development, from a clone of this repository:

```bash
claude --plugin-dir ./plugins/solid-coder
```

## Usage

- Just write code with Claude — the `solid-coding` skill loads automatically.
- `/solid-coder:review` — review and fix all uncommitted changes.
- `/solid-coder:review src/billing` — review and fix a directory.
- `/solid-coder:review main..HEAD` — review and fix everything changed on the current branch.

## Data handling

The plugin reads no personal data, stores nothing, runs no scripts and contacts no external service. It consists only of instructions (skills and an agent) that Claude Code reads.

## Support

Questions and feedback: [Daniel Andres on LinkedIn](https://www.linkedin.com/in/daniel-n-andres/) or [GitHub issues](https://github.com/daniel-n-andres/solid_coder/issues).

## License

MIT
