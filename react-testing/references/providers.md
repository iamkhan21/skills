# Providers

- [Custom Render](#custom-render)
- [Common Providers](#common-providers)
- [Combining Providers](#combining-providers)

## Custom Render

Create a custom render that includes required providers:

```tsx
// test-utils.tsx
import { render, RenderOptions } from "@testing-library/react";
import { MemoryRouter } from "react-router-dom";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
}

export function renderWithProviders(
  ui: React.ReactElement,
  options?: Omit<RenderOptions, "wrapper">
) {
  const queryClient = createTestQueryClient();
  const AllProviders: React.FC<{ children: React.ReactNode }> = ({ children }) => (
    <QueryClientProvider client={queryClient}>
      <MemoryRouter initialEntries={["/initial-path"]}>
        {children}
      </MemoryRouter>
    </QueryClientProvider>
  );
  return render(ui, { wrapper: AllProviders, ...options });
}
```

Include only providers the test needs. Keep the client and store stable for the lifetime of a render, but fresh across tests. Disable query retries unless testing retry behavior, and dispose clients/subscriptions in teardown when they schedule background work. Use the installed router/store APIs rather than introducing providers just for a shared helper.

## Common Providers

### React Router

```tsx
<MemoryRouter initialEntries={["/initial-path"]}>
  {children}
</MemoryRouter>
```

### Redux

```tsx
<Provider store={createStore(reducer, preloadedState)}>
  {children}
</Provider>
```

### Theme

```tsx
<ThemeProvider theme="dark">
  {children}
</ThemeProvider>
```

## Combining Providers

For tests that need only specific providers:

```tsx
function renderWithRouter(ui: React.ReactElement) {
  return render(ui, { wrapper: MemoryRouter });
}

function renderWithQueryClient(ui: React.ReactElement) {
  const queryClient = createTestQueryClient();
  return render(ui, {
    wrapper: ({ children }) => (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    ),
  });
}
```
