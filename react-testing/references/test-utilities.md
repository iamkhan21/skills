# Test Utilities

- [Basic Setup Function](#basic-setup-function)
- [Domain-Specific Setup](#domain-specific-setup)
- [When to Extract](#when-to-extract)
- [Combining with Providers](#combining-with-providers)

## Basic Setup Function

Combine `userEvent.setup()` with render for consistent test setup:

```typescript
// test-utils.tsx
function setup(jsx: React.ReactElement) {
  const user = userEvent.setup();
  render(jsx);
  return { user };
}

// Usage
test("submits form", async () => {
  const { user } = setup(<LoginForm />);
  
  await user.type(screen.getByLabelText(/email/i), "test@example.com");
  await user.click(screen.getByRole("button", { name: /sign in/i }));
});
```

Use `screen` for all DOM queries—single source of truth.

## Domain-Specific Setup

Extract repeated interactions into reusable actions:

```typescript
// test-utils.tsx
function setupLoginForm(overrides: Partial<LoginFormProps> = {}) {
  const user = userEvent.setup();
  render(<LoginForm {...overrides} />);
  
  return {
    user,
    fillEmail: (email: string) => user.type(screen.getByLabelText(/email/i), email),
    fillPassword: (password: string) => user.type(screen.getByLabelText(/password/i), password),
    submit: () => user.click(screen.getByRole("button", { name: /sign in/i })),
  };
}

// Usage
test("shows error on invalid credentials", async () => {
  const { fillEmail, fillPassword, submit } = setupLoginForm();
  
  await fillEmail("wrong@example.com");
  await fillPassword("bad-password");
  await submit();
  
  expect(await screen.findByText(/invalid credentials/i)).toBeInTheDocument();
});
```

## When to Extract

| Extract Actions When | Keep Inline When |
|---------------------|------------------|
| Same flow in 3+ tests | Only used once or twice |
| Complex multi-step flow | Simple one-line interactions |
| Tests differ only in data | Tests have unique flows |

## Combining with Providers

```typescript
function setupWithProviders(jsx: React.ReactElement) {
  const user = userEvent.setup();
  render(jsx, { wrapper: AllProviders });
  return { user };
}
```

See [providers.md](providers.md) for custom render patterns.
