---
name: typescript-refactoring
description: Refactor existing TypeScript or JavaScript without unintended behavior changes. Use for cleanup, duplication, deep nesting, large components, tangled services, complex types, hard-to-test code, or a refactoring plan. Diagnose the actual problem, preserve public contracts, and make small verified changes. Not a substitute for feature design, bug fixing, or a toolchain migration.
---

# TypeScript Refactoring

## Quick Start

1. Identify the dominant smell before changing structure.
2. Define scope, boundaries, and non-goals.
3. Choose one refactoring strategy that fits the smell.
4. State the safety net: tests, characterization tests, or explicit verification steps.
5. Make one conceptual transformation at a time.
6. Verify after each meaningful step.

For a small local change, a short scope and verification note is enough. Use a written plan only for multi-step, cross-module, or risky work; do not impose a six-part response on every cleanup.

Read references selectively from the symptom table below. They are a technique catalog, not a checklist of changes to apply.

## Core Workflow

### 1. Diagnose Before You Rearrange

Do not start with generic cleanup. First decide what kind of problem this is.

- Long function or deep nesting
- Duplication
- Mixed responsibilities
- Side effects tangled with business logic
- Poor naming masking unclear responsibilities
- Complex or misleading types
- Module-boundary or architecture pressure
- Legacy code with weak tests or unclear behavior

If the request is vague, translate it into a concrete target smell before making changes.

### 2. Define Boundaries

Identify the junction points between the code being changed and the rest of the system.

- Preserve public entry points by default unless the user wants API changes.
- Name what must stay behaviorally stable: values (including falsy and nullish inputs), errors, side-effect order, timing, mutation, and identity where observable.
- State what is out of scope.
- If architecture might change, start with the smallest seam that buys safety.

See [references/01-preparation.md](references/01-preparation.md) and [references/02-planning-workflow.md](references/02-planning-workflow.md).

### 3. Choose A Proportional Safety Net

Use the best available safety net instead of assuming ideal test coverage.

- If good tests exist, run and extend them around the touched behavior.
- If tests are weak, add characterization tests around boundaries and risky cases.
- If tests are missing, slow down and use narrow changes plus explicit manual verification checkpoints.
- Treat compiler errors, type errors, and lint failures as guardrails, not annoyances.

Do not use missing tests as permission for a rewrite.

### 4. Refactor In Small, Reversible Steps

Prefer a sequence of understandable moves over a clever rewrite.

- One conceptual transformation at a time
- Keep behavior stable
- Re-run relevant checks after meaningful changes
- Keep test refactors separate from production refactors when practical
- Stop when the original pain point is materially reduced

See [references/03-process-heuristics.md](references/03-process-heuristics.md).

## High-Value First Moves

| Symptom | First move | Then read |
|---------|------------|-----------|
| Long functions, deep nesting | Extract a pure helper or add guard clauses to flatten control flow | [07-conditions](references/07-conditions.md), [06-duplication](references/06-duplication.md) |
| Hard to test | Isolate side effects from decision logic and identify the seam that blocks testing | [10-side-effects](references/10-side-effects.md), [09-abstraction](references/09-abstraction.md) |
| Copy-paste code | Classify duplication as identical, similar, or conceptual before extracting anything | [06-duplication](references/06-duplication.md) |
| Unclear names | Rename to reveal intent and check whether bad names hide mixed responsibilities | [05-names](references/05-names.md) |
| Complex types / generics | Simplify contracts first; reduce `any`, assertions, or overly clever conditional types | [12-generics](references/12-generics.md), [16-static-typing](references/16-static-typing.md) |
| Tangled modules | Identify dependency direction and extract a stable seam or contract | [13-module-integration](references/13-module-integration.md), [14-architecture](references/14-architecture.md) |
| Imperative loops everywhere | Use a clearer data flow only if it improves readability, not because pipelines look nicer | [08-functional-pipeline](references/08-functional-pipeline.md), [15-declarative-style](references/15-declarative-style.md) |
| Messy error handling | Separate expected failures from exceptional ones and clarify handling boundaries | [11-error-handling](references/11-error-handling.md) |
| Outdated formatting / lint issues | Take easy compiler, formatter, and lint wins before deeper refactors | [04-low-hanging-fruit](references/04-low-hanging-fruit.md) |
| Large-scale migration pressure | Start with a façade, seam, or strangler-style boundary, not a big-bang rewrite | [14-architecture](references/14-architecture.md) |

Treat these as options, not diagnoses from file length or syntax alone. A loop, switch, long function, or repeated block can be the clearest implementation. Preserve it if changing it does not solve the user's problem.

## Real-World Cases

For legacy code, characterize current behavior (including suspicious quirks) before changing it. Name concrete checks and unknowns instead of claiming equivalence from types alone. Vague cleanup requests justify a small local improvement, not a broad redesign.

### Refactor + Feature Requests

If the user asks for both refactoring and new behavior:

- separate structural cleanup from behavior changes in small phases or review sections
- refactor first only when it makes the requested behavior change safer or simpler
- keep bug fixes explicit rather than presenting them as a pure refactor

## TypeScript-Specific Guidance

- Use type errors as feedback about contracts and design pressure.
- Prefer narrowing, helper types, and clearer domain types over `as any` or broad assertions.
- Preserve useful caller-facing type information during incremental changes.
- Avoid introducing generic abstractions until simpler concrete types stop being enough.

See [references/16-static-typing.md](references/16-static-typing.md) and [references/12-generics.md](references/12-generics.md).

## Do Not Do This

- Do not mix bug fixes or feature work into a refactor unless the user asked for it.
- Do not rename, redesign, and repartition code all at once unless a smaller sequence is clearly impossible.
- Do not introduce new layers, interfaces, or patterns before you can explain the coupling or duplication they remove.
- Do not widen types just to make the code compile.
- Do not force declarative or architectural patterns when a simpler local change solves the problem.
- Do not keep refactoring after the original pain point is clearly addressed.

## Supporting References

Use [test-code guidance](references/17-test-code.md) when reorganizing tests and [comments/docs guidance](references/18-comments-docs.md) when updating documentation. Other techniques are linked from the symptom table; read only those relevant to the current change.

## Verification And Handoff

- Establish the current check results before editing; distinguish pre-existing failures from regressions.
- Use the project's installed tools and scripts. Do not install a preferred formatter or reformat unrelated files.
- Review the diff for changed behavior, public types, accidental deletions, and unrelated churn.
- Report what became simpler, the checks actually run, and remaining uncertainty. Typechecking alone does not prove runtime equivalence.
- Keep commits and other repository operations within the user's authorization; example commit names are not instructions to commit.

## Templates And Quick Reference

- Plan template: [references/19-plan-template.md](references/19-plan-template.md)
- Cheatsheet: [references/20-cheatsheet.md](references/20-cheatsheet.md)

## Language Note

Examples use TypeScript. The principles apply broadly, but code patterns and tooling references here are TypeScript and JavaScript specific.
