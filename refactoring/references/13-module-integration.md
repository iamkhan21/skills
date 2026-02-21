# Module Integration

Design modules with clear boundaries and controlled dependencies.

## Coupling

How tightly modules depend on each other. Prefer loose coupling.

### Types of Coupling (Worst to Best)

#### Content Coupling (Worst)

One module directly accesses another's internals.

```ts
// Bad: Direct access to internal state
class UserService {
  constructor(private db: Database) {}
  
  getUser(id: string): User {
    return db.tables.users.rows.find(u => u.id === id) // Knows db internals
  }
}

// Good: Access through interface
class UserService {
  constructor(private userRepo: UserRepository) {}
  
  getUser(id: string): User {
    return this.userRepo.findById(id)
  }
}
```

#### Common Coupling

Modules share global data.

```ts
// Bad: Shared global state
let currentUser: User | null = null

class OrderService {
  createOrder(items: Item[]): Order {
    return { userId: currentUser?.id, items } // Depends on global
  }
}

class PaymentService {
  processPayment(amount: number): void {
    charge(currentUser?.id, amount) // Depends on same global
  }
}

// Good: Pass dependencies explicitly
class OrderService {
  createOrder(userId: string, items: Item[]): Order {
    return { userId, items }
  }
}
```

#### Control Coupling

One module controls another's flow via flags.

```ts
// Bad: Flag controls behavior
function processOrder(order: Order, isPremium: boolean): number {
  if (isPremium) {
    return calculatePremiumOrder(order)
  }
  return calculateRegularOrder(order)
}

// Good: Strategy pattern
interface OrderCalculator {
  calculate(order: Order): number
}

function processOrder(order: Order, calculator: OrderCalculator): number {
  return calculator.calculate(order)
}
```

#### Data Coupling (Best)

Modules share only data through interfaces.

```ts
// Good: Share only necessary data through clear interface
interface UserSummary {
  id: string
  name: string
}

class UserAPI {
  getUserSummary(id: string): UserSummary {
    const user = this.db.getUser(id)
    return { id: user.id, name: user.name } // Only expose what's needed
  }
}
```

## Cohesion

How related a module's responsibilities are. Prefer high cohesion.

### Low Cohesion (Bad)

Module does many unrelated things.

```ts
// Low cohesion: Does too many things
class UserService {
  createUser(data: UserData): User { ... }
  sendEmail(user: User, message: string): void { ... }
  generateReport(users: User[]): Report { ... }
  validatePayment(payment: Payment): boolean { ... }
  logActivity(action: string): void { ... }
}
```

### High Cohesion (Good)

Module focuses on one responsibility.

```ts
// High cohesion: Single responsibility
class UserService {
  constructor(
    private userRepo: UserRepository,
    private emailService: EmailService
  ) {}
  
  create(data: UserData): User { ... }
  findById(id: string): User | undefined { ... }
  update(id: string, data: Partial<UserData>): User { ... }
  delete(id: string): void { ... }
}

class EmailService {
  send(to: string, subject: string, body: string): void { ... }
  sendTemplate(to: string, template: string, data: object): void { ... }
}

class ReportService {
  generateUserReport(users: User[]): Report { ... }
  generateSalesReport(orders: Order[]): Report { ... }
}
```

## Contracts Between Modules

Define clear interfaces for communication.

```ts
// Contract: What the module expects and provides
interface UserRepository {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  save(user: User): Promise<void>
  delete(id: string): Promise<void>
}

// Implementation
class SqlUserRepository implements UserRepository {
  constructor(private db: Database) {}
  
  async findById(id: string): Promise<User | null> {
    const row = await this.db.query('SELECT * FROM users WHERE id = ?', [id])
    return row ? this.toUser(row) : null
  }
  
  // ... other methods
}

// Consumer depends on contract, not implementation
class UserService {
  constructor(private userRepo: UserRepository) {}
  
  async getUser(id: string): Promise<User> {
    const user = await this.userRepo.findById(id)
    if (!user) throw new NotFoundError('User', id)
    return user
  }
}
```

## Dependency Direction

Dependencies should point toward stability.

### Dependency Inversion

High-level modules shouldn't depend on low-level modules. Both should depend on abstractions.

```ts
// Bad: High-level depends on low-level
class UserService {
  private db = new PostgresDatabase() // Direct dependency
  
  getUser(id: string): User {
    return this.db.query('SELECT * FROM users WHERE id = $1', [id])
  }
}

// Good: Both depend on abstraction
interface Database {
  query<T>(sql: string, params: any[]): Promise<T>
}

class UserService {
  constructor(private db: Database) {} // Depends on abstraction
  
  async getUser(id: string): Promise<User> {
    return this.db.query('SELECT * FROM users WHERE id = $1', [id])
  }
}

class PostgresDatabase implements Database {
  async query<T>(sql: string, params: any[]): Promise<T> {
    // Implementation
  }
}
```

### Dependency Graph

```
          Controllers (Unstable - changes often)
                ↓
           Services
                ↓
          Repositories
                ↓
         Abstractions (Stable - rarely changes)
                ↑
         Implementations
```

## Module Communication Patterns

### Synchronous (Direct)

```ts
// Direct call
const user = await userService.getUser(id)
```

### Event-Based (Decoupled)

```ts
// Publisher
class UserService {
  constructor(private eventBus: EventBus) {}
  
  async createUser(data: UserData): Promise<User> {
    const user = await this.repo.save(data)
    this.eventBus.emit('user.created', user)
    return user
  }
}

// Subscriber
class EmailService {
  constructor(eventBus: EventBus) {
    eventBus.on('user.created', (user: User) => {
      this.sendWelcomeEmail(user)
    })
  }
}
```

## Checklist

- [ ] Modules loosely coupled
- [ ] Each module has high cohesion
- [ ] Clear contracts between modules
- [ ] Dependencies point toward abstractions
- [ ] No circular dependencies
- [ ] Single responsibility per module
