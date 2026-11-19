# solid-coder-guard

Companion to [solid-coder](../solid-coder). After every `Write`, `Edit` or `MultiEdit` in Claude Code, a separate model call checks the new code against SOLID, DRY, KISS, YAGNI, composition over inheritance, Law of Demeter, Clean Code, null handling and design-pattern misuse. Clear, significant violations are fed back to Claude as a one-line finding prefixed with `solid-coder:`, so Claude can fix them right away.

Works on its own, and works best together with `solid-coder`, whose skill guides how code is written while this hook checks the result.

## What is skipped

- Non-code files: Markdown, text, JSON, YAML, TOML, INI, CSV, lockfiles, `.env`, licenses, images, generated or vendored files.
- Edits that only change comments or docstrings, or comment code in or out.
- Whitespace, formatting, import ordering and plain renames.
- Edits that extract, rename, move or simplify existing logic, so fixes are not re-flagged.

For `Edit`/`MultiEdit` the hook only sees the changed fragment, so it does not judge function length, nesting depth, DRY or single responsibility there.

## Installation

`solid-coder-guard` is available only from this repository's marketplace. Anthropic's plugin directory accepts hooks that can steer Claude only from Verified organizations, so the directory version of `solid-coder` ships without the edit hook. I'm working on getting verified so the edit hook can be listed in the directory as well.

In a Claude Code session:

```
/plugin marketplace add daniel-n-andres/solid_coder
/plugin install solid-coder-guard@daniel-andres-plugins
```

Or from a terminal:

```bash
claude plugin marketplace add daniel-n-andres/solid_coder
claude plugin install solid-coder-guard@daniel-andres-plugins
```

Start a new Claude Code session after installing. To pause the hook: `claude plugin disable solid-coder-guard@daniel-andres-plugins`.

## Data handling

The hook sends the file path and the edited code of each `Write`/`Edit`/`MultiEdit` to a model call made by Claude Code itself (the same provider Claude Code already uses). It stores nothing and contacts no other service. The edited code is wrapped in a block that the prompt treats as data only.

## Cost and tuning

One extra model call per edit; when it reports a violation, Claude spends another turn fixing it. Strictness and skip rules live in the `prompt` in `hooks/hooks.json`. To pause it, disable the plugin with `/plugin`.

## License

MIT
