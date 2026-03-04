# Testing Suspense & Error Boundaries

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
function BrokenComponent(): JSX.Element {
  throw new Error("Something went wrong");
}

test("renders error fallback when child throws", () => {
  // Suppress React's error logging in test output
  const spy = jest.spyOn(console, "error").mockImplementation(() => {});

  render(
    <ErrorBoundary fallback={<div>Something went wrong</div>}>
      <BrokenComponent />
    </ErrorBoundary>
  );

  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();

  spy.mockRestore();
});
```

### Testing error triggered by user action

```tsx
test("shows error UI when API call fails", async () => {
  const spy = jest.spyOn(console, "error").mockImplementation(() => {});

  // MSW handler that returns an error
  server.use(
    http.get("/api/data", () => {
      return new HttpResponse(null, { status: 500 });
    })
  );

  render(
    <ErrorBoundary fallback={<div>Failed to load</div>}>
      <DataDisplay />
    </ErrorBoundary>
  );

  expect(await screen.findByText(/failed to load/i)).toBeInTheDocument();

  spy.mockRestore();
});
```

### Testing error boundary reset

```tsx
test("recovers after error is resolved", async () => {
  const spy = jest.spyOn(console, "error").mockImplementation(() => {});
  const user = userEvent.setup();

  render(
    <ErrorBoundary
      fallback={({ resetErrorBoundary }) => (
        <div>
          <p>Error occurred</p>
          <button onClick={resetErrorBoundary}>Retry</button>
        </div>
      )}
    >
      <DataDisplay />
    </ErrorBoundary>
  );

  expect(await screen.findByText(/error occurred/i)).toBeInTheDocument();

  // Fix the underlying issue (e.g., restore MSW handler)
  server.resetHandlers();

  await user.click(screen.getByRole("button", { name: /retry/i }));
  expect(await screen.findByText(/data loaded/i)).toBeInTheDocument();

  spy.mockRestore();
});
```

## Combined Patterns

### Suspense + Error Boundary

```tsx
test("handles loading → error → retry flow", async () => {
  const spy = jest.spyOn(console, "error").mockImplementation(() => {});
  const user = userEvent.setup();

  server.use(
    http.get("/api/data", () => new HttpResponse(null, { status: 500 }))
  );

  render(
    <ErrorBoundary fallback={<button>Retry</button>}>
      <Suspense fallback={<div>Loading...</div>}>
        <DataDisplay />
      </Suspense>
    </ErrorBoundary>
  );

  // Loading state
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Error state
  expect(await screen.findByRole("button", { name: /retry/i })).toBeInTheDocument();

  // Fix and retry
  server.resetHandlers();
  await user.click(screen.getByRole("button", { name: /retry/i }));
  expect(await screen.findByText(/data loaded/i)).toBeInTheDocument();

  spy.mockRestore();
});
```

### Why suppress console.error?

React logs errors to the console when error boundaries catch them. This is expected behavior, but it clutters test output. The `jest.spyOn(console, "error").mockImplementation(() => {})` pattern suppresses this noise. Always restore it after the test to avoid masking real errors.
