# Async Patterns

- [findBy vs waitFor](#findby-vs-waitfor)
- [Common Patterns](#common-patterns)
- [Fake Timers](#fake-timers)

## findBy vs waitFor

| Method | Use When |
|--------|----------|
| `findBy*` | Waiting for single element to appear |
| `waitFor` | Waiting for condition, multiple assertions |

```typescript
// findBy: element appears
expect(await screen.findByText(/loaded/i)).toBeInTheDocument();

// waitFor: multiple conditions
await waitFor(() => {
  expect(screen.getByRole("status")).toHaveTextContent(/success/i);
  expect(screen.getByRole("button")).toBeEnabled();
});
```

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

```typescript
beforeEach(() => jest.useFakeTimers());
afterEach(() => jest.useRealTimers());

test("shows notification after delay", async () => {
  render(<Toast delay={3000} />);
  
  expect(screen.queryByText(/saved/i)).not.toBeInTheDocument();
  
  jest.advanceTimersByTime(3000);
  
  expect(screen.getByText(/saved/i)).toBeInTheDocument();
});

test("handles async timers", async () => {
  render(<PollingComponent />);
  
  await act(async () => {
    jest.advanceTimersByTime(5000);
  });
});
```
