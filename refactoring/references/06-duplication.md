# Duplication

DRY (Don't Repeat Yourself) - but apply with nuance.

## Types of Duplication

### Identical Duplication

Exact same code in multiple places.

```ts
// Duplicated
function validateEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}

function isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}

// Extracted
function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return emailRegex.test(email)
}
```

### Similar Duplication

Same structure, different values.

```ts
// Similar structure
function validateUsername(name: string): boolean {
  return name.length >= 3 && name.length <= 20
}

function validatePassword(password: string): boolean {
  return password.length >= 8 && password.length <= 100
}

// Parameterized
function validateLength(value: string, min: number, max: number): boolean {
  return value.length >= min && value.length <= max
}

function validateUsername(name: string): boolean {
  return validateLength(name, 3, 20)
}

function validatePassword(password: string): boolean {
  return validateLength(password, 8, 100)
}
```

### Conceptual Duplication

Same idea, different implementation.

```ts
// Same concept: "send notification"
async function sendEmailNotification(user: User, message: string) {
  await emailService.send(user.email, message)
}

async function sendSmsNotification(user: User, message: string) {
  await smsService.send(user.phone, message)
}

// Unified abstraction
interface NotificationChannel {
  send(recipient: string, message: string): Promise<void>
}

async function sendNotification(
  channel: NotificationChannel,
  recipient: string,
  message: string
) {
  await channel.send(recipient, message)
}
```

## Extraction Techniques

### Extract Function

```ts
// Before
function calculateTotal(items: CartItem[]) {
  let total = 0
  for (const item of items) {
    total += item.price * item.quantity
    if (item.discount) {
      total -= item.discount
    }
  }
  return total
}

// After
function calculateTotal(items: CartItem[]) {
  return items.reduce((sum, item) => sum + calculateItemTotal(item), 0)
}

function calculateItemTotal(item: CartItem): number {
  const base = item.price * item.quantity
  return item.discount ? base - item.discount : base
}
```

### Extract Method (OOP)

```ts
// Before
class OrderProcessor {
  process(order: Order) {
    // Validation logic inline
    if (!order.items.length) throw new Error('Empty order')
    if (order.total < 0) throw new Error('Invalid total')
    // ... rest of processing
  }
}

// After
class OrderProcessor {
  process(order: Order) {
    this.validateOrder(order)
    // ... rest of processing
  }

  private validateOrder(order: Order): void {
    if (!order.items.length) throw new Error('Empty order')
    if (order.total < 0) throw new Error('Invalid total')
  }
}
```

### Extract Module

```ts
// Before: All in one file
// user.ts
export function validateEmail(email: string) { ... }
export function validatePassword(password: string) { ... }
export function validateUsername(name: string) { ... }
export function createUser(data: UserData) { ... }
export function updateUser(user: User, data: UserData) { ... }

// After: Separated by concern
// validation.ts
export function validateEmail(email: string) { ... }
export function validatePassword(password: string) { ... }
export function validateUsername(name: string) { ... }

// user.ts
import { validateEmail, validatePassword, validateUsername } from './validation'
export function createUser(data: UserData) { ... }
export function updateUser(user: User, data: UserData) { ... }
```

## When NOT to Deduplicate

Not all duplication is bad. Duplication can be intentional.

### Different Rates of Change

```ts
// Don't deduplicate: These change independently
function formatCurrency(amount: number, currency: string): string {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(amount)
}

function formatCompactCurrency(amount: number, currency: string): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
    notation: 'compact'
  }).format(amount)
}
```

### Different Contexts

```ts
// Don't deduplicate: Different domains, different rules
function calculateShippingCost(weight: number): number {
  return weight * 0.5 + 5 // E-commerce shipping
}

function calculateDeliveryFee(weight: number): number {
  return weight * 0.3 + 2 // Food delivery, different rules
}
```

### Rule of Three

Wait for 3 instances before abstracting:

```ts
// Instance 1: Just write it
function sendWelcomeEmail(user: User) { ... }

// Instance 2: Similar, but wait
function sendPasswordResetEmail(user: User) { ... }

// Instance 3: Now consider extraction
function sendVerificationEmail(user: User) { ... }

// After 3 instances, extract pattern
function sendTemplatedEmail(user: User, template: EmailTemplate) { ... }
```

## Checklist

- [ ] Identified type of duplication (identical/similar/conceptual)
- [ ] Considered extraction level (function/method/module)
- [ ] Verified same rate of change
- [ ] Applied rule of three before abstracting
