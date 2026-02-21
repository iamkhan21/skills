# Anti-Patterns

- [Testing Implementation Details](#testing-implementation-details)
- [Nested describe/beforeEach](#nested-describebeforeeach)
- [Query Mistakes](#query-mistakes)

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
// BAD: queryBy for existence check
expect(screen.queryByRole("button")).toBeInTheDocument(); // Silent failure

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
