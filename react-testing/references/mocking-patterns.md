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

Prefer mocks at boundaries where real behavior is slow, unavailable, or nondeterministic:

- Network requests (APIs, webhooks)
- Email/SMS gateways, payment APIs
- Time (`Date`, timers) and randomness (`Math.random`)
- Browser APIs hard to reproduce in JSDOM (IntersectionObserver, matchMedia)
- Third-party SDKs (analytics, auth, storage)

Keep your components, hooks, utilities, and state management real by default. Mocking your own external-service adapter is still boundary mocking. A narrow internal mock can be appropriate for isolation, but document the integration it excludes; do not mock the behavior the test is meant to verify.

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

beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
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
afterEach(() => jest.restoreAllMocks());

test("fetches user data", async () => {
  const fetchSpy = jest.spyOn(globalThis, "fetch").mockResolvedValueOnce(
    new Response(JSON.stringify({ id: 1, name: "Test" }), {
      status: 200,
      headers: { "Content-Type": "application/json" },
    })
  );
  
  const user = await getUser(1);
  expect(fetchSpy).toHaveBeenCalledWith("/api/users/1");
  expect(user.name).toBe("Test");
});
```

## Browser APIs

### localStorage / sessionStorage

```typescript
// Prefer the DOM environment's implementation, with a valid test origin.
beforeEach(() => {
  localStorage.clear();
  sessionStorage.clear();
});
afterEach(() => {
  localStorage.clear();
  sessionStorage.clear();
});
```

### window.location

```typescript
const originalUrl = window.location.href;
beforeEach(() => window.history.replaceState(null, "", "/checkout"));
afterEach(() => window.history.replaceState(null, "", originalUrl));
```

Use this only for same-origin URL state in a configured DOM environment. Do not delete or replace `window.location`; modern environments can make it non-configurable. For navigation, prefer the real memory router, a navigation adapter, or a browser test.

### IntersectionObserver

```typescript
class MockIntersectionObserver {
  observe = jest.fn();
  unobserve = jest.fn();
  disconnect = jest.fn();
}

Object.defineProperty(window, "IntersectionObserver", {
  configurable: true,
  value: MockIntersectionObserver,
});
```

### matchMedia

```typescript
Object.defineProperty(window, "matchMedia", {
  configurable: true,
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

```tsx
// analytics/adapter.ts
export const analytics = {
  track: (event: string, data?: object) => {
    return window.gtag("event", event, data);
  },
  identify: (userId: string) => {
    return window.gtag("set", { user_id: userId });
  },
};

// component.test.tsx (transformed Jest module setup)
jest.mock("../analytics/adapter", () => ({
  analytics: { track: jest.fn(), identify: jest.fn() },
}));

import { analytics } from "../analytics/adapter";

test("tracks button click", async () => {
  const user = userEvent.setup();
  render(<SignupForm />);
  await user.click(screen.getByRole("button", { name: /sign up/i }));
  
  expect(analytics.track).toHaveBeenCalledWith("signup_clicked", {
    source: "homepage",
  });
});
```

### SDK Contracts

Mock only the SDK methods the app actually calls, using the installed SDK's types and version. Prefer the app's existing adapter rather than inventing a vendor API. Test the adapter separately if its request mapping is important.

## Timers

### Fake Timers

```tsx
const user = userEvent.setup({ advanceTimers: jest.advanceTimersByTime });
```

Enable fake timers before setup. Advance React-triggering timers inside `act`, unmount components, flush remaining timers, and restore real timers. See [async patterns](async-patterns.md) for the complete lifecycle.

## React Context

### Custom Render

```tsx
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

These are defaults, not absolute prohibitions:

| Avoid Mocking | Reason |
|---------------|--------|
| Your components | Test real integration |
| Your hooks | Use `renderHook` with real hook |
| Internal utilities | Test actual behavior |
| State management | Use real store with test state |
| React Router | Use `<MemoryRouter>` wrapper |

Restore globals changed with `Object.defineProperty` using their original property descriptors (or delete a newly added property) in teardown. `restoreAllMocks` does not undo arbitrary global assignments. Observer stubs alone do not exercise visibility behavior; drive observer callbacks or use a real browser when that behavior matters.
