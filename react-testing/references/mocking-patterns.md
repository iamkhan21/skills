# Mocking Patterns

- [System Boundaries](#system-boundaries)
- [HTTP Requests](#http-requests)
- [Browser APIs](#browser-apis)
- [Third-Party SDKs](#third-party-sdks)
- [Timers](#timers)
- [React Context](#react-context)
- [Module Mocking](#module-mocking)
- [Don't Mock These](#dont-mock-these)

## System Boundaries

Mock only what you don't control. True system boundaries include:

- Network requests (APIs, webhooks)
- Email/SMS gateways, payment APIs
- Time (`Date`, timers) and randomness (`Math.random`)
- Browser APIs hard to reproduce in JSDOM (IntersectionObserver, matchMedia)
- Third-party SDKs (analytics, auth, storage)

**Never mock inside your own React tree**—your components, hooks, utilities, state management.

## HTTP Requests

### MSW (Recommended)

```typescript
// handlers.ts
import { http, HttpResponse } from "msw";

export const handlers = [
  http.get("/api/users/:id", ({ params }) => {
    return HttpResponse.json({ id: params.id, name: "Test User" });
  }),
  http.post("/api/orders", () => {
    return HttpResponse.json({ orderId: "123" }, { status: 201 });
  }),
];

// setup.ts
import { setupServer } from "msw/node";
import { handlers } from "./handlers";

export const server = setupServer(...handlers);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Override for specific test
test("handles server error", async () => {
  server.use(
    http.get("/api/users/:id", () => {
      return new HttpResponse(null, { status: 500 });
    })
  );
  // test code
});
```

### fetch Mock

```typescript
global.fetch = jest.fn();

beforeEach(() => {
  fetch.mockClear();
});

test("fetches user data", async () => {
  fetch.mockResolvedValueOnce({
    ok: true,
    json: async () => ({ id: 1, name: "Test" }),
  });
  
  const user = await getUser(1);
  expect(fetch).toHaveBeenCalledWith("/api/users/1");
  expect(user.name).toBe("Test");
});
```

## Browser APIs

### localStorage / sessionStorage

```typescript
const localStorageMock = (() => {
  let store: Record<string, string> = {};
  return {
    getItem: (key: string) => store[key] || null,
    setItem: (key: string, value: string) => { store[key] = value; },
    removeItem: (key: string) => { delete store[key]; },
    clear: () => { store = {}; },
  };
})();

Object.defineProperty(window, "localStorage", { value: localStorageMock });
```

### window.location

```typescript
const originalLocation = window.location;

delete (window as any).location;
window.location = { ...originalLocation, href: "" };

afterAll(() => {
  window.location = originalLocation;
});
```

### IntersectionObserver

```typescript
class MockIntersectionObserver {
  observe = jest.fn();
  unobserve = jest.fn();
  disconnect = jest.fn();
}

Object.defineProperty(window, "IntersectionObserver", {
  value: MockIntersectionObserver,
});
```

### matchMedia

```typescript
Object.defineProperty(window, "matchMedia", {
  writable: true,
  value: jest.fn().mockImplementation((query: string) => ({
    matches: false,
    media: query,
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
  })),
});
```

## Third-Party SDKs

### Adapter Pattern

```typescript
// analytics/adapter.ts
export const analytics = {
  track: (event: string, data?: object) => {
    return window.gtag("event", event, data);
  },
  identify: (userId: string) => {
    return window.gtag("set", { user_id: userId });
  },
};

// __mocks__/analytics/adapter.ts
export const analytics = {
  track: jest.fn(),
  identify: jest.fn(),
};

// component.test.ts
jest.mock("../analytics/adapter");

import { analytics } from "../analytics/adapter";

test("tracks button click", async () => {
  render(<SignupForm />);
  await userEvent.click(screen.getByRole("button", { name: /sign up/i }));
  
  expect(analytics.track).toHaveBeenCalledWith("signup_clicked", {
    source: "homepage",
  });
});
```

### Stripe

```typescript
jest.mock("@stripe/stripe-js", () => ({
  loadStripe: jest.fn().mockResolvedValue({
    redirectToCheckout: jest.fn().mockResolvedValue({ error: null }),
  }),
}));
```

## Timers

### Fake Timers

```typescript
beforeEach(() => {
  jest.useFakeTimers();
});

afterEach(() => {
  jest.useRealTimers();
});

test("shows notification after delay", () => {
  render(<Toast message="Saved!" delay={3000} />);
  
  expect(screen.queryByText("Saved!")).not.toBeInTheDocument();
  
  jest.advanceTimersByTime(3000);
  
  expect(screen.getByText("Saved!")).toBeInTheDocument();
});
```

## React Context

### Custom Render

```typescript
// test-utils.tsx
import { render, RenderOptions } from "@testing-library/react";
import { ThemeProvider } from "../theme-context";

const AllProviders: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <ThemeProvider>{children}</ThemeProvider>
);

export const renderWithProviders = (
  ui: React.ReactElement,
  options?: Omit<RenderOptions, "wrapper">
) => render(ui, { wrapper: AllProviders, ...options });

// component.test.tsx
import { renderWithProviders } from "../test-utils";

test("applies theme", () => {
  renderWithProviders(<Button>Click me</Button>);
  expect(screen.getByRole("button")).toBeInTheDocument();
});
```

## Module Mocking

### Partial Mock

```typescript
jest.mock("../utils/api", () => ({
  ...jest.requireActual("../utils/api"),
  fetchUser: jest.fn().mockResolvedValue({ id: 1, name: "Mocked" }),
}));
```

### Mock Implementation Per Test

```typescript
const mockCallback = jest.fn();

test.each([
  ["success", { status: 200 }, true],
  ["failure", { status: 500 }, false],
])("handles %s", async (_, response, expected) => {
  mockCallback.mockResolvedValueOnce(response);
  const result = await doSomething(mockCallback);
  expect(result.success).toBe(expected);
});
```

## Don't Mock These

| Avoid Mocking | Reason |
|---------------|--------|
| Your components | Test real integration |
| Your hooks | Use `renderHook` with real hook |
| Internal utilities | Test actual behavior |
| State management | Use real store with test state |
| React Router | Use `<MemoryRouter>` wrapper |
