---
name: typescript-refactoring
description: Use this skill when improving existing TypeScript or JavaScript code without intentionally changing product behavior. Reach for it whenever the user says a file or module is too big, messy, brittle, hard to test, full of duplication, deeply nested, over-coupled, confusingly named, legacy, or in need of cleanup, restructuring, simplification, or technical-debt reduction. Also use it when the user asks how to safely split a large component, untangle a service, reduce complexity, isolate side effects, simplify types, or plan a refactor, even if they never say the word "refactor."
---

# TypeScript Refactoring

## What This Skill Does

Use this skill to make code easier to understand, test, and change while preserving behavior unless the user explicitly asks for behavior changes.

This skill is especially helpful when the user says things like:

- "clean this up"
- "this component is doing too much"
- "this file is too big"
- "please untangle this"
- "reduce complexity without changing behavior"
- "make this easier to test"
- "how should I restructure this?"

## Quick Start

1. Identify the dominant smell before changing structure.
2. Define scope, boundaries, and non-goals.
3. Choose one refactoring strategy that fits the smell.
4. State the safety net: tests, characterization tests, or explicit verification steps.
5. Make one conceptual transformation at a time.
6. Verify after each meaningful step.

## Response Shape

When responding to the user, prefer this structure:

1. Name the main pain point or smell.
2. State what is in scope and what is not.
3. Explain the first refactoring move and why it is the safest useful step.
4. Explain how behavior will be preserved.
5. Apply or describe incremental changes.
6. Call out risks, tradeoffs, and logical next steps.

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
- Name what must stay behaviorally stable.
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

## Real-World Cases

### Low-Test Or Legacy Code

When behavior is unclear, caution is part of the refactor.

- Add characterization tests for the behavior that already exists.
- Name the risky behavior surface before changing structure: inputs, outputs, side effects, ordering, fallbacks, errors.
- Preserve suspicious legacy behavior unless the user explicitly wants a bug fix.
- Document assumptions and unknowns instead of bluffing certainty.
- Prefer narrow seams and reversible steps.

When describing the safety net, name a few concrete checks rather than saying "add tests" in the abstract.

### Vague Cleanup Requests

If the user says "make it cleaner" or "tidy this up":

- identify the likely dominant smell
- state the non-goals
- choose the smallest change that meaningfully reduces pain

Do not turn a vague request into a broad redesign.

### Refactor + Feature Requests

If the user asks for both refactoring and new behavior:

- acknowledge both goals
- separate structural cleanup from feature work
- prefer refactor-first then feature, or at least clearly distinct phases
- make the packaging explicit: separate commits, review sections, or staged phases
- keep the first phase small and local

Do not present combined feature work as a pure refactor.

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

## Techniques Index

| Level | File | Techniques |
|-------|------|------------|
| Setup | [04-low-hanging-fruit.md](references/04-low-hanging-fruit.md) | Formatting, linting, language/API features |
| Code | [05-names.md](references/05-names.md) | Naming problems and solutions |
| Code | [06-duplication.md](references/06-duplication.md) | DRY, extraction techniques |
| Code | [07-conditions.md](references/07-conditions.md) | Guard clauses, complexity reduction |
| Code | [08-functional-pipeline.md](references/08-functional-pipeline.md) | Composition, data flow |
| Design | [09-abstraction.md](references/09-abstraction.md) | Abstraction levels, leak prevention |
| Design | [10-side-effects.md](references/10-side-effects.md) | Isolation, command-query separation |
| Design | [11-error-handling.md](references/11-error-handling.md) | Error types, strategies |
| Design | [12-generics.md](references/12-generics.md) | Generics, composition |
| Architecture | [13-module-integration.md](references/13-module-integration.md) | Coupling, cohesion |
| Architecture | [14-architecture.md](references/14-architecture.md) | Layers, dependencies, strangler fig |
| Architecture | [15-declarative-style.md](references/15-declarative-style.md) | Declarative patterns |
| Architecture | [16-static-typing.md](references/16-static-typing.md) | Type design |
| Support | [17-test-code.md](references/17-test-code.md) | Refactoring tests |
| Support | [18-comments-docs.md](references/18-comments-docs.md) | Documentation |

## Principles

- **Diagnose first**: pick the smell before picking the move.
- **Small steps**: each step should be understandable and reversible.
- **Behavior preservation**: refactoring changes structure, not product intent.
- **Scope discipline**: keep in-scope and out-of-scope explicit.
- **Practicality over purity**: prefer the simplest change that relieves the real pain.
- **Know when to stop**: stop when the code is materially easier to change and further cleanup has poor payoff.

## Templates And Quick Reference

- Plan template: [references/19-plan-template.md](references/19-plan-template.md)
- Cheatsheet: [references/20-cheatsheet.md](references/20-cheatsheet.md)

## Language Note

Examples use TypeScript. The principles apply broadly, but code patterns and tooling references here are TypeScript and JavaScript specific.
