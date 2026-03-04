# Declarative Style

Describe WHAT you want, not HOW to get it. Reduce boilerplate and increase readability.

## Declarative vs Imperative

### Imperative: How

```ts
// Imperative: Step-by-step instructions
const activeUsers: User[] = []
for (let i = 0; i < users.length; i++) {
  if (users[i].isActive) {
    activeUsers.push(users[i])
  }
}
```

### Declarative: What

```ts
// Declarative: Describe the result
const activeUsers = users.filter(user => user.isActive)
```

## Declarative Patterns

### Array Operations

```ts
// Imperative
const names: string[] = []
for (const user of users) {
  names.push(user.name.toUpperCase())
}

// Declarative
const names = users.map(user => user.name.toUpperCase())
```

```ts
// Imperative
let total = 0
for (const item of cart) {
  if (item.inStock) {
    total += item.price * item.quantity
  }
}

// Declarative
const total = cart
  .filter(item => item.inStock)
  .reduce((sum, item) => sum + item.price * item.quantity, 0)
```

### Object Transformations

```ts
// Imperative
const userMap: Record<string, User> = {}
for (const user of users) {
  userMap[user.id] = user
}

// Declarative
const userMap = Object.fromEntries(
  users.map(user => [user.id, user])
)
```

### Configuration over Code

```ts
// Imperative: Logic scattered
function formatValue(value: unknown, type: string): string {
  if (type === 'currency') {
    return '$' + Number(value).toFixed(2)
  } else if (type === 'percent') {
    return (Number(value) * 100).toFixed(0) + '%'
  } else if (type === 'date') {
    return new Date(value as string).toLocaleDateString()
  } else {
    return String(value)
  }
}

// Declarative: Configuration-driven
const formatters: Record<string, (value: unknown) => string> = {
  currency: v => '$' + Number(v).toFixed(2),
  percent: v => (Number(v) * 100).toFixed(0) + '%',
  date: v => new Date(v as string).toLocaleDateString(),
  default: v => String(v)
}

function formatValue(value: unknown, type: string): string {
  const formatter = formatters[type] ?? formatters.default
  return formatter(value)
}
```

### Data-Driven Logic

```ts
// Imperative
function getPermissions(role: string): string[] {
  if (role === 'admin') {
    return ['read', 'write', 'delete', 'manage']
  } else if (role === 'editor') {
    return ['read', 'write']
  } else if (role === 'viewer') {
    return ['read']
  }
  return []
}

// Declarative
const rolePermissions: Record<string, string[]> = {
  admin: ['read', 'write', 'delete', 'manage'],
  editor: ['read', 'write'],
  viewer: ['read']
}

function getPermissions(role: string): string[] {
  return rolePermissions[role] ?? []
}
```

### SQL-like Queries

```ts
// Declarative with fluent API
const result = await db
  .select('users.id', 'users.name', 'orders.total')
  .from('users')
  .leftJoin('orders', 'users.id', 'orders.userId')
  .where('users.isActive', true)
  .where('orders.total', '>', 100)
  .orderBy('orders.total', 'desc')
  .limit(10)
```

## Declarative UI

### Component-Based

```tsx
// Declarative: UI as function of state
function UserList({ users }: { users: User[] }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          <UserCard user={user} />
        </li>
      ))}
    </ul>
  )
}

// Usage
<UserList users={activeUsers} />
```

### Conditional Rendering

```tsx
// Imperative style (avoid)
function UserStatus({ user }: { user: User }) {
  let statusElement
  
  if (user.isOnline) {
    if (user.isBusy) {
      statusElement = <span className="busy">Busy</span>
    } else {
      statusElement = <span className="online">Online</span>
    }
  } else {
    statusElement = <span className="offline">Offline</span>
  }
  
  return <div>{statusElement}</div>
}

// Declarative style
function UserStatus({ user }: { user: User }) {
  return (
    <div>
      {user.isOnline && user.isBusy && <span className="busy">Busy</span>}
      {user.isOnline && !user.isBusy && <span className="online">Online</span>}
      {!user.isOnline && <span className="offline">Offline</span>}
    </div>
  )
}
```

## Trade-offs

### Benefits

- More readable
- Less boilerplate
- Easier to reason about
- Less prone to off-by-one errors

### Costs

- Less explicit about control flow
- Can hide performance characteristics
- May create intermediate data structures

```ts
// Declarative but inefficient (creates 3 arrays)
const result = items
  .filter(isValid)
  .map(transform)
  .filter(isNeeded)

// Imperative but efficient (single pass)
const result: TransformedItem[] = []
for (const item of items) {
  if (!isValid(item)) continue
  const transformed = transform(item)
  if (!isNeeded(transformed)) continue
  result.push(transformed)
}
```

### When to Choose

| Scenario | Prefer |
|----------|--------|
| Readability matters | Declarative |
| Performance critical | Imperative |
| Complex business logic | Imperative with good naming |
| Data transformations | Declarative |
| Simple conditions | Declarative |
| Complex state management | Hybrid |

## Checklist

- [ ] Use array methods (map, filter, reduce) over loops
- [ ] Configuration-driven over conditional logic
- [ ] Data structures over control flow
- [ ] Express intent clearly
- [ ] Consider performance trade-offs
