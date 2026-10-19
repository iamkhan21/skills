# Testing Suspense & Error Boundaries

Use the project's actual boundary component and reset API. `ErrorBoundary` below assumes a wrapper accepting a fallback element; React itself does not export that component. Use a controlled unresolved resource for deterministic loading assertions, not a real network delay.

- [Suspense Fallbacks](#suspense-fallbacks)
- [Error Boundaries](#error-boundaries)
- [Combined Patterns](#combined-patterns)

## Suspense Fallbacks

### Testing loading states

```tsx
test("shows loading fallback while data loads", async () => {
  render(
    <Suspense fallback={<div>Loading...</div>}>
      <UserProfile userId={1} />
    </Suspense>
  );

  // Fallback renders immediately
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Content appears after data loads
  expect(await screen.findByText(/john/i)).toBeInTheDocument();
  expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
});
```

### With lazy-loaded components

```tsx
test("shows fallback for lazy component", async () => {
  const LazyComponent = React.lazy(() => import("./HeavyComponent"));

  render(
    <Suspense fallback={<div>Loading component...</div>}>
      <LazyComponent />
    </Suspense>
  );

  expect(screen.getByText(/loading component/i)).toBeInTheDocument();
  expect(await screen.findByText(/heavy content/i)).toBeInTheDocument();
});
```

## Error Boundaries

### Testing that error fallback renders

```tsx
// A component that throws
function BrokenComponent(): never {
  throw new Error("Something went wrong");
}

test("renders error fallback when child throws", () => {
  render(
    <ErrorBoundary fallback={<div>Something went wrong</div>}>
      <BrokenComponent />
    </ErrorBoundary>
  );

  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();

});
```

### Async failures

Error boundaries catch render-time errors, not ordinary event-handler exceptions or arbitrary rejected promises. A failed HTTP request reaches a boundary only if the data layer or component throws during rendering. Otherwise, test the component's own error state.

## Combined Patterns

For loading → error → retry behavior:

1. Render with the providers and Suspense boundary actually used by the app.
2. Keep a controlled request pending and assert the loading state.
3. Resolve it as an error and await the appropriate error UI.
4. Set up a successful response before clicking Retry.
5. Verify Retry invokes the real boundary/cache reset or refetch mechanism; a button with no handler does not test recovery.
6. Await successful content and verify the fallback disappears.

Resetting MSW handlers only changes future responses; it does not reset a cached rejection or an error boundary. Restore request handlers and dispose per-test caches in teardown.

## Expected Error Logs

Prefer the runner's or React version's supported error-reporting setup. If an expected error needs filtering, match that error narrowly, preserve unrelated console messages, and restore spies in `afterEach` or `finally` so failed assertions cannot leak suppression into other tests.
