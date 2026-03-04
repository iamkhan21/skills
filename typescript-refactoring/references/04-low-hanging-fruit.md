# Low-Hanging Fruit

Quick wins that improve code with minimal risk. Start here before deeper refactoring.

## Code Formatting & Linting

Use Biome for both formatting and linting in one tool. It's fast (written in Rust) and replaces Prettier + ESLint.

```bash
# One-time setup
npm install --save-dev --exact @biomejs/biome
npx @biomejs/biome init

# Format and lint all files
npx @biomejs/biome check --write .
```

Alternatively, use OxLint for linting (fast, zero-config) with a separate formatter:

```bash
npx oxlint .
```

### Commit Strategy

```bash
git commit -m "style: apply Biome formatting"
```

One commit for formatting. Don't mix with other changes.

### Incremental Lint Fixing

Fix each lint rule as separate commit.

```bash
# Fix one category at a time
npx @biomejs/biome check --write --only=style src/
git commit -m "style: fix style lint rules"

npx @biomejs/biome check --write --only=correctness src/
git commit -m "refactor: fix correctness lint rules"
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

- [ ] Biome formatting applied
- [ ] Lint rules fixed (one category per commit)
- [ ] Language features adopted
- [ ] Standard APIs used where appropriate
- [ ] IDE tooling utilized
