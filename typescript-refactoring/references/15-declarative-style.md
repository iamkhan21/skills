# Declarative Style

Describe WHAT you want, not HOW to get it. Reduce boilerplate and increase readability.
Choose the form that is clearest in context. The array examples assume dense arrays and pure callbacks; changing evaluation order or handling of holes is not behavior-preserving by default.

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
// Use a Map for arbitrary IDs, including keys such as '__proto__'.
const userMap = new Map(users.map(user => [user.id, user]))
```

Replacing an object with a Map changes the access API. Migrate callers explicitly. Plain object assignment and `Object.fromEntries` also differ for special keys; do not treat them as universally equivalent.

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
const formatters = new Map<string, (value: unknown) => string>([
  ['currency', v => '$' + Number(v).toFixed(2)],
  ['percent', v => (Number(v) * 100).toFixed(0) + '%'],
  ['date', v => new Date(v as string).toLocaleDateString()]
])

function formatValue(value: unknown, type: string): string {
  const formatter = formatters.get(type)
  return formatter ? formatter(value) : String(value)
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
const rolePermissions = new Map<string, readonly string[]>([
  ['admin', ['read', 'write', 'delete', 'manage']],
  ['editor', ['read', 'write']],
  ['viewer', ['read']]
])

function getPermissions(role: string): string[] {
  return [...(rolePermissions.get(role) ?? [])]
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
// Valid explicit branching
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

Use the version that is easier to read. Repeated boolean expressions are not automatically an improvement over branching.

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

Measure hot paths before optimizing. A single-pass rewrite interleaves callbacks differently from staged methods; require pure callbacks with no observable ordering dependencies before treating them as equivalent.

| Scenario | Prefer |
|----------|--------|
| Readability matters | Declarative |
| Performance critical | Imperative |
| Complex business logic | Imperative with good naming |
| Data transformations | Declarative |
| Simple conditions | Declarative |
| Complex state management | Hybrid |

## Checklist

- [ ] Chosen form is clearer than the alternative, whether loop, switch, or pipeline
- [ ] Lookups handle unknown/prototype-like keys and preserve return-value ownership
- [ ] Callback order, sparse data, mutation, and identity remain stable
- [ ] Express intent clearly
- [ ] Consider performance trade-offs
