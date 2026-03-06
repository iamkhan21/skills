# Preparation

Before touching code, decide what problem you are actually solving and how you will know you did not break behavior.

## Diagnose The Dominant Smell

Do not begin with generic cleanup. Name the main pressure first.

- Long function or deep nesting
- Duplication
- Mixed responsibilities
- Side effects tangled with decision logic
- Complex or misleading types
- Module-boundary or architecture pressure
- Legacy code with weak tests or unclear behavior

If the request is vague, translate it into one concrete smell before planning changes.

## Define Scope And Boundaries

Identify junction points between the code you will touch and the rest of the system.

```ts
// Bad: vague scope
// "improve user module"

// Better: explicit scope and boundaries
// IN SCOPE: UserService, UserRepository, User types
// OUT OF SCOPE: AuthService, PaymentService, controllers
// JUNCTION POINTS: UserService public methods, UserRepository contract
```

### Boundary Checklist

1. Identify entry points: public APIs, exported functions, UI surfaces, CLI commands.
2. Trace important dependencies: data stores, HTTP clients, caches, feature flags.
3. Mark where behavior must remain stable.
4. Write down non-goals so the refactor does not turn into a redesign.

## Name The Behavior Surface

Before changing legacy or low-test code, list the behavior you are trying to preserve.

- Inputs and outputs
- Side effects
- Ordering and timing expectations
- Fallback behavior
- Error cases and messages
- Weird legacy quirks that may actually matter

This turns "be careful" into concrete checks.

## Choose A Proportional Safety Net

Use the best safety net available instead of assuming ideal tests.

### If Good Tests Exist

- Run them before changing structure.
- Extend them around the touched boundary if coverage is thin.

### If Tests Are Weak

- Add characterization tests around the public seam.
- Prefer narrow regression tests over broad test rewrites.

### If Tests Are Missing

- Slow down.
- Make smaller, reversible steps.
- Write a short manual verification list for the behavior surface.
- Treat compiler, type, and lint checks as additional guardrails.

### Characterization Tests

Capture what the code does now, not what you wish it did.

```ts
it('returns -1 for empty cart (current behavior)', () => {
  expect(calculateTotal([])).toBe(-1)
})
```

## Tighten Mechanical Guardrails

Make the compiler and linter help you.

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true
  }
}
```

### Biome

```jsonc
{
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedVariables": "error"
      }
    }
  },
  "formatter": {
    "enabled": true
  }
}
```

### OxLint

```bash
npx oxlint .
```

## Checklist

- [ ] Dominant smell named
- [ ] Scope and non-goals written down
- [ ] Junction points identified
- [ ] Behavior surface listed for risky code
- [ ] Safety net chosen for current reality
- [ ] Compiler, type, and lint checks ready
