# Refactoring Cheatsheet

Quick reference for common refactoring techniques.

## Preparation

| Step | Action |
|------|--------|
| Define scope | Identify boundaries and junction points |
| Add tests | Cover edge cases, happy path, error cases |
| Stricten linter | Enable strict mode, fix warnings as errors |

## Process

| Principle | Practice |
|-----------|----------|
| Small steps | Each commit leaves code working |
| One technique | Don't mix different refactorings |
| No features | Only structure changes, no behavior changes |
| Test always | Run tests after each change |

## Code-Level Techniques

### Names

| Problem | Solution |
|---------|----------|
| Too short | Expand with context (`x` → `userIndex`) |
| Inaccurate | Rename to match behavior |
| Inconsistent | Use same name for same concept |
| Booleans | `is/has/can/should` + condition |

### Duplication

| Type | Solution |
|------|----------|
| Identical | Extract function |
| Similar | Parameterize |
| Conceptual | Create abstraction |

### Conditions

| Problem | Solution |
|---------|----------|
| Deep nesting | Guard clauses (early return) |
| Complex switch | Strategy pattern |
| Type checking | Polymorphism |
| Null checks | Null object pattern |

### Functional Pipeline

```
// Pattern
items
  .filter(predicate)
  .map(transform)
  .reduce(aggregate, initial)

// Principles
- Pure functions (same input → same output)
- Immutability (return new data)
- Composition over nesting
```

## Design-Level Techniques

### Abstraction

| Rule | Description |
|------|-------------|
| Level matching | Hide how, show what |
| No leaks | Don't expose internals |
| Rule of three | Wait for 3 instances before abstracting |

### Side Effects

| Principle | Practice |
|-----------|----------|
| CQS | Query returns data, command returns void |
| Isolation | Functional core, imperative shell |
| Dependency injection | Pass dependencies, don't reach for them |

### Error Handling

| Error type | Strategy |
|------------|----------|
| Expected | Return Result/Either type |
| Unexpected | Throw, catch at boundary |
| Transient | Retry with backoff |

## Architecture-Level Techniques

### Coupling vs Cohesion

| Metric | Goal |
|--------|------|
| Coupling | Low (modules independent) |
| Cohesion | High (related code together) |

### Dependency Direction

```
Presentation → Application → Domain ← Infrastructure
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
# Run tests
npm test

# Check coverage
npm run test:coverage

# Run linter (Biome)
npx @biomejs/biome check .
```

### During Refactoring

```bash
# After each change
npm test -- --related

# Format and lint
npx @biomejs/biome check --write .

# Type check
npm run typecheck
```

### Commit Message Format (Conventional Commits)

```
<type>: <description>

# Common types for refactoring
refactor: extract validation to separate function
refactor: rename user to account for clarity
refactor: reduce nesting with guard clauses
chore: remove dead code in payment module
style: apply Biome formatting
test: add characterization tests for UserService
```

## Technique Decision Tree

```
Is the code working correctly?
├── No → Fix bugs first (separate PR)
└── Yes
    ├── Are there tests?
    │   ├── No → Add characterization tests
    │   └── Yes
    │       └── Is scope clear?
    │           ├── No → Define boundaries
    │           └── Yes → Start refactoring
    │               ├── Quick wins first (format, lint, language features)
    │               ├── Then code-level (names, duplication, conditions)
    │               ├── Then design-level (abstraction, side effects)
    │               └── Finally architecture (modules, layers)
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

## Checklist Before Committing

- [ ] Tests pass
- [ ] No new warnings
- [ ] Code formatted
- [ ] One technique applied
- [ ] Commit message clear
