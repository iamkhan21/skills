---
name: refactoring
description: Apply systematic code refactoring with small steps, clear boundaries, and proven techniques. Use when improving existing code, reducing technical debt, cleaning up legacy code, or when user mentions refactoring, code cleanup, or code improvement.
---

# Refactoring

## Quick Start

1. Define scope and boundaries
2. Cover with tests
3. Refactor in small steps (one technique per commit)
4. Verify after each change

## Workflow

### Before Refactoring

- Define scope, find boundaries (junction points between code to change and everything else)
- Cover with tests (edge cases, explicit/implicit input, specify desired result on boundaries)
- Make linter/compiler stricter

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
| Architecture | [14-architecture.md](references/14-architecture.md) | Layers, dependencies |
| Architecture | [15-declarative-style.md](references/15-declarative-style.md) | Declarative patterns |
| Architecture | [16-static-typing.md](references/16-static-typing.md) | Type design |
| Support | [17-test-code.md](references/17-test-code.md) | Refactoring tests |
| Support | [18-comments-docs.md](references/18-comments-docs.md) | Documentation |

## Principles

- **Small steps**: Each commit leaves code in working state (Fowler: "Make each refactoring step as small as possible, so that you can always see the program working")
- **One technique at a time**: Don't mix refactoring techniques
- **No scope creep**: No features or bug fixes during refactor
- **Test-driven**: Verify after every change

## Templates

See [references/19-plan-template.md](references/19-plan-template.md) for refactor plan issue/PR template.

## Quick Reference

See [references/20-cheatsheet.md](references/20-cheatsheet.md) for technique summary.
