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
4. **Which are deep modules?** Push logic into hooks/utilities behind simple interfaces

Ask: "Which component or flow, what are its critical behaviors, and how do you run tests?"

## TDD Loop

Use vertical slices (tracer bullets), not horizontal:

```
WRONG: Write all tests → Write all code
RIGHT: One test → Minimal code → Repeat
```

Each cycle:

1. **Pick one behavior** — "submits valid form", "shows spinner while loading"
2. **Write one failing test** — Express behavior from user's perspective
3. **Write minimal implementation** — Just enough to pass
4. **Refactor** — Extract hooks, utilities, deepen modules

### Example Cycle

```tsx
// 1. Write failing test
test("shows error when email is invalid", async () => {
  const { user } = setup(<LoginForm />);
  await user.type(screen.getByLabelText(/email/i), "not-an-email");
  await user.click(screen.getByRole("button", { name: /submit/i }));
  expect(await screen.findByText(/invalid email/i)).toBeInTheDocument();
});

// 2. Write minimal code
const [error, setError] = useState("");
if (!email.includes("@")) setError("invalid email");

// 3. Refactor (after green)
// Extract email validation to utility
// Extract form state to custom hook
```

## Refactoring Tests

After green, refactor test code too:

- Extract repeated setup to helper functions
- Extract common flows to domain-specific actions
- Consolidate similar tests with `test.each`
- Remove redundant assertions

Only refactor while tests are green.

## Per-Test Checklist

```
[ ] Test name describes behavior, not implementation
[ ] Uses public interfaces only: render, screen, props, callbacks
[ ] Would survive internal refactors
[ ] Covers one logical behavior
[ ] Only mocks system boundaries
[ ] Setup is straightforward
```
