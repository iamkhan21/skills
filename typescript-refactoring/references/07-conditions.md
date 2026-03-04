# Conditions

Reduce conditional complexity for clearer code.

## Guard Clauses

Return early to reduce nesting.

```ts
// Before: Nested conditionals
function processPayment(amount: number, user: User) {
  if (user) {
    if (user.isActive) {
      if (amount > 0) {
        if (user.balance >= amount) {
          user.balance -= amount
          return { success: true }
        } else {
          return { success: false, error: 'Insufficient funds' }
        }
      } else {
        return { success: false, error: 'Invalid amount' }
      }
    } else {
      return { success: false, error: 'User inactive' }
    }
  } else {
    return { success: false, error: 'User not found' }
  }
}

// After: Guard clauses
function processPayment(amount: number, user: User) {
  if (!user) return { success: false, error: 'User not found' }
  if (!user.isActive) return { success: false, error: 'User inactive' }
  if (amount <= 0) return { success: false, error: 'Invalid amount' }
  if (user.balance < amount) return { success: false, error: 'Insufficient funds' }

  user.balance -= amount
  return { success: true }
}
```

## Ternary for Simple Conditions

Use ternary when both branches are simple expressions.

```ts
// Before: Verbose if-else for simple assignment
let discount
if (user.isPremium) {
  discount = 0.2
} else {
  discount = 0
}

// After: Ternary
const discount = user.isPremium ? 0.2 : 0
```

### When NOT to Use Ternary

```ts
// Bad: Complex logic in ternary
const result = user
  ? user.isActive
    ? user.balance > 0
      ? 'active-with-balance'
      : 'active-no-balance'
    : 'inactive'
  : 'not-found'

// Good: Guard clauses
function getUserStatus(user?: User): string {
  if (!user) return 'not-found'
  if (!user.isActive) return 'inactive'
  return user.balance > 0 ? 'active-with-balance' : 'active-no-balance'
}
```

## Replace Conditionals with Strategy Pattern

For complex conditional logic, use polymorphism.

```ts
// Before: Complex switch/if-else
function calculateShipping(order: Order): number {
  switch (order.shippingMethod) {
    case 'standard':
      return order.weight * 0.5 + 5
    case 'express':
      return order.weight * 0.8 + 10
    case 'overnight':
      return order.weight * 1.2 + 25
    case 'international':
      return order.weight * 2 + 15 + order.distance * 0.1
    default:
      throw new Error(`Unknown shipping method: ${order.shippingMethod}`)
  }
}

// After: Strategy pattern
interface ShippingStrategy {
  calculate(order: Order): number
}

class StandardShipping implements ShippingStrategy {
  calculate(order: Order): number {
    return order.weight * 0.5 + 5
  }
}

class ExpressShipping implements ShippingStrategy {
  calculate(order: Order): number {
    return order.weight * 0.8 + 10
  }
}

class OvernightShipping implements ShippingStrategy {
  calculate(order: Order): number {
    return order.weight * 1.2 + 25
  }
}

class InternationalShipping implements ShippingStrategy {
  calculate(order: Order): number {
    return order.weight * 2 + 15 + order.distance * 0.1
  }
}

const shippingStrategies: Record<string, ShippingStrategy> = {
  standard: new StandardShipping(),
  express: new ExpressShipping(),
  overnight: new OvernightShipping(),
  international: new InternationalShipping(),
}

function calculateShipping(order: Order): number {
  const strategy = shippingStrategies[order.shippingMethod]
  if (!strategy) throw new Error(`Unknown shipping method: ${order.shippingMethod}`)
  return strategy.calculate(order)
}
```

## Replace Conditionals with Polymorphism

When conditionals check object type, use polymorphism instead.

```ts
// Before: Type checking
function getArea(shape: Shape): number {
  if (shape.type === 'circle') {
    return Math.PI * shape.radius ** 2
  } else if (shape.type === 'rectangle') {
    return shape.width * shape.height
  } else if (shape.type === 'triangle') {
    return (shape.base * shape.height) / 2
  }
  throw new Error('Unknown shape')
}

// After: Polymorphism
interface Shape {
  getArea(): number
}

class Circle implements Shape {
  constructor(private radius: number) {}
  getArea(): number {
    return Math.PI * this.radius ** 2
  }
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  getArea(): number {
    return this.width * this.height
  }
}

class Triangle implements Shape {
  constructor(private base: number, private height: number) {}
  getArea(): number {
    return (this.base * this.height) / 2
  }
}

// Usage
function getArea(shape: Shape): number {
  return shape.getArea()
}
```

## Simplify Boolean Conditions

```ts
// Before: Complex boolean logic
if (user && user.isActive && (user.role === 'admin' || user.role === 'moderator')) {
  // ...
}

// After: Extract to function
function canModerate(user?: User): boolean {
  return !!user && user.isActive && ['admin', 'moderator'].includes(user.role)
}

if (canModerate(user)) {
  // ...
}
```

## Null Object Pattern

Replace null checks with null object.

```ts
// Before: Null checks everywhere
function sendNotification(user: User | null) {
  if (user) {
    if (user.email) {
      emailService.send(user.email, 'Welcome!')
    }
  }
}

// After: Null object
interface Notifiable {
  notify(message: string): void
}

class User implements Notifiable {
  constructor(private email: string) {}
  notify(message: string): void {
    emailService.send(this.email, message)
  }
}

class NullUser implements Notifiable {
  notify(_message: string): void {
    // Do nothing
  }
}

function sendNotification(user: Notifiable) {
  user.notify('Welcome!')
}
```

## Checklist

- [ ] Guard clauses used for early returns
- [ ] Ternary for simple conditions only
- [ ] Complex conditionals replaced with strategy/pattern
- [ ] Boolean logic extracted to named functions
- [ ] Null object pattern considered for null handling
