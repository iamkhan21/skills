# Low-Hanging Fruit

Use local mechanical improvements when they solve the actual problem. Newer syntax is not automatically clearer or behavior-preserving.

## Code Formatting & Linting

- Use the repo's installed formatter, linter, and scripts; do not install a preferred replacement during a refactor.
- Limit automatic fixes to relevant files and inspect the diff. A correctness fix can change behavior.
- Keep broad formatting separate from structural work when it would obscure review.
- Create commits only when authorized; one commit per lint rule is not required.

## Language Features

Prefer native features when their semantics and runtime support match the existing contract.

### Examples (JavaScript/TypeScript)

```ts
// Before: Plain data records, own enumerable string properties, no '__proto__' key
const merged = {}
for (const key of Object.keys(defaults)) {
  merged[key] = defaults[key]
}
for (const key of Object.keys(options)) {
  merged[key] = options[key]
}

// After: Object spread
const merged = { ...defaults, ...options }
```

```ts
// Before: Dense arrays (no holes)
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
// Before: Nullish checks, no truthiness-based fallback
return user == null || user.profile == null || user.profile.avatar == null
  ? undefined
  : user.profile.avatar.url

// After: Optional chaining
return user?.profile?.avatar?.url
```

```ts
// Before: Manual default value
const name = user && user.name ? user.name : 'Anonymous'

// After: Preserve the original fallback for an empty name
const name = user?.name || 'Anonymous'
```

Check edge cases before replacing syntax:

| Change | Potential behavior difference |
|--------|-------------------------------|
| `||` → `??` | Empty strings, `0`, `false`, and `NaN` stop taking the fallback |
| Assignment → object spread | Inherited properties, symbols, accessors, and `__proto__` handling can differ |
| Nested loops → `flat()` | Sparse array holes may be removed instead of becoming `undefined` |
| Truthy guards → optional chaining | `?.` stops only on `null` / `undefined`, not other falsy values |
| Mutating → copying | Caller-visible identity, aliases, and mutation contracts change |

## Standard API Features

Use built-in APIs instead of custom implementations.

```ts
// Use built-in FormData when its encoding matches the request contract
const formData = new FormData()
formData.append('name', name)
formData.append('email', email)

// Parse a URL with an explicit base when relative URLs are allowed
const params = new URL(url, baseUrl).searchParams
```

`URLSearchParams` decodes escapes and treats `+` as a space; its representation is not a drop-in replacement for a raw array of string pairs. Check duplicate keys, malformed input, and encoding expectations.

```ts
// JSON-based clone: existing behavior may be intentional
function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj))
}

// Alternative, NOT an equivalent refactor
const cloned = structuredClone(obj)
```

`structuredClone` preserves values JSON loses (such as `Date` and `undefined`) and rejects some values, including functions. JSON serialization may call `toJSON`. Treat a switch as a contract change unless the supported data shapes make them equivalent.

## Environment Features

Use IDE and tooling capabilities.

### Rename Symbol (F2 in VS Code)

- Renames all occurrences
- Handles imports/exports
- Review references and public/string-based names; symbol renaming cannot update every external consumer automatically

### Extract Function/Variable

- IDE can extract selected code
- Automatically determines parameters
- Updates all call sites

### Find All References

- Locate all usages
- Safe refactoring starting point

## Checklist

- [ ] Existing tools reused; unrelated formatting avoided
- [ ] Automatic fixes reviewed for semantic changes
- [ ] Falsy/nullish values, sparse data, errors, identity, and mutation checked where relevant
- [ ] Runtime support and public consumers verified
