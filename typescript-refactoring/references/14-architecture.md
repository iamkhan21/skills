# Architecture

Use these patterns only for demonstrated boundary pressure within the approved scope. Preserve existing architecture for local refactors; layers, ports, and swappable implementations are options, not a checklist every project must satisfy.

Structure code at the system level for maintainability and scalability.

- [Layers](#layered-architecture)
- [Ports and adapters](#ports-and-adapters-hexagonal)
- [Dependency direction](#dependency-direction)
- [Feature organization](#feature-based-organization)
- [Common patterns](#common-patterns)
- [Incremental migration](#strangler-fig-pattern)
- [Checklist](#checklist)

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

## Strangler Fig Pattern

For large-scale migrations where you can't rewrite everything at once. Gradually replace old code while keeping the system running.

### How It Works

1. **Identify a seam** — a boundary where old and new code can coexist
2. **Build the new implementation** alongside the old one
3. **Route traffic** through a facade that delegates to old or new
4. **Migrate incrementally** — move one feature/route/module at a time
5. **Remove the old code** once all traffic flows through the new path

### Implementation

```ts
// Step 1: Create a facade that delegates to old implementation
class UserServiceFacade implements UserService {
  constructor(
    private legacy: LegacyUserService,
    private modern: ModernUserService,
    private flags: FeatureFlags
  ) {}

  async getUser(id: string): Promise<User> {
    if (this.flags.isEnabled('modern-user-service')) {
      return this.modern.getUser(id)
    }
    return this.legacy.getUser(id)
  }
}

// Step 2: Migrate one method at a time
// Step 3: When all methods use modern, remove facade and legacy
```

### When to Use

- Replacing a framework (e.g., Express → Fastify)
- Migrating to a new database
- Rewriting a module with different architecture
- Moving from monolith to services

### Key Principles

- This pattern lets old and new coexist; use it when incremental rollout justifies the temporary complexity
- Each step is independently deployable
- Verify rollback against data/schema changes, events, and external effects; routing back alone does not undo stateful migrations
- The facade is temporary scaffolding — remove it when done

## Checklist

Apply only to architecture changes that are actually in scope:

- [ ] Layers clearly separated
- [ ] Domain logic isolated from infrastructure
- [ ] Dependencies point inward
- [ ] Ports define interfaces, adapters implement
- [ ] Features can be understood in isolation
- [ ] Easy to swap implementations (e.g., DB, external services)
