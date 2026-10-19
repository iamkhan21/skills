# Testing Workflow

- [Planning](#planning)
- [TDD Loop](#tdd-loop)
- [Refactoring Tests](#refactoring-tests)
- [Per-Test Checklist](#per-test-checklist)

## Planning

Before writing tests, clarify:

1. **Which behaviors matter?** Happy path, validation, error states, edge cases
2. **Observable outcomes?** Visible text, element presence, navigation, callbacks, network calls
3. **Test environment?** Jest or Vitest, custom render helper, test file location
4. **What stays real?** Identify required providers and external boundaries

Infer these from the request and repository first. Ask only when a missing answer changes what should be tested.

## TDD Loop

When implementing behavior test-first, use small vertical slices:

```
WRONG: Write all tests → Write all code
RIGHT: One test → Minimal code → Repeat
```

Each cycle:

1. **Pick one behavior** — "submits valid form", "shows spinner while loading"
2. **Write one failing test** — Express behavior from user's perspective
3. **Write minimal implementation** — Just enough to pass
4. **Refactor if needed** — Remove actual duplication or clarify intent; extraction is optional

### Example Cycle

```tsx
// 1. Write failing test
test("shows error when email is invalid", async () => {
  const { user } = setup(<LoginForm />);
  await user.type(screen.getByLabelText(/email/i), "not-an-email");
  await user.click(screen.getByRole("button", { name: /submit/i }));
  expect(await screen.findByText(/invalid email/i)).toBeInTheDocument();
});

// 2. Implement validation in the submission handler, not during render.
// 3. Rerun the test, then simplify only if it improves clarity.
```

## Refactoring Tests

After green, refactor test code too:

- Extract repeated setup to helper functions
- Extract common flows to domain-specific actions
- Consolidate similar tests with `test.each`
- Remove redundant assertions

Only refactor while tests are green.
For an existing failure, diagnose it first; do not change intended behavior or weaken assertions just to get green. Adding tests alone does not authorize restructuring production code.

## Per-Test Checklist

```
[ ] Test name describes behavior, not implementation
[ ] Uses public interfaces only: render, screen, props, callbacks
[ ] Would survive internal refactors
[ ] Covers one logical behavior
[ ] Mocks are narrow and do not replace the behavior under test
[ ] Setup is straightforward
```
