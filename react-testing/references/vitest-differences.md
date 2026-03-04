# Vitest vs Jest

Vitest is API-compatible with Jest for most cases. The main differences:

## Import Style

```typescript
// Jest: globals available by default
test("works", () => { ... });

// Vitest: import from vitest (or enable globals in config)
import { describe, test, expect, vi } from "vitest";
```

## Mock Functions

| Jest | Vitest |
|------|--------|
| `jest.fn()` | `vi.fn()` |
| `jest.mock("module")` | `vi.mock("module")` |
| `jest.spyOn(obj, "method")` | `vi.spyOn(obj, "method")` |
| `jest.useFakeTimers()` | `vi.useFakeTimers()` |
| `jest.useRealTimers()` | `vi.useRealTimers()` |
| `jest.advanceTimersByTime(ms)` | `vi.advanceTimersByTime(ms)` |
| `jest.requireActual("module")` | `vi.importActual("module")` |

## Module Mocking

```typescript
// Jest: hoisted automatically
jest.mock("../api", () => ({
  fetchUser: jest.fn(),
}));

// Vitest: also hoisted, but use vi.importActual for partial mocks
vi.mock("../api", async () => {
  const actual = await vi.importActual("../api");
  return { ...actual, fetchUser: vi.fn() };
});
```

## Fake Timers

```typescript
// Jest
beforeEach(() => jest.useFakeTimers());
afterEach(() => jest.useRealTimers());

// Vitest
beforeEach(() => vi.useFakeTimers());
afterEach(() => vi.useRealTimers());
```

## Snapshot Testing

Works identically. Vitest stores snapshots in the same `__snapshots__/` directory format.

## Configuration

```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "jsdom",
    globals: true, // enables jest-like globals without imports
    setupFiles: ["./src/test-setup.ts"],
  },
});
```

## Setup File

```typescript
// test-setup.ts — identical for both Jest and Vitest
import "@testing-library/jest-dom";
```

`@testing-library/jest-dom` works with Vitest despite the name.
