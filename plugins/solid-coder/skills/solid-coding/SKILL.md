---
name: solid-coding
description: This skill should be used whenever Claude writes, designs, extends or refactors source code in any language, including when the user asks to "implement a feature", "add a class/module/service", "refactor this", "clean up this code", "make this more maintainable", "which design pattern fits", "apply SOLID", "remove duplication", or mentions DRY, KISS, YAGNI, Law of Demeter, composition over inheritance, Clean Code, Null Object, or any Gang-of-Four or architectural pattern (Strategy, Factory, Observer, Repository, CQRS, MVVM, ...). It provides the quality bar and a smell-to-pattern guide for producing well-structured code.
---

# Solid Coding

Apply these principles to every piece of code written or changed. They are language-agnostic; express them in the idioms of the language at hand (interfaces/protocols/traits/duck typing, classes or plain functions and modules).

## Working method

1. **Before writing**: identify the responsibilities involved and where each one belongs. Check whether the change extends existing behaviour (prefer adding a new unit over editing a stable one) and whether the codebase already has an abstraction to reuse.
2. **While writing**: keep each unit to one reason to change, depend on abstractions at boundaries (I/O, frameworks, external services), keep functions small with intention-revealing names, and handle absence and errors explicitly.
3. **After writing**: re-read the change against the checklist below and fix violations before reporting the work as done.
4. **Choose the simplest design that meets today's requirements.** KISS and YAGNI outrank patterns: introduce a pattern only when a concrete smell from the table below is present, not in anticipation of one. Match the existing conventions of the codebase.

## Checklist

| Principle | Ask | Detail |
|---|---|---|
| **S**ingle Responsibility | Does this unit have more than one reason to change? | `references/solid.md` |
| **O**pen/Closed | Will a new variant force editing this code (growing `if`/`switch` on type)? | `references/solid.md` |
| **L**iskov Substitution | Can every subtype replace its base without surprising callers? | `references/solid.md` |
| **I**nterface Segregation | Do clients depend on methods they never call? | `references/solid.md` |
| **D**ependency Inversion | Does policy code construct or import concrete infrastructure directly? | `references/solid.md` |
| DRY | Is the same *knowledge* (rule, constant, algorithm) written in two places? | `references/dry-kiss-yagni.md` |
| KISS | Is there a simpler construct that does the same job? | `references/dry-kiss-yagni.md` |
| YAGNI | Is any code here for a requirement that does not exist yet? | `references/dry-kiss-yagni.md` |
| Composition over inheritance | Is inheritance used only to reuse code rather than to model "is-a"? | `references/composition-and-demeter.md` |
| Law of Demeter | Are there call chains reaching through objects (`a.b().c().d()`)? | `references/composition-and-demeter.md` |
| Clean Code | Names, function size, nesting depth, magic values, error handling, comments | `references/clean-code.md` |
| Null handling | Are there repeated `null`/`nil`/`None` checks for "no-op" behaviour? | `references/patterns-behavioral.md` (Null Object) |

## Smell → remedy guide

| Smell in the code | Consider |
|---|---|
| `switch`/`if` chain on a type or mode, repeated in several places | Polymorphism, **Strategy**, **State** |
| Object behaves differently depending on an internal status field | **State** |
| Constructor with many optional parameters or telescoping overloads | **Builder** |
| `new ConcreteThing()` scattered through business logic | **Factory Method**, **Abstract Factory**, **Dependency Injection** |
| Repeated `if (x != null)` before calling a no-op default | **Null Object** |
| Third-party/legacy API shape leaking into domain code | **Adapter**, **Facade** |
| Many cross-cutting add-ons (logging, caching, retry) wrapped around one call | **Decorator**, **Proxy** |
| Class explosion from combining two independent dimensions | **Bridge** |
| Tree of items treated differently from single items | **Composite** |
| Many objects calling each other in a tangled web | **Mediator**, **Observer** / events |
| Several objects must react when one changes | **Observer**, **Event-Driven** |
| Undo/redo, queued or logged operations | **Command**, **Memento** |
| Sequence of handlers, each may handle or pass on | **Chain of Responsibility** |
| Algorithms identical except for a few steps | **Template Method** (or Strategy for composition) |
| New operations keep being added over a stable class hierarchy | **Visitor** |
| Data access (SQL/HTTP/ORM) mixed into business logic | **Repository**, **Service Layer** |
| UI code holding business rules | **MVC** / **MVVM** |
| Read and write paths with very different shapes or load | **CQRS** (only at real scale) |
| Expensive object created repeatedly / created but often unused | **Object Pool**, **Lazy Initialization**, **Flyweight**, **Prototype** |
| Global mutable state accessed from everywhere | Replace **Singleton**/globals with **Dependency Injection** |

Pattern details (intent, use when, avoid when, sketch):

- `references/patterns-creational.md` — Factory Method, Abstract Factory, Builder, Prototype, Singleton, Object Pool, Lazy Initialization
- `references/patterns-structural.md` — Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy
- `references/patterns-behavioral.md` — Chain of Responsibility, Command, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor, Null Object
- `references/patterns-architectural.md` — Dependency Injection, Repository, Service Layer, MVC, MVVM, Event-Driven Architecture, CQRS, Event Sourcing

Load only the reference file needed for the decision at hand.

## Guardrails

- Do not refactor unrelated code while implementing a feature; mention notable violations instead, unless the user asked for a cleanup.
- Do not add an interface with a single implementation unless it sits at an I/O or framework boundary, or tests need a seam.
- Prefer small, behaviour-preserving steps; keep existing tests passing and add tests for extracted units where the project has a test suite.
- When two principles conflict (for example DRY vs. decoupling two modules that only look alike), state the trade-off briefly and pick the option with fewer reasons to change together.
