# Abstraction

Hide complexity behind simple interfaces. But be careful - bad abstractions are worse than no abstraction.

## Abstraction Levels

### High-Level: Intent

What the code should do, not how.

```ts
// High-level: Clear intent
function sendWelcomeEmail(user: User): Promise<void> {
  return emailService.send({
    to: user.email,
    template: 'welcome',
    data: { name: user.name }
  })
}

// Low-level: How it works (implementation details)
function sendWelcomeEmail(user: User): Promise<void> {
  const smtp = new SMTPClient(config.smtpHost, config.smtpPort)
  const html = renderTemplate('welcome', { name: user.name })
  const message = buildMimeMessage(user.email, 'Welcome!', html)
  return smtp.send(message)
}
```

### Finding the Right Level

```ts
// Too low-level: Implementation details leak
function saveUser(query: string, params: any[]): Promise<void> {
  return database.execute(query, params)
}

// Too high-level: Vague, unclear what happens
function save(data: any): Promise<void> { ... }

// Right level: Clear what, hides how
function saveUser(user: User): Promise<void> {
  return userRepository.save(user)
}
```

## Leaky Abstractions

Abstraction that exposes implementation details it should hide.

### Signs of Leaky Abstraction

```ts
// Leaky: Caller must know about pagination internals
async function getUsers(page: number): Promise<User[]> {
  const offset = page * 20 // Magic number exposed
  const limit = 20         // Caller can't control
  return db.query('SELECT * FROM users LIMIT ? OFFSET ?', [limit, offset])
}

// Fixed: Abstraction hides pagination details
interface PaginatedResult<T> {
  items: T[]
  hasNextPage: boolean
  totalItems: number
}

async function getUsers(page: number, pageSize = 20): Promise<PaginatedResult<User>> {
  const offset = page * pageSize
  const items = await db.query('SELECT * FROM users LIMIT ? OFFSET ?', [pageSize, offset])
  const totalItems = await db.query('SELECT COUNT(*) FROM USERS')
  return {
    items,
    hasNextPage: offset + items.length < totalItems,
    totalItems
  }
}
```

### When Abstraction Leaks

```ts
// Leaky: Performance concerns exposed
function getOrders(): Order[] {
  return ordersCache.get() ?? fetchAllOrdersFromDb() // Cache behavior exposed
}

// If caching is important, make it explicit
function getOrders(options?: { useCache: boolean }): Order[] {
  if (options?.useCache ?? true) {
    return ordersCache.get() ?? fetchAllOrdersFromDb()
  }
  return fetchAllOrdersFromDb()
}
```

## When to Create Abstraction

### Rule of Three

Wait for patterns to emerge before abstracting.

```ts
// Instance 1
function validateEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}

// Instance 2 - Similar, but wait
function validatePhone(phone: string): boolean {
  return /^\d{10}$/.test(phone)
}

// Instance 3 - Pattern emerges
function validateUrl(url: string): boolean {
  return /^https?:\/\/.+/.test(url)
}

// Now extract abstraction
function matchesPattern(value: string, pattern: RegExp): boolean {
  return pattern.test(value)
}

function validateEmail(email: string): boolean {
  return matchesPattern(email, /^[^\s@]+@[^\s@]+\.[^\s@]+$/)
}
```

### When Pattern is Clear

```ts
// Clear pattern from start
interface Validator<T> {
  validate(value: T): boolean
  getMessage(): string
}

class EmailValidator implements Validator<string> {
  validate(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  }
  getMessage(): string {
    return 'Invalid email format'
  }
}
```

## Abstraction Should Simplify

### Good Abstraction

```ts
// Simple interface, hides complexity
interface FileStorage {
  save(path: string, content: Buffer): Promise<void>
  load(path: string): Promise<Buffer>
  delete(path: string): Promise<void>
}

// Usage is simple
await storage.save('data.json', buffer)
```

### Bad Abstraction

```ts
// Exposes too much, complicates usage
interface FileStorage {
  save(path: string, content: Buffer, options: {
    encoding?: BufferEncoding
    mode?: number
    flags?: string
    recursive?: boolean
    tmpfile?: boolean
  }): Promise<void>
  load(path: string, options: {
    encoding?: BufferEncoding
    flag?: string
  }): Promise<Buffer>
}

// Usage is complicated
await storage.save('data.json', buffer, { 
  encoding: 'utf-8',
  mode: 0o644,
  flags: 'w',
  recursive: true 
})
```

## Checklist

- [ ] Abstraction hides implementation details
- [ ] Interface is simpler than implementation
- [ ] No unnecessary options exposed
- [ ] Abstraction level matches caller's needs
- [ ] Pattern repeated at least 3 times (or clearly obvious)
