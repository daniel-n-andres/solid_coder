---
name: review
description: Review code against SOLID, DRY, KISS, YAGNI, Law of Demeter, Clean Code and design patterns, then apply the fixes directly.
argument-hint: "[path ... | git-range]  (default: uncommitted changes)"
disable-model-invocation: true
allowed-tools: Read, Grep, Glob
---

# SOLID review and fix

Review the requested code for design-principle violations and fix them directly in the working tree.

Arguments: `$ARGUMENTS`

## 1. Determine the scope

- If an argument exists as a path (`test -e`), review that file or directory.
- Otherwise, if it is a git range (contains `..`) or resolves with `git rev-parse --verify`, review the files changed in that range or commit.
- If there are no arguments, review uncommitted changes: the union of `git diff --name-only HEAD` (skip this part in a repository without commits) and untracked files from `git ls-files --others --exclude-standard`. If that is empty or the folder is not a git repository, ask the user which path to review and stop.

Keep only source-code files. State the resolved scope in one line.

## 2. Get the findings

Dispatch the `solid-coder:solid-reviewer` agent with a task message containing:
- the resolved list of files or the git range,
- the references directory: `${CLAUDE_PLUGIN_ROOT}/skills/solid-coding/references`,
- for a diff-based scope, the instruction to focus on the changed lines and their immediate design context.

For a large scope (more than ~25 files), split it into groups by directory and dispatch one agent per group in parallel.

## 3. Apply the fixes

Before editing, check `git status`. If any file in scope has uncommitted changes that are not part of this review's scope, mention it.

Apply the findings directly, highest severity first:
- Make behaviour-preserving refactorings only. Do not change public APIs used outside the scope unless the finding requires it; if it does, update every call site found with Grep.
- Follow the project's existing conventions and formatter.
- Skip a finding if applying it would require a design decision only the user can make (for example, choosing between two architectures) or touching code outside the repository; list it under "Not applied" with the reason.
- Skip findings you verify to be wrong; list them under "Not applied" as well.

## 4. Verify

Detect the project's test and build commands (package.json scripts, Makefile, pyproject/tox, Cargo, Gradle, Swift Package, etc.). Run the relevant tests or build. If they fail because of the refactoring, fix or revert that specific change. If no tests exist, say so.

## 5. Report

Reply with:
- **Scope**: what was reviewed.
- **Applied**: one line per fix — `path:line` · principle · what changed.
- **Not applied**: one line per skipped finding with the reason.
- **Verification**: the command run and its result (pass/fail with failing output), or that no tests exist.

Do not commit.
