# Low-Hanging Fruit

Quick wins that improve code with minimal risk. Start here before deeper refactoring.

## Code Formatting

Automate with Prettier or equivalent.

```bash
# One-time setup
npm install --save-dev prettier

# Format all files
npx prettier --write .
```

### Benefits

- Consistent style across codebase
- Eliminates bike-shedding in PRs
- Zero cognitive overhead

### Commit Strategy

```bash
git commit -m "Apply Prettier formatting"
```

One commit for formatting. Don't mix with other changes.

## Code Linting

Fix each lint rule as separate commit.

```jsonc
// .eslintrc.json
{
  "rules": {
    "no-unused-vars": "error",
    "prefer-const": "error",
    "no-console": "warn"
  }
}
```

### Incremental Fixing

```bash
# Fix one rule at a time
eslint --fix --rule 'prefer-const: error' src/
git commit -m "Fix prefer-const lint rule"

eslint --fix --rule 'no-unused-vars: error' src/
git commit -m "Remove unused variables"
```

### Why Separate Commits?

- Easier to review
- Clear what changed
- Simple to revert if needed

## Language Features

Replace self-written code with native language features.

### Examples (JavaScript/TypeScript)

```ts
// Before: Manual object merging
const merged = {}
for (const key in defaults) {
  merged[key] = defaults[key]
}
for (const key in options) {
  merged[key] = options[key]
}

// After: Object spread
const merged = { ...defaults, ...options }
```

```ts
// Before: Manual array flattening
const flattened = []
for (const arr of arrays) {
  for (const item of arr) {
    flattened.push(item)
  }
}

// After: flat()
const flattened = arrays.flat()
```

```ts
// Before: Manual null checking
if (user && user.profile && user.profile.avatar) {
  return user.profile.avatar.url
}

// After: Optional chaining
return user?.profile?.avatar?.url
```

```ts
// Before: Manual default value
const name = user && user.name ? user.name : 'Anonymous'

// After: Nullish coalescing
const name = user?.name ?? 'Anonymous'
```

## Standard API Features

Use built-in APIs instead of custom implementations.

```ts
// Before: Custom FormData
const formData = new FormData()
formData.append('name', name)
formData.append('email', email)

// Built-in FormData is already there

// Before: Custom URL parsing
const params = url.split('?')[1].split('&').map(p => p.split('='))

// After: URLSearchParams
const params = new URLSearchParams(url.split('?')[1])
```

```ts
// Before: Custom deep clone
function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj))
}

// After: structuredClone (modern environments)
const cloned = structuredClone(obj)
```

## Environment Features

Use IDE and tooling capabilities.

### Rename Symbol (F2 in VS Code)

- Renames all occurrences
- Handles imports/exports
- Safe, automated

### Extract Function/Variable

- IDE can extract selected code
- Automatically determines parameters
- Updates all call sites

### Find All References

- Locate all usages
- Safe refactoring starting point

## Checklist

- [ ] Prettier/formatter applied
- [ ] Lint rules fixed (one per commit)
- [ ] Language features adopted
- [ ] Standard APIs used where appropriate
- [ ] IDE tooling utilized
