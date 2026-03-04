# Process Heuristics

Guidelines for the refactoring process itself.

## Small Steps

Each step should be atomic and verifiable.

```bash
# Bad: One giant commit
git commit -m "refactor: rewrite user module"

# Good: Series of small commits
git commit -m "refactor: extract validateEmail function"
git commit -m "refactor: move validateEmail to validation module"
git commit -m "test: add unit tests for validateEmail"
git commit -m "refactor: replace inline validation with validateEmail call"
```

### Why Small Steps?

- Easy to identify where bug was introduced
- Simple to revert if needed
- Clear git history
- Forces clear thinking

## One Technique at a Time

Don't mix refactoring techniques in one commit.

```bash
# Bad: Mixing techniques
git commit -m "refactor: rename user to account and extract validation"

# Good: Separate commits
git commit -m "refactor: rename user to account"
git commit -m "refactor: extract validation logic"
```

## No Features, No Bug Fixes

Refactoring should not change behavior.

```ts
// During refactoring, DON'T:
// - Add new features
// - Fix bugs (even obvious ones)
// - Change return values
// - Modify error messages

// DO:
// - Keep exact same behavior
// - Improve code structure only
// - Fix bugs in separate commit/PR
```

### Why Separate?

- Easier code review
- Clear git blame
- Isolates risk
- Cleaner history

## Test Every Change

After each refactoring step:

1. Run relevant tests
2. Verify behavior unchanged
3. Commit if green

```bash
# After each small change
npm test -- --related

# Quick sanity check
npm run build
```

## Transformation Priority Premise (TPP)

When refactoring, prefer simpler transformations:

1. **{} → nil** - Add null/undefined handling
2. **nil → constant** - Replace null with value
3. **constant → constant+** - Make constant more complex
4. **constant → scalar** - Replace constant with variable
5. **statement → statements** - Add statements
6. **unconditional → if** - Split execution path
7. **scalar → array** - Handle multiple items
8. **array → container** - Use collection abstraction
9. **statement → recursion** - Replace iteration
10. **if → while** - Repeat conditionally
11. **expression → function** - Extract computation
12. **variable → assignment** - Replace constant with variable

Apply transformations in order of priority (simpler first).

## Refactor Tests Separately

Don't mix test refactoring with production code refactoring.

```bash
# Commit 1: Refactor production code
git commit -m "refactor: extract UserService from UserController"

# Commit 2: Refactor tests
git commit -m "test: update tests to use UserService directly"

# Commit 3: Improve test structure
git commit -m "test: extract test fixtures for user tests"
```

## When to Stop

Refactoring can become an end in itself. Stop when:

- **Original pain point is resolved** — the problem that motivated the refactor is fixed
- **Scope boundary is reached** — you defined what's in/out at the start; respect it
- **Diminishing returns** — each subsequent change improves less than the last
- **Risk exceeds benefit** — further changes risk introducing behavior changes
- **You're gold-plating** — making code "perfect" beyond what the team needs

If you discover new issues during refactoring, document them as separate tasks rather than expanding scope. A completed refactor that solves the original problem is better than an ambitious one that never lands.

## Feature Flags for Safe Rollout

For refactors that touch production behavior boundaries (even when behavior shouldn't change), consider feature flags to deploy intermediate states safely:

```ts
// Gradually roll out the new code path
if (featureFlags.isEnabled('new-user-service')) {
  return newUserService.getUser(id)
}
return legacyUserService.getUser(id)
```

This lets you ship refactoring work incrementally and roll back instantly if something breaks. Remove the flag once the new path is proven stable.

## Pull Request Guidelines

Small but detailed PRs:

- Title: Clear, specific change
- Description: Why + what + how
- Commits: Logical, atomic steps
- Tests: Prove it works

### PR Template

```markdown
## Why
[Problem being solved]

## What
[Changes made]

## How
[Key technical decisions]

## Testing
[How to verify]
```

## Checklist

- [ ] Each commit is atomic
- [ ] One technique per commit
- [ ] No feature changes mixed in
- [ ] Tests pass after each commit
- [ ] PR description explains rationale
