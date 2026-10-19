# Functional Pipeline

Transform data through a chain of pure functions.
Use a pipeline only when clearer than the loop. Preserve mutation/identity contracts and callback ordering; staged array methods and an interleaved loop are not equivalent for stateful callbacks, exceptions, or sparse arrays.

## Core Concepts

### Pure Functions

Same input always produces same output, no side effects.

```ts
// Pure: No side effects, predictable
function double(x: number): number {
  return x * 2
}

// Impure: Depends on external state
let multiplier = 2
function doubleImpure(x: number): number {
  return x * multiplier // What if multiplier changes?
}
```

### Immutability

Prefer local, predictable data flow. Returning new data is useful when callers do not depend on mutation; changing a mutating API is a contract change.

```ts
// Bad: Mutates input
function addItem(cart: CartItem[], item: CartItem): CartItem[] {
  cart.push(item)
  return cart
}

// Good: Returns new array
function addItem(cart: CartItem[], item: CartItem): CartItem[] {
  return [...cart, item]
}
```

## Composition Over Nesting

```ts
// Before: Nested function calls (hard to read)
const result = formatCurrency(
  applyDiscount(
    calculateTotal(
      filterAvailable(items)
    ),
    discountRate
  )
)

// After: Pipeline with intermediate variables
const availableItems = filterAvailable(items)
const total = calculateTotal(availableItems)
const discountedTotal = applyDiscount(total, discountRate)
const formattedTotal = formatCurrency(discountedTotal)

```

Prefer named intermediates over adding a pipeline helper. A helper typed as `(T) => T` cannot compose transformations from arrays to numbers to strings without losing type safety.

## Array Methods as Pipeline

```ts
// Before: Imperative loop
const activeUserEmails: string[] = []
for (const user of users) {
  if (user.isActive) {
    activeUserEmails.push(user.email.toLowerCase())
  }
}

// After: Functional pipeline
const activeUserEmails = users
  .filter(user => user.isActive)
  .map(user => user.email.toLowerCase())
```

### Common Pipeline Operations

```ts
const processed = items
  .filter(item => item.isValid)      // Keep only valid
  .map(item => transform(item))       // Transform each
  .sort((a, b) => a.id - b.id)        // Sort
  .slice(0, 10)                       // Take first 10
  .reduce((acc, item) => acc + item.value, 0) // Aggregate
```

## Avoid Intermediate Variables

When operations are clear, chain them.

```ts
// Before: Unnecessary intermediate
const filtered = users.filter(isActive)
const mapped = filtered.map(getEmail)
const sorted = mapped.sort()

// After: Direct pipeline (if readable)
const emails = users
  .filter(isActive)
  .map(getEmail)
  .sort()

// Keep intermediate variables for complex logic
const activeUsers = users.filter(user => {
  const hasRecentActivity = user.lastLogin > thirtyDaysAgo
  const hasValidSubscription = user.subscription?.isActive
  return hasRecentActivity && hasValidSubscription
})

const emailList = activeUsers.map(user => user.email)
```

## Data Flow Clarity

### Explicit Transformations

```ts
// Clear data transformation pipeline
type RawUser = { name: string; email: string; age: string }
type ValidatedUser = { name: string; email: string; age: number }
type User = ValidatedUser & { id: string }

function parseUser(raw: RawUser): ValidatedUser {
  return {
    name: raw.name.trim(),
    email: raw.email.toLowerCase(),
    age: parseInt(raw.age, 10)
  }
}

function assignId(user: ValidatedUser): User {
  return { ...user, id: generateId() }
}

function createUser(raw: RawUser): User {
  return assignId(parseUser(raw))
}
```

`assignId` is not pure when `generateId` uses randomness or external state; keep that effect explicit.

## Reduce for Aggregation

```ts
// Before: Imperative aggregation
let total = 0
let count = 0
for (const item of items) {
  if (item.category === 'electronics') {
    total += item.price
    count++
  }
}
const average = total / count

// After: Functional reduce
const { total, count } = items
  .filter(item => item.category === 'electronics')
  .reduce(
    (acc, item) => ({
      total: acc.total + item.price,
      count: acc.count + 1
    }),
    { total: 0, count: 0 }
  )
const average = total / count
```

## When NOT to Use Functional Pipeline

### Performance Critical Paths

```ts
// Functional: Creates intermediate arrays
const result = items
  .filter(isValid)
  .map(transform)
  .filter(isNeeded)
  .map(process)

// Imperative: Single pass (more efficient)
const result: ProcessedItem[] = []
for (const item of items) {
  if (!isValid(item)) continue
  const transformed = transform(item)
  if (!isNeeded(transformed)) continue
  result.push(process(transformed))
}
```

### Complex State Dependencies

Keep explicit loops for stateful workflows, early exits, and cancellation when they communicate control flow better. Do not turn retries into an incidental pipeline example; retry classification, replay safety, and exhaustion belong to the [error-handling contract](11-error-handling.md).

## Checklist

- [ ] Side effects and callback ordering remain explicit and stable
- [ ] Mutation and identity behavior preserved at public boundaries
- [ ] Pipeline reads top-to-bottom
- [ ] Each step does one thing
- [ ] Intermediate variables used for clarity, not just to break pipeline
