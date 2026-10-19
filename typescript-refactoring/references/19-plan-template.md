# Refactor Plan Template

Use this for multi-step or risky refactors. For a local cleanup, a short scope and check list in chat is enough. Omit irrelevant sections; do not create an issue, document, or commit unless requested or required by the project.

## Template

```markdown
## Dominant Smell

[What kind of refactor problem is this really? Long function, mixed responsibilities, duplication, side effects, type pressure, module boundary issue, legacy behavior uncertainty, etc.]

## Problem Statement

[The problem being solved, from the developer's perspective. What pain points exist? What would ideal look like?]

## Scope

- In scope: [specific files, modules, functions, seams]
- Out of scope: [related but deferred items]
- Boundary to preserve: [public APIs, contracts, behavior that must remain stable]

## Solution

[The proposed solution. What is the first safe move? Why this approach over alternatives?]

## Safety Net

- Existing tests to rely on: [list]
- Characterization tests to add: [list or n/a]
- Manual verification checkpoints: [list or n/a]
- Compiler, type, lint checks: [list]

## Behavior Surface To Preserve

- Inputs and outputs: [list]
- Side effects: [list]
- Ordering and timing expectations: [list]
- Fallbacks and defaults: [list]
- Errors and edge cases: [list]

## Phases Or Commits

[Detailed implementation plan broken into small, reversible steps. Each step should leave the codebase in a working state.]

1. [Step description]
2. [Step description]
3. [Step description]
...

## Mixed Work Packaging

[If feature work or bug fixes are included, explain how they are separated from the refactor. Otherwise state that this is refactor-only work.]

## Further Notes

- Assumptions: [list]
- Unknowns and risks: [list]
- Follow-up work: [list]
```

For the planning process that produces this template, see [02-planning-workflow.md](02-planning-workflow.md).
