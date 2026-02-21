# Design Patterns for Testable React

- [Deep Modules](#deep-modules)
- [Dependency Injection](#dependency-injection)
- [Return via Callbacks/Output](#return-via-callbacksoutput)
- [Keep Props Small](#keep-props-small)
- [Value Objects](#value-objects)

## Deep Modules

Hide complex implementation behind simple interfaces:

```typescript
// BAD: Shallow module with leaky surface
function Form() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState("");
  // ...20 more lines of logic
}

// GOOD: Deep module with simple interface
function Form({ onSubmit }: { onSubmit: (data: FormData) => Promise<void> }) {
  const { values, errors, isSubmitting, handleSubmit } = useFormSubmit(onSubmit);
  // Complex logic hidden in hook
}
```

Tests interact with simple props/callbacks, not internal complexity.

## Dependency Injection

Accept dependencies via props or context:

```typescript
// BAD: Creates dependency inside
function UserProfile() {
  const user = useFetch("/api/user"); // Hard to mock
}

// GOOD: Accepts dependency
function UserProfile({ fetchUser }: { fetchUser: () => Promise<User> }) {
  const user = useQuery(["user"], fetchUser);
}

// Test
render(<UserProfile fetchUser={() => Promise.resolve(mockUser)} />);
```

## Return via Callbacks/Output

Prefer observable output over hidden side effects:

```typescript
// BAD: Mutates global singleton
function login(credentials) {
  authService.login(credentials);
  currentUser = authService.user; // Global mutation
}

// GOOD: Emits via callback
function LoginForm({ onLogin }: { onLogin: (user: User) => void }) {
  const handleSubmit = async (creds) => {
    const user = await login(creds);
    onLogin(user);
  };
}

// Test
const onLogin = jest.fn();
render(<LoginForm onLogin={onLogin} />);
// Assert onLogin was called with expected user
```

## Keep Props Small

Fewer props = fewer test scenarios:

```typescript
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

## Value Objects

Use domain types to simplify test setup:

```typescript
// Instead of primitives everywhere
expect(charge.amount).toBe(9999);
expect(charge.currency).toBe("USD");

// Use value object
expect(charge.amount).toEqual(Money.fromDollars(99.99, "USD"));
```
