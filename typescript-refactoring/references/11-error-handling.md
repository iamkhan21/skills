# Error Handling

Handle errors at the appropriate level with clear strategies.

## Types of Errors

### Expected Errors

These are part of the domain, not bugs.

- Validation failures
- Resource not found
- Business rule violations
- Rate limiting

```ts
// Expected error: User input validation
class ValidationError extends Error {
  constructor(public field: string, message: string) {
    super(message)
    this.name = 'ValidationError'
  }
}

function validateAge(age: number): void {
  if (age < 0 || age > 150) {
    throw new ValidationError('age', 'Age must be between 0 and 150')
  }
}
```

### Unexpected Errors

These are bugs or external failures.

- Null pointer exceptions
- Type errors
- Network failures
- Database connection errors

```ts
// Unexpected error: Something went wrong
class UnexpectedError extends Error {
  constructor(message: string, public cause?: Error) {
    super(message)
    this.name = 'UnexpectedError'
  }
}
```

## Either/Result Pattern

Return success or error explicitly, don't throw for expected cases.

```ts
// Basic Either type
type Either<E, T> = 
  | { success: true; value: T }
  | { success: false; error: E }

// Result type
type Result<T> = Either<string, T>

// Usage
function divide(a: number, b: number): Result<number> {
  if (b === 0) {
    return { success: false, error: 'Division by zero' }
  }
  return { success: true, value: a / b }
}

// Handling
const result = divide(10, 2)
if (result.success) {
  console.log(result.value) // 5
} else {
  console.error(result.error)
}
```

### Full Result Implementation

```ts
type Result<T, E = Error> = 
  | { ok: true; value: T }
  | { ok: false; error: E }

function Ok<T>(value: T): Result<T> {
  return { ok: true, value }
}

function Err<E>(error: E): Result<never, E> {
  return { ok: false, error }
}

// Usage with domain errors
type UserError = 
  | { type: 'not_found'; id: string }
  | { type: 'validation'; field: string; message: string }

function findUser(id: string): Result<User, UserError> {
  if (!id) {
    return Err({ type: 'validation', field: 'id', message: 'ID required' })
  }
  
  const user = users.find(u => u.id === id)
  if (!user) {
    return Err({ type: 'not_found', id })
  }
  
  return Ok(user)
}

// Handling with type narrowing
const result = findUser('123')
if (result.ok) {
  console.log(result.value.name)
} else {
  switch (result.error.type) {
    case 'not_found':
      console.error(`User ${result.error.id} not found`)
      break
    case 'validation':
      console.error(`Invalid ${result.error.field}: ${result.error.message}`)
      break
  }
}
```

## Custom Error Types

```ts
// Base application error
class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 500
  ) {
    super(message)
    this.name = this.constructor.name
  }
}

// Specific errors
class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} with id ${id} not found`, 'NOT_FOUND', 404)
  }
}

class ValidationError extends AppError {
  constructor(public readonly errors: FieldError[]) {
    super('Validation failed', 'VALIDATION_ERROR', 400)
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 'UNAUTHORIZED', 401)
  }
}

// Usage
function getUser(id: string): User {
  const user = users.find(u => u.id === id)
  if (!user) {
    throw new NotFoundError('User', id)
  }
  return user
}
```

## Handle at Appropriate Level

### Low Level: Throw Specific Errors

```ts
function parseConfig(data: unknown): Config {
  if (!isConfig(data)) {
    throw new ValidationError(['Invalid config structure'])
  }
  return data
}
```

### Mid Level: Transform and Propagate

```ts
async function loadConfig(path: string): Promise<Config> {
  try {
    const data = await fs.readFile(path, 'utf-8')
    return parseConfig(JSON.parse(data))
  } catch (error) {
    if (error instanceof ValidationError) {
      throw error // Re-throw domain errors
    }
    throw new AppError(`Failed to load config from ${path}`, 'CONFIG_LOAD_ERROR')
  }
}
```

### High Level: Handle and Respond

```ts
async function handleRequest(req: Request, res: Response) {
  try {
    const config = await loadConfig(req.body.path)
    res.json({ success: true, config })
  } catch (error) {
    if (error instanceof AppError) {
      res.status(error.statusCode).json({
        success: false,
        code: error.code,
        message: error.message
      })
    } else {
      // Unexpected error
      console.error('Unexpected error:', error)
      res.status(500).json({
        success: false,
        code: 'INTERNAL_ERROR',
        message: 'An unexpected error occurred'
      })
    }
  }
}
```

## Retry Strategies

```ts
async function withRetry<T>(
  fn: () => Promise<T>,
  options: {
    maxRetries?: number
    delay?: number
    backoff?: 'fixed' | 'exponential'
  } = {}
): Promise<T> {
  const { maxRetries = 3, delay = 1000, backoff = 'exponential' } = options
  
  let lastError: Error | undefined
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn()
    } catch (error) {
      lastError = error as Error
      
      if (attempt < maxRetries) {
        const waitTime = backoff === 'exponential' 
          ? delay * Math.pow(2, attempt)
          : delay
        await new Promise(resolve => setTimeout(resolve, waitTime))
      }
    }
  }
  
  throw lastError
}

// Usage
const data = await withRetry(() => fetchData(), { maxRetries: 3, delay: 500 })
```

## Checklist

- [ ] Expected vs unexpected errors distinguished
- [ ] Custom error types for domain errors
- [ ] Either/Result pattern for expected failures
- [ ] Errors handled at appropriate level
- [ ] Retry strategy for transient failures
- [ ] Error messages include actionable information
