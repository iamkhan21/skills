# Test Code

Tests need refactoring too. Keep them readable and maintainable.

> **Note:** For React component and hook testing specifically (render, screen, userEvent, MSW), defer to the `react-testing` skill. This file covers general test code refactoring patterns.

## Test Behavior, Not Implementation

```ts
// Bad: Testing implementation details
describe('UserService', () => {
  it('sets isLoading to true then false', async () => {
    const service = new UserService()
    service.loadUsers()
    expect(service.isLoading).toBe(true)
    await service.loadingPromise
    expect(service.isLoading).toBe(false)
  })
})

// Good: Testing observable behavior
describe('UserService', () => {
  it('returns users after loading', async () => {
    const service = new UserService()
    const users = await service.getUsers()
    expect(users).toHaveLength(3)
    expect(users[0]).toHaveProperty('name')
  })
})
```

## Test Structure: Arrange-Act-Assert

```ts
describe('calculateTotal', () => {
  it('sums item prices multiplied by quantity', () => {
    // Arrange
    const items: CartItem[] = [
      { price: 10, quantity: 2 },
      { price: 5, quantity: 3 }
    ]
    
    // Act
    const total = calculateTotal(items)
    
    // Assert
    expect(total).toBe(35) // 10*2 + 5*3 = 35
  })
})
```

## One Concept Per Test

```ts
// Bad: Multiple assertions, unclear what fails
it('validates user correctly', () => {
  expect(validateUser({ name: '', email: '' }).isValid).toBe(false)
  expect(validateUser({ name: 'A', email: '' }).isValid).toBe(false)
  expect(validateUser({ name: 'Alice', email: 'invalid' }).isValid).toBe(false)
  expect(validateUser({ name: 'Alice', email: 'a@b.com' }).isValid).toBe(true)
})

// Good: One concept, clear failure message
describe('validateUser', () => {
  it('fails when name is empty', () => {
    expect(validateUser({ name: '', email: 'a@b.com' }).isValid).toBe(false)
  })
  
  it('fails when email is empty', () => {
    expect(validateUser({ name: 'Alice', email: '' }).isValid).toBe(false)
  })
  
  it('fails when email is invalid', () => {
    expect(validateUser({ name: 'Alice', email: 'invalid' }).isValid).toBe(false)
  })
  
  it('passes with valid data', () => {
    expect(validateUser({ name: 'Alice', email: 'a@b.com' }).isValid).toBe(true)
  })
})
```

## Test Fixtures

Extract test data to keep tests focused.

```ts
// fixtures/user.ts
export function createTestUser(overrides?: Partial<User>): User {
  return {
    id: 'test-user-id',
    name: 'Test User',
    email: 'test@example.com',
    createdAt: new Date('2024-01-01'),
    ...overrides
  }
}

// Test file
describe('UserService', () => {
  it('finds user by email', async () => {
    const user = createTestUser({ email: 'specific@test.com' })
    // ...
  })
})
```

## Refactoring Tests

### Extract Test Helpers

```ts
// Before: Repeated setup
it('calculates total with discount', () => {
  const cart = [
    { price: 100, quantity: 1, discount: 10 },
    { price: 50, quantity: 2, discount: 5 }
  ]
  const total = calculateTotal(cart)
  expect(total).toBe(185)
})

it('calculates total without discount', () => {
  const cart = [
    { price: 100, quantity: 1 },
    { price: 50, quantity: 2 }
  ]
  const total = calculateTotal(cart)
  expect(total).toBe(200)
})

// After: Helper function
function createCartItem(
  price: number,
  quantity: number,
  discount?: number
): CartItem {
  return { price, quantity, discount: discount ?? 0 }
}

it('calculates total with discount', () => {
  const cart = [
    createCartItem(100, 1, 10),
    createCartItem(50, 2, 5)
  ]
  expect(calculateTotal(cart)).toBe(185)
})
```

### Extract Assertions

```ts
// Before: Repeated assertions
it('creates valid user response', async () => {
  const response = await createUser(userData)
  expect(response).toHaveProperty('id')
  expect(response.id).toMatch(/^[a-f0-9-]+$/)
  expect(response).toHaveProperty('createdAt')
  expect(response.createdAt).toBeInstanceOf(Date)
})

// After: Custom matcher or helper
function expectValidUser(response: unknown) {
  expect(response).toMatchObject({
    id: expect.stringMatching(/^[a-f0-9-]+$/),
    createdAt: expect.any(Date)
  })
}

it('creates valid user response', async () => {
  const response = await createUser(userData)
  expectValidUser(response)
})
```

## Test Organization

### Group Related Tests

```ts
describe('UserService', () => {
  describe('create', () => {
    it('creates user with valid data')
    it('rejects duplicate email')
    it('hashes password')
  })
  
  describe('findById', () => {
    it('returns user when found')
    it('returns null when not found')
  })
  
  describe('update', () => {
    it('updates allowed fields')
    it('ignores forbidden fields')
    it('validates email format')
  })
})
```

## Mocking Patterns

### Dependency Injection

```ts
// Before: Hard to mock
class UserService {
  private db = new Database() // Hard-coded dependency
  
  async getUser(id: string) {
    return this.db.find(id)
  }
}

// After: Injectable dependencies
class UserService {
  constructor(private db: Database) {}
  
  async getUser(id: string) {
    return this.db.find(id)
  }
}

// Test
const mockDb = { find: jest.fn() }
const service = new UserService(mockDb)
```

### Interface-Based Mocking

```ts
// Interface for mocking
interface UserRepository {
  findById(id: string): Promise<User | null>
  save(user: User): Promise<void>
}

// Mock implementation
class MockUserRepository implements UserRepository {
  private users = new Map<string, User>()
  
  async findById(id: string): Promise<User | null> {
    return this.users.get(id) ?? null
  }
  
  async save(user: User): Promise<void> {
    this.users.set(user.id, user)
  }
}

// Use in tests
const repo = new MockUserRepository()
const service = new UserService(repo)
```

## Checklist

- [ ] Tests verify behavior, not implementation
- [ ] Arrange-Act-Assert structure
- [ ] One concept per test
- [ ] Test fixtures for data
- [ ] Helpers for repeated patterns
- [ ] Dependencies injected for mocking
