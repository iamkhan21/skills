# TypeScript Baseline (strict + DOM + aliases)

This doc describes settings to add, not a full `tsconfig.json` template.

## Required `tsconfig.json` settings

Add or ensure:

- `compilerOptions.strict: true`
- `compilerOptions.lib` includes DOM types:
  - `"DOM"`
  - `"DOM.Iterable"`
  - plus a modern ECMAScript lib such as `"ESNext"`

## Path aliases

Goal: use `@/…` imports for anything under `src/`.

In `tsconfig.json`:

- `compilerOptions.baseUrl: "."`
- `compilerOptions.paths`:
  - `"@/*": ["src/*"]`

## Make the alias work everywhere

When you add TS `paths`, also ensure runtime tooling resolves it:

- Bundler/dev server: configure its alias mapping to `src` (exact config depends on the scaffold).
- Vitest:
  - If tests run through Vite config, the alias usually applies automatically.
  - If a separate config is used, ensure the alias is present there too.

## Typecheck script

Add `package.json`:

- `typecheck`: `tsc --noEmit`

If the project uses project references or multiple `tsconfig.*.json` files, adapt the command to the repo's structure (e.g. `tsc -p tsconfig.app.json --noEmit`).
