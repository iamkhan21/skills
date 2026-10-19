---
name: react-testing
description: Write, debug, or refactor React component and hook tests with React Testing Library, Jest or Vitest, user-event, and MSW. Use for test setup, queries, async UI, providers, API mocking, flaky tests, or questions about testing a React interaction. Not for non-React tests or visual/layout verification that needs a real browser.
---

# React Testing

## Quick Start

Inspect the installed runner, React and Testing Library versions, setup file, and nearby tests first. Reuse existing helpers. Examples assume runner APIs and jest-dom matchers are configured; see [runner setup](references/vitest-differences.md).

```tsx
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

**Test behavior, not implementation.** Verify observable behavior through public interfaces (DOM, props, callbacks), not private state. Prefer tests that survive internal refactors.

## Workflow

1. Identify the behavior and failure mode from the request and code; ask only if a missing requirement affects the test.
2. Reproduce a reported bug with a focused test when practical. Test intended behavior, not an implementation accident.
3. Use real components and necessary providers, await interactions and outcomes, and isolate each test's state.
4. Run focused tests, then relevant neighboring tests and type/lint checks. Report what ran and what remains unverified.

For more detail, see [workflow](references/workflow.md). Keep production changes within the user's scope.

## Framework Note

Examples use Jest syntax. For Vitest projects, the APIs are nearly identical — see [references/vitest-differences.md](references/vitest-differences.md) for the key differences (`vi.fn()` vs `jest.fn()`, module mocking, etc.). `@testing-library/jest-dom` works with both.

## Queries

Prefer queries by user perception: `getByRole` → `getByLabelText` → `getByText` → `getByTestId` (last resort).

See [references/query-cheatsheet.md](references/query-cheatsheet.md).

## User Interactions

Use `userEvent.setup()` for realistic interactions. Prefer over `fireEvent`.

See [references/user-interactions.md](references/user-interactions.md).

## Async Patterns

Use `findBy*` for an element appearing, `findAllBy*` for several, and `waitFor` for an eventual condition such as a callback. Keep interactions outside retry callbacks; prefer one focused assertion per `waitFor`. Use disappearance checks for loading UI rather than arbitrary sleeps.

See [references/async-patterns.md](references/async-patterns.md).

## Accessibility Matchers

Beyond `toBeInTheDocument()`, use `jest-dom` matchers that verify accessible state: `toBeVisible()`, `toBeEnabled()`, `toHaveAccessibleName()`, `toBeInvalid()`. These catch real usability issues, not just DOM presence.

See [references/accessibility-matchers.md](references/accessibility-matchers.md).

## Test Utilities

Extract repeated setup when it helps readability. Prefer `screen` for document queries and `within` for scoped queries; use render-bound queries for a custom container.

See [references/test-utilities.md](references/test-utilities.md).

## Mocking

Prefer mocks at boundaries: network, time, randomness, browser APIs, and external SDK adapters. Keep internal components and hooks real by default. A narrow internal mock can be justified for expensive or unavailable dependencies; state what integration it excludes and cover that separately when important.

See [references/mocking-patterns.md](references/mocking-patterns.md).

## Providers

Use custom render helper for required providers (router, store, theme). Create a fresh QueryClient per test to avoid cache leaks.

See [references/providers.md](references/providers.md).

## Testing Hooks

```typescript
const { result } = renderHook(() => useCounter(0));
act(() => result.current.increment());
expect(result.current.count).toBe(1);
```

See [references/hooks-testing.md](references/hooks-testing.md).

## Suspense & Error Boundaries

Test loading fallbacks by rendering inside `<Suspense>` and asserting the fallback appears, then awaiting the resolved content. Test error boundaries by rendering a component that throws and verifying the fallback UI.

See [references/suspense-error-boundaries.md](references/suspense-error-boundaries.md).

## Anti-Patterns & Code Smells

Common mistakes: testing implementation details, nested describe/beforeEach with shared mutable state, wrong query types. Test struggles are design feedback — if tests are hard to write, the component may need refactoring.

See [references/anti-patterns.md](references/anti-patterns.md).

## Troubleshooting

Covers `act()` warnings, tests that pass alone but fail together, JSDOM limitations, query errors, and async pitfalls.

See [references/troubleshooting.md](references/troubleshooting.md).

## Environment Guardrails

- Check compatibility against installed versions rather than assuming runner APIs or React behavior are identical across versions.
- RTL helpers handle `act` for normal rendering and interactions. Await async work; use explicit `act` for direct hook updates or manual timer advancement, not to hide warnings.
- DOM simulators do not prove layout, browser navigation, or concurrent scheduling. Use real-browser tests for those requirements.
- Restore spies, timers, and global changes; ensure DOM cleanup is configured, especially with Vitest globals disabled.

## Test Design

- One behavior per test
- Test name describes behavior, not implementation
- Prefer integration-style tests over shallow unit tests
- Prioritize critical paths, failure states, and regressions; honor the project's coverage policy rather than inventing a percentage target.

See [references/design-patterns.md](references/design-patterns.md) and [references/workflow.md](references/workflow.md).
