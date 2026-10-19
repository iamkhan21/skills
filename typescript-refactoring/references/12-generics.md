# Generics

Use generics when behavior is the same but types vary. Know when to use generics vs inheritance vs composition.

## When to Use Generics

### Same Logic, Different Types

```ts
// Without generics: Duplicated logic
function getFirstItem(items: string[]): string | undefined {
  return items[0]
}

function getFirstNumber(numbers: number[]): number | undefined {
  return numbers[0]
}

// With generics: Single implementation
function getFirst<T>(items: T[]): T | undefined {
  return items[0]
}
```

### Container Types

```ts
// Generic container
class Box<T> {
  constructor(private value: T) {}
  
  get(): T {
    return this.value
  }
  
  map<U>(fn: (value: T) => U): Box<U> {
    return new Box(fn(this.value))
  }
}

const numberBox = new Box(42)
const stringBox = numberBox.map(n => n.toString())
```

### Utility Functions

```ts
// Generic utilities
function identity<T>(value: T): T {
  return value
}

function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>
  for (const key of keys) {
    result[key] = obj[key]
  }
  return result
}

function omit<T, K extends keyof T>(obj: T, keys: K[]): Omit<T, K> {
  const result = { ...obj }
  for (const key of keys) {
    delete result[key]
  }
  return result
}
```

## When NOT to Use Generics

### When Types Have Different Behavior

```ts
// Bad: Generics force same behavior
function process<T>(value: T): string {
  // Can't do anything type-specific with T
  return String(value) // Loses type information
}

// Good: Use union types or overloads
function process(value: string): string
function process(value: number): number
function process(value: string | number): string | number {
  return value
}
```

### When a Relationship Needs a Constraint

```ts
// Express the required shape; inheritance alone does not constrain T.
interface Entity {
  id: string
}

interface Repository<T extends Entity> {
  save(item: T): void
  find(id: string): T | undefined
}

type User = { id: string; name: string }
type UserRepository = Repository<User>
// type StringRepository = Repository<string> // Error: string has no id
```

## Generics vs Inheritance vs Composition

### Use Generics When

- Same behavior for different types
- Type information needs to flow through
- Building utilities or containers

```ts
// Generic: Type flows through
function wrapInArray<T>(value: T): T[] {
  return [value]
}
```

### Use Inheritance When

- "Is-a" relationship exists
- Shared behavior with specialization
- Need to override methods

```ts
// Inheritance: Dog IS-A Animal
abstract class Animal {
  abstract makeSound(): void
  
  move(): void {
    console.log('Moving')
  }
}

class Dog extends Animal {
  makeSound(): void {
    console.log('Woof')
  }
}
```

### Use Composition When

- "Has-a" relationship
- Behavior can be shared across unrelated types
- Need to combine multiple behaviors

```ts
// Composition: Car HAS-A Engine, HAS-A Wheels
interface Engine {
  start(): void
  stop(): void
}

interface Wheels {
  rotate(): void
}

class Car {
  constructor(
    private engine: Engine,
    private wheels: Wheels
  ) {}
  
  drive(): void {
    this.engine.start()
    this.wheels.rotate()
  }
}
```

## Generic Constraints

```ts
// Constrain generic to have certain properties
interface WithId {
  id: string
}

function findById<T extends WithId>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id)
}

// Usage
const users = [{ id: '1', name: 'Alice' }, { id: '2', name: 'Bob' }]
const user = findById(users, '1') // Works: users have id
```

### Multiple Constraints

```ts
interface Timestamped {
  createdAt: Date
  updatedAt: Date
}

function findByDate<T extends WithId & Timestamped>(
  items: T[],
  date: Date
): T | undefined {
  return items.find(item => item.createdAt <= date)
}
```

## Generic Factories

```ts
// Preserve constructor arguments and result type without a global cache.
function create<T, Args extends unknown[]>(
  Constructor: new (...args: Args) => T,
  ...args: Args
): T {
  return new Constructor(...args)
}
```

Direct construction is simpler when no shared creation policy exists. Do not cache instances by class name: names can collide or be minified, later constructor arguments can be ignored, and constructing before a cache lookup still runs side effects.

## Checklist

- [ ] Same logic applies to multiple types
- [ ] Type information needs to be preserved
- [ ] Not forcing different behaviors into one generic
- [ ] Considered inheritance for "is-a" relationships
- [ ] Considered composition for "has-a" relationships
- [ ] Constraints used when type must have certain properties
