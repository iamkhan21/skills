# Architecture

Structure code at the system level for maintainability and scalability.

## Layered Architecture

Separate concerns into horizontal layers.

```
┌─────────────────────────────┐
│     Presentation Layer      │  Controllers, Views, API endpoints
├─────────────────────────────┤
│     Application Layer       │  Use cases, orchestration
├─────────────────────────────┤
│       Domain Layer          │  Business logic, entities
├─────────────────────────────┤
│     Infrastructure Layer    │  Database, external services
└─────────────────────────────┘
```

### Implementation

```ts
// Domain Layer (core business logic)
// domain/user.ts
export class User {
  constructor(
    readonly id: string,
    readonly email: string,
    readonly name: string
  ) {}
  
  canEdit(document: Document): boolean {
    return document.authorId === this.id
  }
}

// Application Layer (use cases)
// application/user-service.ts
export class UserService {
  constructor(
    private userRepo: UserRepository,
    private emailService: EmailService
  ) {}
  
  async register(data: RegisterUser): Promise<User> {
    const user = new User(generateId(), data.email, data.name)
    await this.userRepo.save(user)
    await this.emailService.sendWelcome(user.email)
    return user
  }
}

// Infrastructure Layer (external concerns)
// infrastructure/user-repository.ts
export class SqlUserRepository implements UserRepository {
  constructor(private db: Database) {}
  
  async save(user: User): Promise<void> {
    await this.db.query(
      'INSERT INTO users (id, email, name) VALUES (?, ?, ?)',
      [user.id, user.email, user.name]
    )
  }
}

// Presentation Layer (entry points)
// presentation/user-controller.ts
export class UserController {
  constructor(private userService: UserService) {}
  
  async register(req: Request, res: Response): Promise<void> {
    const user = await this.userService.register(req.body)
    res.status(201).json(user)
  }
}
```

## Ports and Adapters (Hexagonal)

Core domain isolated from external concerns.

```
        ┌──────────────────────────────┐
        │         Adapters             │
        │  ┌─────┐  ┌─────┐  ┌─────┐   │
        │  │ API │  │ CLI │  │  DB │   │
        │  └──┬──┘  └──┬──┘  └──┬──┘   │
        └─────┼────────┼────────┼──────┘
              │        │        │
        ┌─────┴────────┴────────┴──────┐
        │          Ports               │
        └─────┬────────┬────────┬──────┘
              │        │        │
        ┌─────┴────────┴────────┴──────┐
        │      Application Core        │
        │    (Domain + Use Cases)      │
        └──────────────────────────────┘
```

### Ports (Interfaces)

```ts
// Ports: Define what the core needs
export interface UserRepository {
  findById(id: string): Promise<User | null>
  save(user: User): Promise<void>
}

export interface NotificationService {
  send(email: string, message: string): Promise<void>
}
```

### Adapters (Implementations)

```ts
// Adapters: Implement ports for specific technologies
export class SqlUserRepository implements UserRepository {
  constructor(private db: Database) {}
  
  async findById(id: string): Promise<User | null> {
    const row = await this.db.query('SELECT * FROM users WHERE id = ?', [id])
    return row ? this.toDomain(row) : null
  }
  
  async save(user: User): Promise<void> {
    await this.db.query(
      'INSERT INTO users (id, email, name) VALUES (?, ?, ?)',
      [user.id, user.email, user.name]
    )
  }
}

export class EmailNotificationService implements NotificationService {
  async send(email: string, message: string): Promise<void> {
    await sendgrid.send({ to: email, content: message })
  }
}

// Test adapter
export class InMemoryUserRepository implements UserRepository {
  private users = new Map<string, User>()
  
  async findById(id: string): Promise<User | null> {
    return this.users.get(id) ?? null
  }
  
  async save(user: User): Promise<void> {
    this.users.set(user.id, user)
  }
}
```

## Dependency Direction

Dependencies flow inward toward the domain.

```
   External         →    Domain
   Infrastructure   →    Core
   
   DB Adapter       →    UserRepo Port
   API Controller   →    UserService
```

```ts
// Domain has no external dependencies
// domain/user.ts - pure TypeScript, no imports from infrastructure

// Infrastructure depends on domain
// infrastructure/user-repository.ts
import { User } from '../domain/user'
import { UserRepository } from '../ports/user-repository'
```

## Feature-Based Organization

Organize by feature rather than layer.

```
src/
├── features/
│   ├── users/
│   │   ├── domain/
│   │   │   └── user.ts
│   │   ├── application/
│   │   │   └── user-service.ts
│   │   ├── infrastructure/
│   │   │   └── user-repository.ts
│   │   └── presentation/
│   │       └── user-controller.ts
│   ├── orders/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   └── presentation/
│   └── products/
└── shared/
    ├── domain/
    └── infrastructure/
```

### Benefits

- Related code stays together
- Easy to understand a feature
- Can delete a feature entirely
- Reduces coupling between features

## Common Patterns

### Repository Pattern

```ts
// Abstraction for data access
interface OrderRepository {
  findById(id: string): Promise<Order | null>
  findByCustomer(customerId: string): Promise<Order[]>
  save(order: Order): Promise<void>
}
```

### Service Layer

```ts
// Orchestrates use cases
class OrderService {
  constructor(
    private orderRepo: OrderRepository,
    private productRepo: ProductRepository,
    private paymentGateway: PaymentGateway
  ) {}
  
  async placeOrder(customerId: string, items: OrderItem[]): Promise<Order> {
    const products = await this.validateProducts(items)
    const total = this.calculateTotal(products, items)
    await this.paymentGateway.charge(customerId, total)
    const order = new Order(generateId(), customerId, items, total)
    await this.orderRepo.save(order)
    return order
  }
}
```

### Domain Events

```ts
// Decouple side effects
class Order {
  private events: DomainEvent[] = []
  
  place(): void {
    this.status = 'placed'
    this.events.push(new OrderPlacedEvent(this.id, this.total))
  }
  
  pullEvents(): DomainEvent[] {
    const events = [...this.events]
    this.events = []
    return events
  }
}
```

## Checklist

- [ ] Layers clearly separated
- [ ] Domain logic isolated from infrastructure
- [ ] Dependencies point inward
- [ ] Ports define interfaces, adapters implement
- [ ] Features can be understood in isolation
- [ ] Easy to swap implementations (e.g., DB, external services)
