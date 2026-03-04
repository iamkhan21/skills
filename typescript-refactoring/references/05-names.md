# Names

Clear names reveal clear thinking. Unclear names signal underlying problems.

## Name Quality Signals

### Unclear Names Indicate Problems

```ts
// Bad: Name hides coupling
function process(data: any) { ... }

// Better: Name reveals dependencies
function validateAndSaveUser(user: User) { ... }

// Best: Split reveals separate concerns
function validateUser(user: User): ValidationResult { ... }
function saveUser(user: User): void { ... }
```

### Name Changes Force Design Improvements

When you can't find a good name, the design might be wrong.

```ts
// Bad: Can't name this well
function handleThing(x: any, flag: boolean) { ... }

// Better: Forced to clarify
function sendEmailNotification(user: User, isUrgent: boolean) { ... }
```

## Common Problems and Solutions

### Too Short Names

```ts
// Bad: Too cryptic
const x = users.filter(u => u.a) // What's u.a?
const d = new Date()

// Good: Descriptive
const activeUsers = users.filter(user => user.isActive)
const currentDate = new Date()
```

### Inaccurate Names

```ts
// Bad: Name doesn't match behavior
function getUser(id: string) {
  return users.find(u => u.id === id) // Returns undefined, not user
}

// Good: Accurate
function findUserById(id: string): User | undefined {
  return users.find(u => u.id === id)
}
```

### Inconsistent Names

```ts
// Bad: Same concept, different names
fetchUsers()
getPosts()
retrieveComments()

// Good: Consistent vocabulary
fetchUsers()
fetchPosts()
fetchComments()
```

## Naming Patterns

### Boolean Names

Pattern: `is/has/can/should` + condition

```ts
// Good
isActive
hasPermission
canEdit
shouldRender

// Avoid
active // Is it a flag or a value?
permission // Unclear
editable // Is this state or capability?
```

### Function Names

Pattern: `verb` + `what` + `conditions`

```ts
// Good
calculateTotal(items: CartItem[]): number
sendEmailToUser(user: User): Promise<void>
validatePassword(password: string): boolean

// Avoid
calc(items) // Too short
process(user) // Vague
check(x) // Cryptic
```

### Getters vs Finders

```ts
// getUser: Assumes existence, throws if not found
function getUser(id: string): User {
  const user = users.find(u => u.id === id)
  if (!user) throw new UserNotFoundError(id)
  return user
}

// findUser: May return undefined
function findUser(id: string): User | undefined {
  return users.find(u => u.id === id)
}
```

### Event Handlers

```ts
// Good: Describes what happens
function handleFormSubmit(event: FormEvent): void
function onUserLoggedIn(user: User): void

// Avoid: Generic
function handleSubmit(event: Event): void
function handleEvent(event: Event): void
```

## Variable Names

### Loop Variables

```ts
// Bad: Single letters lose meaning in complex loops
for (let i = 0; i < users.length; i++) {
  for (let j = 0; j < users[i].posts.length; j++) {
    console.log(users[i].posts[j]) // What's i and j?
  }
}

// Good: Descriptive indices
for (let userIndex = 0; userIndex < users.length; userIndex++) {
  const user = users[userIndex]
  for (let postIndex = 0; postIndex < user.posts.length; postIndex++) {
    console.log(user.posts[postIndex])
  }
}

// Even better: For-of
for (const user of users) {
  for (const post of user.posts) {
    console.log(post)
  }
}
```

### Temporary Variables

```ts
// Bad: Generic temp names
const temp = getItems()
const result = temp.filter(isValid)

// Good: Descriptive intermediate values
const allItems = getItems()
const validItems = allItems.filter(isValid)
```

## Checklist

- [ ] Names reveal intent
- [ ] Consistent vocabulary used
- [ ] Booleans use is/has/can/should
- [ ] Functions use verb + what + conditions
- [ ] No abbreviations without good reason
- [ ] Same concept uses same name throughout
