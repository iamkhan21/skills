# Side Effects

Isolate side effects for predictable, testable code.

## What Are Side Effects?

A function has side effects if it:

- Modifies external state
- Performs I/O (network, disk, database)
- Depends on external mutable state
- Throws exceptions

```ts
// Pure function: No side effects
function add(a: number, b: number): number {
  return a + b
}

// Side effects: Modifies external state
let total = 0
function addToTotal(value: number): number {
  total += value // Side effect!
  return total
}

// Side effects: I/O
async function fetchUser(id: string): Promise<User> {
  return await api.get(`/users/${id}`) // Side effect!
}

// Side effects: Depends on mutable external state
function getCurrentTime(): Date {
  return new Date() // Side effect! (time changes)
}
```

## Command-Query Separation (CQS)

- **Query**: Returns data, no side effects
- **Command**: Performs action, may have side effects, returns void or status

```ts
// Bad: Mixes query and command
function popItem<T>(stack: T[]): T | undefined {
  return stack.pop() // Returns item AND modifies stack
}

// Good: Separated
function peekItem<T>(stack: T[]): T | undefined {
  return stack[stack.length - 1] // Query: just look
}

function removeLastItem<T>(stack: T[]): void {
  stack.pop() // Command: just modify
}
```

### CQS in Practice

```ts
// Mixed: Query with side effect
async function getUserAndIncrementVisits(id: string): Promise<User> {
  const user = await db.getUser(id)
  await db.incrementVisits(id)
  return user
}

// Separated
async function getUser(id: string): Promise<User> {
  return db.getUser(id) // Query
}

async function incrementVisits(id: string): Promise<void> {
  await db.incrementVisits(id) // Command
}

// Usage
const user = await getUser(id)
await incrementVisits(id)
```

## Functional Core, Imperative Shell

Keep business logic pure, push side effects to the edges.

```ts
// Impure: Side effects mixed with logic
class OrderService {
  async processOrder(items: CartItem[]): Promise<Order> {
    const total = items.reduce((sum, item) => sum + item.price, 0)
    
    // Side effect in the middle
    const discount = await this.fetchDiscount(this.userId)
    
    const finalTotal = total - discount
    
    // Side effect at the end
    await this.saveOrder({ items, total: finalTotal })
    
    return { items, total: finalTotal }
  }
}

// Pure core + impure shell
class OrderService {
  // Pure: Easy to test, predictable
  calculateTotal(items: CartItem[], discount: number): number {
    const subtotal = items.reduce((sum, item) => sum + item.price, 0)
    return subtotal - discount
  }

  // Impure shell: Orchestrates side effects
  async processOrder(items: CartItem[]): Promise<Order> {
    const discount = await this.fetchDiscount(this.userId) // Side effect
    const total = this.calculateTotal(items, discount) // Pure
    const order = { items, total }
    await this.saveOrder(order) // Side effect
    return order
  }
}
```

## Referential Transparency

A function is referentially transparent if you can replace its call with its return value.

```ts
// Referentially transparent
function double(x: number): number {
  return x * 2
}
const result = double(2) + double(2) // 8
const result = 4 + 4 // Same result if we substitute

// NOT referentially transparent
let counter = 0
function increment(): number {
  return ++counter
}
const result = increment() + increment() // 3 (0+1, then 1+1)
// Can't substitute - each call returns different value
```

### Benefits

```ts
// Referentially transparent: Can memoize, parallelize, test easily
function formatCurrency(amount: number, currency: string): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency
  }).format(amount)
}

// Easy to test
expect(formatCurrency(100, 'USD')).toBe('$100.00')

// Can cache results
const cache = new Map<string, string>()
function cachedFormat(amount: number, currency: string): string {
  const key = `${amount}-${currency}`
  if (!cache.has(key)) {
    cache.set(key, formatCurrency(amount, currency))
  }
  return cache.get(key)!
}
```

## Isolating Side Effects

### Dependency Injection

```ts
// Bad: Side effect hidden in function
function sendNotification(user: User, message: string): Promise<void> {
  return emailService.send(user.email, message) // Hidden dependency
}

// Good: Side effect injected
interface EmailSender {
  send(to: string, message: string): Promise<void>
}

function sendNotification(
  user: User,
  message: string,
  emailService: EmailSender
): Promise<void> {
  return emailService.send(user.email, message)
}

// Test with mock
const mockEmailService = {
  send: jest.fn()
}
await sendNotification(user, 'Hello', mockEmailService)
expect(mockEmailService.send).toHaveBeenCalledWith(user.email, 'Hello')
```

### Effect Functions

```ts
// Bad: Side effect mixed with logic
function validateAndSave(user: User): ValidationResult {
  const errors = validate(user)
  if (errors.length === 0) {
    database.save(user) // Side effect
  }
  return { isValid: errors.length === 0, errors }
}

// Good: Return effect description, let caller execute
function validate(user: User): ValidationResult {
  const errors = []
  if (!user.email) errors.push('Email required')
  if (!user.name) errors.push('Name required')
  return { isValid: errors.length === 0, errors }
}

type Effect = () => Promise<void>

function createSaveEffect(user: User): Effect {
  return () => database.save(user)
}

// Usage: Separate validation from side effect
const result = validate(user)
if (result.isValid) {
  const saveEffect = createSaveEffect(user)
  await saveEffect()
}
```

## Checklist

- [ ] Side effects identified and isolated
- [ ] Queries don't modify state
- [ ] Commands clearly marked (return void or status)
- [ ] Pure functions used for business logic
- [ ] Side effects pushed to boundaries
- [ ] Dependencies injected for testing
