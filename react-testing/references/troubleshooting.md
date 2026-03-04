# Troubleshooting

- [act() Warnings](#act-warnings)
- [Tests Pass Alone but Fail Together](#tests-pass-alone-but-fail-together)
- [JSDOM Limitations](#jsdom-limitations)
- [Common Query Errors](#common-query-errors)
- [Async Pitfalls](#async-pitfalls)

## act() Warnings

The "not wrapped in act(...)" warning means a state update happened outside React's awareness. Common causes and fixes:

### Async state update after test ends

```typescript
// WARNING: Component fetches data after test finishes
test("renders user", () => {
  render(<UserProfile />);
  expect(screen.getByText(/loading/i)).toBeInTheDocument();
  // Test ends, but fetch resolves and calls setState → warning
});

// FIX: Wait for the async operation to complete
test("renders user", async () => {
  render(<UserProfile />);
  expect(await screen.findByText(/john/i)).toBeInTheDocument();
});
```

### Timer-based updates

```typescript
// WARNING: setTimeout fires after test
// FIX: Use fake timers and advance them explicitly
beforeEach(() => jest.useFakeTimers());
afterEach(() => jest.useRealTimers());

test("shows toast after delay", () => {
  render(<Toast delay={1000} />);
  jest.advanceTimersByTime(1000);
  expect(screen.getByText(/saved/i)).toBeInTheDocument();
});
```

### Key insight

Don't blindly wrap things in `act()`. In most cases, the warning is telling you to properly await an async operation. `findBy*` queries and `waitFor` already wrap in `act` internally. If you're using those and still getting warnings, the real issue is usually an unhandled promise or timer.

## Tests Pass Alone but Fail Together

Almost always caused by shared mutable state between tests:

1. **Mock not cleaned up** — Add `afterEach(() => jest.restoreAllMocks())` or use `jest.clearAllMocks()`
2. **Shared QueryClient** — Create a new `QueryClient` per test, not at module level
3. **MSW handlers not reset** — Use `afterEach(() => server.resetHandlers())`
4. **Global state mutation** — Zustand/Redux stores persist between tests; reset in `afterEach`
5. **DOM not cleaned up** — RTL's `cleanup` runs automatically with Jest/Vitest; if using a custom setup, call it manually

## JSDOM Limitations

JSDOM doesn't implement layout. These will not work:

| Missing | Workaround |
|---------|------------|
| `offsetHeight`, `offsetWidth` | Mock via `Object.defineProperty` |
| `getBoundingClientRect()` | Mock to return fixed values |
| `IntersectionObserver` | Use mock class (see mocking-patterns.md) |
| `matchMedia` | Mock (see mocking-patterns.md) |
| `window.scrollTo` | `window.scrollTo = jest.fn()` |
| CSS transitions/animations | Won't fire `transitionend` — dispatch manually |
| `<canvas>` rendering | Use `jest-canvas-mock` |

If your component relies heavily on layout, consider E2E tests (Playwright/Cypress) for those specific behaviors.

## Common Query Errors

### "Unable to find an element with the role..."

1. Check the element actually renders — use `screen.debug()` to inspect the DOM
2. Check the role is correct — use `screen.logTestingPlaygroundURL()` for suggestions
3. Check `name` option uses regex: `{ name: /submit/i }` not `{ name: "Submit" }` (exact match may miss)
4. Element may be hidden — `getByRole` ignores hidden elements by default; use `{ hidden: true }` if needed

### "Found multiple elements with..."

Scope your query with `within`:

```typescript
const dialog = within(screen.getByRole("dialog"));
dialog.getByRole("button", { name: /confirm/i });
```

## Async Pitfalls

### Forgetting to await

```typescript
// BUG: This assertion runs before the click completes
user.click(button); // missing await
expect(screen.getByText(/done/i)).toBeInTheDocument();

// FIX
await user.click(button);
```

### Using waitFor incorrectly

```typescript
// BAD: Side effects inside waitFor (runs multiple times)
await waitFor(async () => {
  await user.click(button); // click fires on every retry!
  expect(screen.getByText(/done/i)).toBeInTheDocument();
});

// GOOD: Side effects outside, only assertions inside
await user.click(button);
await waitFor(() => {
  expect(screen.getByText(/done/i)).toBeInTheDocument();
});
```
