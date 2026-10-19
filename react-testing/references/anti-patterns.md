# Anti-Patterns & Code Smells

- [Testing Implementation Details](#testing-implementation-details)
- [Nested describe/beforeEach](#nested-describebeforeeach)
- [Query Mistakes](#query-mistakes)
- [Code Smells as Design Feedback](#code-smells-as-design-feedback)
- [Refactoring Triggers](#refactoring-triggers)
- [Coverage Guidance](#coverage-guidance)

## Testing Implementation Details

### Internal State

```typescript
// BAD: Tests internal state
expect(component.instance().state.isOpen).toBe(true);

// GOOD: Tests observable behavior
expect(screen.getByRole("dialog")).toBeVisible();
```

### Component Names

```typescript
// BAD: Queries by component name
wrapper.find("SubmitButton").simulate("click");

// GOOD: Queries by accessible name
await user.click(screen.getByRole("button", { name: /submit/i }));
```

### Private Methods

```typescript
// BAD: Tests private/internal function
expect(form.validateEmail("bad")).toBe(false);

// GOOD: Tests through public interface
await user.type(screen.getByLabelText(/email/i), "bad");
await user.click(screen.getByRole("button", { name: /submit/i }));
expect(screen.getByText(/invalid email/i)).toBeInTheDocument();
```

## Nested describe/beforeEach

```typescript
// BAD: Mutable shared state, hard to trace
describe("checkout", () => {
  let cart;
  beforeEach(() => { cart = createCart(); });

  describe("with items", () => {
    beforeEach(() => { cart.add(item); });
    test("calculates total", () => { /* which cart state? */ });
  });
});

// GOOD: Inline setup, clear and isolated
test("calculates total for items", () => {
  const cart = createCart([item1, item2]);
  expect(checkout(cart).total).toBe(item1.price + item2.price);
});
```

## Query Mistakes

### Wrong Query Type

```typescript
// Less helpful diagnostics: this assertion still fails if no button exists
expect(screen.queryByRole("button")).toBeInTheDocument();

// GOOD: getBy fails fast
expect(screen.getByRole("button")).toBeInTheDocument();

// BAD: getBy for absence check
expect(screen.getByRole("alert")).not.toBeInTheDocument(); // Throws first

// GOOD: queryBy for absence
expect(screen.queryByRole("alert")).not.toBeInTheDocument();
```

### Direct DOM Access

```typescript
// BAD: Query selector
container.querySelector(".submit-button");

// GOOD: Accessible query
screen.getByRole("button", { name: /submit/i });
```

## Code Smells as Design Feedback

Use test struggles as clues, not proof that production code needs restructuring. Check the test setup and requirements first:

| Smell | Meaning |
|-------|---------|
| Many tests per module | May reflect legitimate behavior; look for unrelated responsibilities |
| Complex test setup | Check helper design, required providers, and boundary coupling |
| Need to mock your own code | Investigate the boundary and what integration the mock excludes |
| Test passes after breaking the target behavior | Strengthen the assertion or remove genuinely redundant coverage |
| Testing private methods | Test public behavior instead |
| Test name mentions implementation | Rename to describe behavior |
| Fragile tests (break on refactor) | Testing implementation details |
| Slow test suite | Measure first: real delays, network calls, render cost, and setup overhead |

## Refactoring Triggers

When tests signal design issues:

1. **Hard to set up** → Simplify setup; consider a dependency seam only if coupling is the cause
2. **Need internal mocks** → Check whether a real boundary can be exercised or isolated
3. **Multiple concerns in one test** → Focus the test before splitting production components
4. **Testing private state** → Assert observable outcomes; public loading/error state may still be part of the contract

## Coverage Guidance

- Follow the project's coverage requirements; there is no universally sufficient percentage.
- Prioritize critical paths, failure states, and regressions over percentage gains.
- Inspect uncovered risky behavior; high coverage alone does not prove useful assertions.
