# TypeScript Baseline (strict + DOM + aliases)

This doc describes settings to add, not a full `tsconfig.json` template.

## App `tsconfig.json` settings

Add or ensure:

- `compilerOptions.strict: true`
- For browser code, `compilerOptions.lib` includes DOM types:
  - `"DOM"`
  - `"DOM.Iterable"`
  - plus the ECMAScript library appropriate to the supported runtime

Preserve scaffold settings for `jsx`, `module`, `moduleResolution`, targets, and framework plugins. Do not add DOM globals to server-only code. Raising `lib` does not polyfill APIs in older browsers.

## Path aliases

If an alias is useful or requested, use `@/…` for imports under `src/`; preserve an existing alias convention instead of adding a second one.

In `tsconfig.json`:

- `compilerOptions.paths`:
  - `"@/*": ["./src/*"]`

`paths` does not require `baseUrl`; do not add it just for aliases. Without `baseUrl`, targets are relative to the config that declares them. If an existing config sets `baseUrl`, mappings resolve relative to it; adapt the example accordingly and adjust paths explicitly if removing it. TypeScript path mappings do not rewrite emitted imports.

## Make the alias work everywhere

When you add TS `paths`, also ensure runtime tooling resolves it:

- Bundler/dev server: configure its alias mapping to `src` (exact config depends on the scaffold).
- Vitest:
  - When Vitest uses the Vite config, it shares configured aliases.
  - A separate Vitest config takes precedence; merge or configure needed aliases and plugins explicitly.

## Typecheck script

Add `package.json`:

- `typecheck`: `tsc --noEmit`

If the project uses project references or multiple `tsconfig.*.json` files, adapt the command to the repo's structure (e.g. `tsc -p tsconfig.app.json --noEmit`).

A solution-style root config may include no source files, so bare `tsc --noEmit` can miss app errors. Preserve a working scaffold check (including build mode when needed) or check each relevant project explicitly. Confirm the selected command actually checks app code.
