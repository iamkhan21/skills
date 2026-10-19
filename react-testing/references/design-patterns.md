# Design Choices for Testable React

Use these options only when a test exposes a genuine design problem. Writing tests does not require introducing hooks, adapters, or value objects.

## Deep Modules

- Test through a small public interface: DOM behavior, props, and callbacks.
- Local state is not a design flaw. Extract a hook when it clarifies responsibilities or enables reuse, not merely to hide lines.
- Test a component-local hook through its component unless the hook has an independent public contract.

## Dependency Injection

- Prefer the app's existing network boundary and providers. A component that fetches internally can often be tested with MSW without changing its props.
- Accept a dependency through props, context, or an adapter when it represents a useful production boundary.
- Do not create test-only dependency plumbing throughout the component tree.

## Return via Callbacks/Output

- Assert visible output or existing public callbacks.
- A callback can make ownership explicit, but adding one changes the API; do not redesign an established auth/store flow solely to make a mock assertion easier.
- Fresh stores and controlled providers often make shared-state behavior testable without changing production contracts.

## Keep Props Small

Use types to make impossible combinations unrepresentable where that reflects real product constraints:

```tsx
// BAD: Many props, many combinations
<Button
  isLoading={false}
  isError={false}
  isSuccess={false}
  isDisabled={false}
  variant="primary"
  size="medium"
  icon="check"
/>

// GOOD: Consolidated state
<Button state="loading" variant="primary" />
```

Do not collapse independent states just to reduce test count. An error and a disabled state may legitimately coexist.

## Value Objects

Reuse domain types already present in the app. Introduce a value object only when it enforces useful invariants, not to shorten assertions. Plain fixtures are often sufficient.
