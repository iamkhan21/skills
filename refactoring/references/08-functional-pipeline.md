# Functional Pipeline

Transform data through a chain of pure functions.

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

Don't mutate data; return new data.

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

// Alternative: Pipe function
const pipe = <T>(...fns: Array<(arg: T) => T>) => 
  (value: T) => fns.reduce((acc, fn) => fn(acc), value)

const processOrder = pipe(
  filterAvailable,
  calculateTotal,
  (total) => applyDiscount(total, discountRate),
  formatCurrency
)

const result = processOrder(items)
```

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
  return pipe(parseUser, assignId)(raw)
}
```

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

```ts
// Don't force functional when state is complex
let retries = 0
while (retries < MAX_RETRIES) {
  try {
    return await fetchData()
  } catch (error) {
    retries++
    await delay(retries * 1000)
  }
}
```

## Checklist

- [ ] Functions are pure (no side effects)
- [ ] Data is not mutated
- [ ] Pipeline reads top-to-bottom
- [ ] Each step does one thing
- [ ] Intermediate variables used for clarity, not just to break pipeline
