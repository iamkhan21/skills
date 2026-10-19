# Vitest vs Jest

Many assertion and mock APIs are similar, but setup, module mocking, and environments are not interchangeable. Follow the installed runner's version and module system.

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
// Jest in a supported transformed CommonJS setup; native ESM differs
jest.mock("../api", () => ({
  fetchUser: jest.fn(),
}));

// Vitest: hoisted; use an async import for partial mocks
vi.mock("../api", async () => {
  const actual = await vi.importActual<typeof import("../api")>("../api");
  return { ...actual, fetchUser: vi.fn() };
});
```

`vi.importActual` is asynchronous, unlike `jest.requireActual`. Hoisted factories cannot freely capture runtime variables; use the runner's supported hoisting helpers where needed. Jest native ESM uses a different mocking flow; do not mechanically rename APIs.

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
// Jest setupFilesAfterEnv (default globals)
import "@testing-library/jest-dom";

// Vitest setupFiles
import "@testing-library/jest-dom/vitest";

// With Vitest globals disabled, register cleanup explicitly:
import { afterEach } from "vitest";
import { cleanup } from "@testing-library/react";
afterEach(cleanup);
```

Choose the matching setup block, not both matcher entry points. Include the TypeScript setup file in the test tsconfig. If a `types` array exists, extend it without dropping environment types.
