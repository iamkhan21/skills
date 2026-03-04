---
name: typescript-refactoring
description: Apply systematic code refactoring with small steps, clear boundaries, and proven techniques for TypeScript and JavaScript codebases. Use when improving existing code, reducing technical debt, cleaning up legacy code, extracting functions, reducing complexity, splitting large files, decoupling modules, removing dead code, improving naming, eliminating duplication, simplifying conditionals, or when the user mentions code smells, technical debt, legacy code, or asks "how should I restructure this?" Also use when the user wants to plan a refactoring effort, even if they don't use the word "refactoring" explicitly.
---

# TypeScript Refactoring

## Quick Start

1. Define scope and boundaries
2. Cover with tests
3. Refactor in small steps (one technique per commit)
4. Verify after each change

## Language Note

Examples use TypeScript. The principles (small steps, test-driven, behavior preservation) apply to any language, but code patterns and tooling references are TypeScript/JavaScript specific.

## Workflow

### Before Refactoring

- Define scope, find boundaries (junction points between code to change and everything else)
- Cover with tests (edge cases, explicit/implicit input, specify desired result on boundaries)
- Run Biome or OxLint to catch low-hanging issues first

See [references/01-preparation.md](references/01-preparation.md)

### Planning a Refactor

- Gather detailed problem description
- Explore codebase to verify assertions
- Consider options and alternatives
- Interview user about implementation details
- Define exact scope (in/out)
- Check test coverage
- Break into tiny commits

See [references/02-planning-workflow.md](references/02-planning-workflow.md)

### During Refactoring

- Small atomic steps
- One technique per commit
- No features or bug fixes during refactor
- Test every change
- Refactor tests separately from production code

See [references/03-process-heuristics.md](references/03-process-heuristics.md)

## What to Refactor: Symptom Guide

| Symptom | Start with |
|---------|-----------|
| Long functions, deep nesting | [07-conditions](references/07-conditions.md), [06-duplication](references/06-duplication.md) |
| Hard to test | [10-side-effects](references/10-side-effects.md), [09-abstraction](references/09-abstraction.md) |
| Copy-paste code | [06-duplication](references/06-duplication.md) |
| Unclear names | [05-names](references/05-names.md) |
| Complex types / generics | [12-generics](references/12-generics.md), [16-static-typing](references/16-static-typing.md) |
| Tangled modules | [13-module-integration](references/13-module-integration.md), [14-architecture](references/14-architecture.md) |
| Imperative loops everywhere | [08-functional-pipeline](references/08-functional-pipeline.md) |
| Messy error handling | [11-error-handling](references/11-error-handling.md) |
| Outdated formatting / lint issues | [04-low-hanging-fruit](references/04-low-hanging-fruit.md) |
| Large-scale migration needed | [14-architecture](references/14-architecture.md) (strangler fig pattern) |

## Techniques Index

| Level | File | Techniques |
|-------|------|------------|
| Setup | [04-low-hanging-fruit.md](references/04-low-hanging-fruit.md) | Formatting, linting, language/API features |
| Code | [05-names.md](references/05-names.md) | Naming problems & solutions |
| Code | [06-duplication.md](references/06-duplication.md) | DRY, extraction techniques |
| Code | [07-conditions.md](references/07-conditions.md) | Guard clauses, complexity reduction |
| Code | [08-functional-pipeline.md](references/08-functional-pipeline.md) | Composition, data flow |
| Design | [09-abstraction.md](references/09-abstraction.md) | Abstraction levels, leak prevention |
| Design | [10-side-effects.md](references/10-side-effects.md) | Isolation, CQS |
| Design | [11-error-handling.md](references/11-error-handling.md) | Error types, strategies |
| Design | [12-generics.md](references/12-generics.md) | Generics, composition |
| Architecture | [13-module-integration.md](references/13-module-integration.md) | Coupling, cohesion |
| Architecture | [14-architecture.md](references/14-architecture.md) | Layers, dependencies, strangler fig |
| Architecture | [15-declarative-style.md](references/15-declarative-style.md) | Declarative patterns |
| Architecture | [16-static-typing.md](references/16-static-typing.md) | Type design |
| Support | [17-test-code.md](references/17-test-code.md) | Refactoring tests |
| Support | [18-comments-docs.md](references/18-comments-docs.md) | Documentation |

## Principles

- **Small steps**: Each commit leaves code in working state
- **One technique at a time**: Don't mix refactoring techniques
- **No scope creep**: No features or bug fixes during refactor
- **Test-driven**: Verify after every change
- **Know when to stop**: Stop when the original pain point is resolved, further changes risk behavior changes, or diminishing returns set in

## Templates

See [references/19-plan-template.md](references/19-plan-template.md) for refactor plan issue/PR template.

## Quick Reference

See [references/20-cheatsheet.md](references/20-cheatsheet.md) for technique summary.
