# Comments and Documentation

Code should be self-documenting. Comments should explain WHY, not WHAT.

## Self-Documenting Code

### Use Clear Names

```ts
// Bad: Needs comment to explain
// Check if user can edit the document
function check(u, d) {
  return u.id === d.authorId || u.role === 'admin'
}

// Good: Self-documenting
function canUserEditDocument(user: User, document: Document): boolean {
  return user.id === document.authorId || user.role === 'admin'
}
```

### Extract to Functions

```ts
// Bad: Comment explains complex logic
// Calculate total with 10% discount for premium users,
// but only if the order is over $50
const total = items.reduce((sum, item) => sum + item.price, 0)
const discountedTotal = user.isPremium && total > 50 
  ? total * 0.9 
  : total

// Good: Function name documents intent
const total = calculateOrderTotal(items, user)

function calculateOrderTotal(items: CartItem[], user: User): number {
  const subtotal = items.reduce((sum, item) => sum + item.price, 0)
  return applyPremiumDiscount(subtotal, user)
}

function applyPremiumDiscount(amount: number, user: User): number {
  const ELIGIBLE_AMOUNT = 50
  const PREMIUM_DISCOUNT = 0.1
  
  if (!user.isPremium || amount <= ELIGIBLE_AMOUNT) {
    return amount
  }
  return amount * (1 - PREMIUM_DISCOUNT)
}
```

### Use Constants

```ts
// Bad: Magic numbers with comments
if (age >= 18) { // Legal adult age
  // ...
}
setTimeout(callback, 86400000) // 24 hours in milliseconds

// Good: Named constants
const LEGAL_ADULT_AGE = 18
const TWENTY_FOUR_HOURS_MS = 24 * 60 * 60 * 1000

if (age >= LEGAL_ADULT_AGE) {
  // ...
}
setTimeout(callback, TWENTY_FOUR_HOURS_MS)
```

## When to Use Comments

### Explain WHY

```ts
// Good: Explains business reason
// Use exponential backoff because the external API rate-limits aggressively
await delay(Math.pow(2, retryCount) * 1000)

// Good: Explains non-obvious decision
// Using Map instead of Object because we need to preserve insertion order
const users = new Map<string, User>()
```

### Document Workarounds

```ts
// Good: Documents workaround for external issue
// Workaround: API returns 500 for empty results, treat as empty array
if (response.status === 500) {
  return []
}
```

### Reference External Sources

```ts
// Good: Links to relevant documentation
// See: https://stripe.com/docs/api/rate_limits
const MAX_RETRIES = 5

// Good: References specification
// RFC 5322 email format: https://www.rfc-editor.org/rfc/rfc5322
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
```

## When NOT to Use Comments

### Don't Explain WHAT

```ts
// Bad: Comment restates code
// Increment counter by 1
counter++

// Bad: Comment describes obvious behavior
// Returns the user's name
function getName(): string {
  return this.name
}

// Bad: Comment that could be code
// Check if user is premium and order is over 50 dollars
if (user.isPremium && total > 50) {
  // ...
}
```

### Don't Comment Bad Code - Fix It

```ts
// Bad: Comment apologizes for bad code
// This is a bit messy but it works
const x = data?.items?.filter?.(i => i?.active)?.map?.(i => i?.value)

// Good: Fix the code
const activeValues = data?.items
  ?.filter(item => item?.active)
  ?.map(item => item?.value) ?? []
```

## JSDoc for Public APIs

Use JSDoc for libraries and public interfaces.

```ts
/**
 * Calculates the total price of items in a cart.
 * 
 * @param items - Array of cart items with price and quantity
 * @param options - Calculation options
 * @returns The total price in the cart's currency
 * 
 * @example
 * ```ts
 * const total = calculateTotal(
 *   [{ price: 10, quantity: 2 }],
 *   { includeTax: true }
 * )
 * // total = 22 (assuming 10% tax)
 * ```
 */
function calculateTotal(
  items: CartItem[],
  options?: CalculateOptions
): number {
  // ...
}
```

### JSDoc for Types

```ts
/**
 * Represents a user in the system.
 * 
 * @property id - Unique identifier
 * @property email - User's email address (must be unique)
 * @property role - User's permission level
 */
interface User {
  id: string
  email: string
  role: 'admin' | 'user' | 'guest'
}
```

## README Documentation

Keep README focused on getting started.

```markdown
# Project Name

Brief description of what this project does.

## Quick Start

```bash
npm install
npm run dev
```

## Usage

Basic usage example.

## Configuration

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| API_URL  | Yes      | -       | API endpoint |

## API

Brief API documentation with examples.
```

## Update Docs When Code Changes

```ts
// Bad: Outdated comment
// Returns the first 10 users (actually returns all users now)
function getUsers(): User[] {
  return userRepo.findAll()
}

// Good: Remove comment, or update it
function getUsers(options?: { limit?: number }): User[] {
  return userRepo.findAll(options?.limit ?? 10)
}
```

## Checklist

- [ ] Code is self-documenting
- [ ] Comments explain WHY, not WHAT
- [ ] No comments that restate code
- [ ] JSDoc for public APIs
- [ ] Constants replace magic numbers
- [ ] Documentation updated with code changes
