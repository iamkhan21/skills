---
name: react-testing
description: React testing with Testing Library. Use when writing tests for React components, hooks, or user interactions; debugging failing tests; refactoring test files; or questions about testing approach. Triggers for: (1) *.test.tsx, *.spec.ts files, (2) render, screen, userEvent usage, (3) Testing Library questions.
---

# React Testing

## Quick Start

```typescript
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

test("submits form with user input", async () => {
  const user = userEvent.setup();
  render(<LoginForm />);
  
  await user.type(screen.getByLabelText(/email/i), "test@example.com");
  await user.click(screen.getByRole("button", { name: /sign in/i }));
  
  expect(await screen.findByText(/welcome/i)).toBeInTheDocument();
});
```

## Core Principle

**Test behavior, not implementation.** Tests verify observable behavior through public interfaces (DOM, props, callbacks), not internal state or private helpers. A refactor that preserves behavior should never break tests.

## Queries

Prefer queries by user perception: `getByRole` → `getByLabelText` → `getByText` → `getByTestId` (last resort).

See [references/query-cheatsheet.md](references/query-cheatsheet.md).

## User Interactions

Use `userEvent.setup()` for realistic interactions. Prefer over `fireEvent`.

See [references/user-interactions.md](references/user-interactions.md).

## Test Utilities

Reduce boilerplate with setup functions that return `user` and domain-specific actions. Use `screen` for all DOM queries.

See [references/test-utilities.md](references/test-utilities.md).

## Mocking

Mock only at system boundaries: network, time, randomness, browser APIs. Never mock your own components, hooks, or internal utilities.

See [references/mocking-patterns.md](references/mocking-patterns.md).

## Providers

Use custom render helper for required providers (router, store, theme).

See [references/providers.md](references/providers.md).

## Testing Hooks

```typescript
const { result } = renderHook(() => useCounter(0));
act(() => result.current.increment());
expect(result.current.count).toBe(1);
```

See [references/hooks-testing.md](references/hooks-testing.md).

## Test Design

- One behavior per test
- Test name describes behavior, not implementation
- Prefer integration-style tests over shallow unit tests
- ~70% coverage sufficient

See [references/design-patterns.md](references/design-patterns.md) and [references/workflow.md](references/workflow.md).
