# Static Typing

Use types to document contracts and catch errors early.

## Types as Documentation

```ts
// Without types: Must read implementation
function process(data) {
  return data.items.map(i => i.price * i.quantity).reduce((a, b) => a + b, 0)
}

// With types: Clear from signature
interface CartData {
  items: Array<{
    price: number
    quantity: number
  }>
}

function calculateTotal(data: CartData): number {
  return data.items
    .map(item => item.price * item.quantity)
    .reduce((sum, total) => sum + total, 0)
}
```

## Avoid `any`, Use `unknown`

### Why Not `any`

```ts
// any opts out of type checking entirely
function process(data: any) {
  return data.foo.bar.baz // No error, but might crash at runtime
}

const result = process(null) // No error
result.toUpperCase() // No error, but crashes
```

### Use `unknown` Instead

```ts
// unknown requires type checking before use
function process(data: unknown) {
  // data.foo // Error: 'data' is of type 'unknown'
  
  if (typeof data === 'object' && data !== null && 'foo' in data) {
    // Now we can safely access data.foo
    return (data as { foo: string }).foo
  }
  
  throw new Error('Invalid data')
}
```

### Type Guards

```ts
// Custom type guard
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'email' in value &&
    typeof (value as User).id === 'string' &&
    typeof (value as User).email === 'string'
  )
}

function processUser(data: unknown): string {
  if (isUser(data)) {
    return data.email // TypeScript knows data is User
  }
  throw new Error('Invalid user data')
}
```

## Discriminated Unions

Use a common property to distinguish types.

```ts
// Discriminated union
type Result<T, E = Error> =
  | { success: true; value: T }
  | { success: false; error: E }

function divide(a: number, b: number): Result<number> {
  if (b === 0) {
    return { success: false, error: new Error('Division by zero') }
  }
  return { success: true, value: a / b }
}

// TypeScript narrows based on discriminant
const result = divide(10, 2)
if (result.success) {
  console.log(result.value) // TypeScript knows value exists
} else {
  console.error(result.error.message) // TypeScript knows error exists
}
```

### State Machines

```ts
type LoadingState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User }
  | { status: 'error'; error: Error }

function handleState(state: LoadingState): string {
  switch (state.status) {
    case 'idle':
      return 'Click to load'
    case 'loading':
      return 'Loading...'
    case 'success':
      return `Hello, ${state.data.name}` // TypeScript knows data exists
    case 'error':
      return `Error: ${state.error.message}` // TypeScript knows error exists
  }
}
```

## Generic Constraints

```ts
// Constraint: Must have id property
interface WithId {
  id: string
}

function findById<T extends WithId>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id)
}

// Works: User has id
const users: User[] = [{ id: '1', name: 'Alice' }]
const user = findById(users, '1')

// Error: string doesn't have id
// findById(['a', 'b'], 'a') // Type error
```

### Multiple Constraints

```ts
interface Timestamped {
  createdAt: Date
  updatedAt: Date
}

function findByDate<T extends WithId & Timestamped>(
  items: T[],
  date: Date
): T[] {
  return items.filter(item => item.createdAt <= date)
}
```

## Utility Types

### Built-in Utilities

```ts
// Partial: All properties optional
function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates }
}

// Required: All properties required
type CompleteUser = Required<User>

// Pick: Select specific properties
type UserSummary = Pick<User, 'id' | 'name' | 'email'>

// Omit: Exclude specific properties
type UserWithoutPassword = Omit<User, 'password'>

// Record: Object with string keys
type UserMap = Record<string, User>

// NonNullable: Exclude null and undefined
type NonNullableString = NonNullable<string | null | undefined> // string
```

### Custom Utilities

```ts
// Deep partial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P]
}

// Make specific keys required
type RequireKeys<T, K extends keyof T> = T & Required<Pick<T, K>>
type UserWithEmail = RequireKeys<User, 'email'>

// Make specific keys optional
type OptionalKeys<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>
type UserWithoutRequiredEmail = OptionalKeys<User, 'email'>
```

## Type Inference

Let TypeScript infer types when possible.

```ts
// Explicit (redundant)
const numbers: number[] = [1, 2, 3]
const doubled: number[] = numbers.map((n: number): number => n * 2)

// Inferred (cleaner)
const numbers = [1, 2, 3]
const doubled = numbers.map(n => n * 2)
```

### When to Be Explicit

```ts
// Function return types: Document intent
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price, 0)
}

// Complex object shapes: Define interface
interface ApiResponse<T> {
  data: T
  status: number
  message: string
}

async function fetchUser(id: string): Promise<ApiResponse<User>> {
  // ...
}
```

## Strict Mode

Enable all strict checks.

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true
  }
}
```

### noUncheckedIndexedAccess

```ts
const items = ['a', 'b', 'c']

// Without noUncheckedIndexedAccess
const item: string = items[10] // No error, but undefined at runtime

// With noUncheckedIndexedAccess
const item: string | undefined = items[10] // Must handle undefined
```

## Checklist

- [ ] No `any` types (use `unknown` if type unknown)
- [ ] Type guards for runtime validation
- [ ] Discriminated unions for state
- [ ] Constraints on generics
- [ ] Explicit return types for public functions
- [ ] Strict mode enabled
