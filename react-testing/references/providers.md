# Providers

- [Custom Render](#custom-render)
- [Common Providers](#common-providers)
- [Combining Providers](#combining-providers)

## Custom Render

Create a custom render that includes required providers:

```typescript
// test-utils.tsx
import { render, RenderOptions } from "@testing-library/react";
import { BrowserRouter } from "react-router-dom";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: { queries: { retry: false } },
});

const AllProviders: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <QueryClientProvider client={queryClient}>
    <BrowserRouter>
      {children}
    </BrowserRouter>
  </QueryClientProvider>
);

export function renderWithProviders(
  ui: React.ReactElement,
  options?: Omit<RenderOptions, "wrapper">
) {
  return render(ui, { wrapper: AllProviders, ...options });
}
```

## Common Providers

### React Router

```typescript
<MemoryRouter initialEntries={["/initial-path"]}>
  {children}
</MemoryRouter>
```

### Redux

```typescript
<Provider store={createStore(reducer, preloadedState)}>
  {children}
</Provider>
```

### Theme

```typescript
<ThemeProvider theme="dark">
  {children}
</ThemeProvider>
```

## Combining Providers

For tests that need only specific providers:

```typescript
function renderWithRouter(ui: React.ReactElement) {
  return render(ui, { wrapper: MemoryRouter });
}

function renderWithQueryClient(ui: React.ReactElement) {
  return render(ui, {
    wrapper: ({ children }) => (
      <QueryClientProvider client={new QueryClient()}>
        {children}
      </QueryClientProvider>
    ),
  });
}
```
