# Async Patterns

- [findBy vs waitFor](#findby-vs-waitfor)
- [Common Patterns](#common-patterns)
- [Fake Timers](#fake-timers)

## findBy vs waitFor

| Method | Use When |
|--------|----------|
| `findBy*` / `findAllBy*` | Waiting for element(s) to appear |
| `waitFor` | Waiting for an eventual assertion, such as a callback |

```typescript
// findBy: element appears
expect(await screen.findByText(/loaded/i)).toBeInTheDocument();

// waitFor: one focused condition (retries until the assertion stops throwing)
await waitFor(() => {
  expect(screen.getByRole("status")).toHaveTextContent(/success/i);
});
```

Perform clicks and other side effects outside `waitFor`. Returning `false` does not trigger a retry; the callback must throw. A removal helper requires the element to exist initially; for an already-absent element, assert with `queryBy*` instead.

## Common Patterns

```typescript
// Wait for element to appear
await screen.findByRole("alert");

// Wait for element to disappear
await waitForElementToBeRemoved(screen.getByText(/loading/i));

// Wait for callback
await waitFor(() => expect(mockCallback).toHaveBeenCalled());
```

## Fake Timers

```tsx
import { act, cleanup } from "@testing-library/react";

beforeEach(() => jest.useFakeTimers());
afterEach(() => {
  cleanup(); // cancel component-owned timers before flushing the remainder
  jest.runOnlyPendingTimers();
  jest.useRealTimers();
});

test("shows notification after delay", async () => {
  render(<Toast delay={3000} />);
  
  expect(screen.queryByText(/saved/i)).not.toBeInTheDocument();
  
  await act(async () => {
    jest.advanceTimersByTime(3000);
  });
  
  expect(screen.getByText(/saved/i)).toBeInTheDocument();
});

test("handles async timers", async () => {
  render(<PollingComponent />);
  
  await act(async () => {
    jest.advanceTimersByTime(5000);
  });
});
```

When fake timers and user-event are combined, use `userEvent.setup({ advanceTimers: jest.advanceTimersByTime })` (or the Vitest equivalent). Do not work around timeouts with `delay: null`. Use the runner's async timer APIs when timer callbacks themselves await work; bound advancement for polling instead of running every timer indefinitely.
