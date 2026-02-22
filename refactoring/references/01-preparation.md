# Preparation

Before touching any code, prepare the refactoring ground.

## Define Scope and Boundaries

Identify junction points between code to change and everything else.

```ts
// Bad: Vague scope "improve user module"
// Good: Specific boundaries

// Boundary example: User module
// - IN SCOPE: UserService, UserRepository, User types
// - OUT OF SCOPE: AuthService, PaymentService, controllers
// - JUNCTION POINTS: UserService.publicMethods, UserRepository interface
```

### How to Find Boundaries

1. Identify entry points (public APIs, UI components)
2. Trace data flow to find dependencies
3. Mark where changes stop propagating
4. Document what stays unchanged

## Cover with Tests

Tests provide safety net for refactoring. Without them, you're refactoring blindly.

### Test Coverage Checklist

- [ ] Happy path works
- [ ] Edge cases covered (empty, null, max values)
- [ ] Error cases handled
- [ ] Implicit inputs tested (time, random, global state)

### What to Test Before Refactoring

```ts
// Explicit inputs
calculateTotal(items: CartItem[]) // Test various item combinations

// Implicit inputs
getCurrentTime() // Mock Date.now
generateId() // Mock Math.random or UUID
```

### Characterization Tests

When no tests exist, write characterization tests that capture current behavior (even if buggy):

```ts
// Capture WHAT the code does, not what it SHOULD do
it('returns -1 for empty cart (current behavior)', () => {
  expect(calculateTotal([])).toBe(-1) // Document current quirk
})
```

## Stricter Linter/Compiler

Make the compiler help you find issues.

```jsonc
// tsconfig.json - enable strict mode
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true
  }
}
```

```
// .eslintrc - treat warnings as errors
{
  "rules": {
    "no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "error"
  }
}
```

### Incremental Strictness

1. Enable one rule at a time
2. Fix all violations
3. Commit
4. Enable next rule

## Checklist

- [ ] Scope defined with clear boundaries
- [ ] Junction points identified
- [ ] Tests cover main scenarios
- [ ] Edge cases tested
- [ ] Linter/compiler stricter settings enabled
