# Refactoring Cheatsheet

Quick reference for common refactoring decisions.

## Preparation

| Step | Action |
|------|--------|
| Name the smell | Decide what pain you are actually reducing |
| Define scope | Identify boundaries, junction points, and non-goals |
| Protect behavior | Choose tests, characterization checks, or manual verification |
| Tighten guardrails | Use type, lint, and build checks as safety rails |

## Process

| Principle | Practice |
|-----------|----------|
| Small steps | Each step leaves code working |
| One concept | Avoid mixing unrelated transformations |
| No hidden behavior changes | Separate features and bug fixes unless explicitly requested |
| Verify often | Run the most relevant checks after meaningful changes |

## First Move By Smell

| Smell | First useful move |
|------|-------------------|
| Long function | Extract one pure helper or add guard clauses |
| Hard to test | Isolate side effects from decision logic |
| Duplication | Classify identical vs similar vs conceptual duplication |
| Unclear names | Rename to reveal intent before introducing structure |
| Type complexity | Simplify contracts before adding generic abstractions |
| Tangled modules | Extract one seam, not a full redesign |
| Vague cleanup | Turn the request into one concrete smell and one non-goal |
| Legacy no-test code | Name the behavior surface and add characterization coverage |

## Code-Level Techniques

### Names

| Problem | Solution |
|---------|----------|
| Too short | Expand with context (`x` -> `userIndex`) |
| Inaccurate | Rename to match behavior |
| Inconsistent | Use same name for same concept |
| Booleans | `is/has/can/should` + condition |

### Duplication

| Type | Solution |
|------|----------|
| Identical | Extract function |
| Similar | Parameterize |
| Conceptual | Create abstraction only after proving a shared concept |

### Conditions

| Problem | Solution |
|---------|----------|
| Deep nesting | Guard clauses (early return) |
| Complex switch | Strategy pattern only if it reduces branching pain |
| Type checking | Polymorphism when the boundary is stable |
| Null checks | Null object pattern when repeated branches add noise |

### Functional Pipeline

```ts
items
  .filter(predicate)
  .map(transform)
  .reduce(aggregate, initial)

// Principles
// - Pure functions (same input -> same output)
// - Immutability (return new data)
// - Composition over nesting
// - Use pipelines only when they make the flow clearer
```

## Design-Level Techniques

### Abstraction

| Rule | Description |
|------|-------------|
| Level matching | Hide how, show what |
| No leaks | Do not expose internals |
| Rule of three | Wait for repeated pressure before abstracting |

### Side Effects

| Principle | Practice |
|-----------|----------|
| CQS | Query returns data, command returns void |
| Isolation | Functional core, imperative shell |
| Dependency injection | Pass dependencies, do not reach for them |

### Error Handling

| Error type | Strategy |
|------------|----------|
| Expected | Return Result or Either type |
| Unexpected | Throw, catch at boundary |
| Transient | Retry with backoff |

## Real-World Cases

| Situation | Guidance |
|-----------|----------|
| Weak tests | Add narrow characterization tests around the risky seam |
| No tests | Use smaller moves plus explicit manual verification |
| Mixed refactor + feature | Split into phases or clearly separate review sections |
| Suspected bug during refactor | Record it and fix separately unless requested |
| Architecture pressure | Start with the smallest seam that lowers coupling |

## Architecture-Level Techniques

### Coupling vs Cohesion

| Metric | Goal |
|--------|------|
| Coupling | Low (modules independent) |
| Cohesion | High (related code together) |

### Dependency Direction

```text
Presentation -> Application -> Domain <- Infrastructure
                (depends on abstractions)
```

### Layers

| Layer | Responsibility |
|-------|---------------|
| Presentation | HTTP, CLI, UI |
| Application | Use cases, orchestration |
| Domain | Business rules, entities |
| Infrastructure | DB, APIs, external services |

## Quick Reference Commands

### Before Refactoring

```bash
npm test
npm run test:coverage
npx @biomejs/biome check .
```

### During Refactoring

```bash
npm test -- --related
npx @biomejs/biome check --write .
npm run typecheck
```

### Commit Message Format

```text
<type>: <description>

refactor: extract validation to helper
refactor: rename user to account for clarity
refactor: reduce nesting with guard clauses
chore: remove dead code in payment module
style: apply Biome formatting
test: add characterization tests for UserService
```

## Technique Decision Tree

```text
Is the code working correctly?
|- No -> Decide whether the user asked for a bug fix; if not, record it separately
`- Yes
   |- Are there tests?
   |  |- No -> Add characterization tests or define manual verification
   |  `- Yes
   |     `- Is scope clear?
   |        |- No -> Define boundaries and non-goals
   |        `- Yes -> Start refactoring
   |           |- Take the smallest safe first move
   |           |- Verify behavior after meaningful steps
   |           |- Stop when the main pain point is reduced
   |           `- Defer extra cleanup instead of drifting scope
```

## Red Flags

| Sign | Likely Issue |
|------|--------------|
| Can't name it well | Unclear responsibility |
| Long function | Multiple responsibilities |
| Many parameters | Hidden dependencies |
| Deep nesting | Complex control flow |
| Comments explaining WHAT | Code not self-documenting |
| Tests need mocks everywhere | Tight coupling |
| New interfaces appear immediately | Premature abstraction |
| Refactor plan includes feature changes | Scope drift |

## Checklist Before Committing

- [ ] Tests pass
- [ ] No new warnings
- [ ] Code formatted
- [ ] One conceptual transformation applied
- [ ] Feature work is separate or explicitly labeled
- [ ] Commit message clear
