# Tooling Baseline (pnpm + Vitest + Oxc)

This skill assumes the project is already scaffolded. It adds a baseline for testing, linting, and formatting.

## pnpm baseline

- Prefer pnpm for all installs.
- Make exact versions the default (so the agent doesn't need `--exact` everywhere):

  - Add `.npmrc`:

    ```
    save-exact=true
    ```

## Install dev dependencies

Install:
- `vitest`
- `happy-dom` (lightweight DOM environment for tests)
- `oxlint`
- `oxfmt`

```sh
pnpm add -D vitest happy-dom oxlint oxfmt
```

## Add `package.json` scripts

Add (or align with existing scripts):

- `typecheck`: `tsc --noEmit`
- `test`: `vitest`
- `test:run`: `vitest run`
- `lint`: `oxlint`
- `lint:fix`: `oxlint --fix`
- `fmt`: `oxfmt` (Oxc convention; `format` also fine)
- `fmt:check`: `oxfmt --check`

Notes:
- Prefer keeping scripts stable across projects; adapt names only if the repo already has conventions.
- If the project does not have TypeScript yet, add it before enabling `typecheck`.

## Oxlint config

- Generate a starter config in the project root:

```sh
pnpm exec oxlint --init
```

- Config discovery:
  - Use either `.oxlintrc.json` or `oxlint.config.ts` (not both).

## Vitest config

Create a `vitest.config.ts` (or add to the existing Vite config):

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'happy-dom', // lighter and faster than jsdom
  },
})
```

- Install `happy-dom` as a dev dep: `pnpm add -D happy-dom`
- `globals: true` makes `describe`, `it`, `expect` available without imports.
  If enabled, add `"types": ["vitest/globals"]` to `tsconfig.json` `compilerOptions` for type support.
- Start with file-based tests: `**/*.test.ts` / `**/*.spec.ts`.
- For files that need a different environment, use the per-file override docblock:

  ```ts
  // @vitest-environment node
  ```

- If the project uses `@testing-library`, add a setup file:

  ```ts
  // vitest.config.ts
  setupFiles: ['./src/test/setup.ts'],
  ```

  ```ts
  // src/test/setup.ts
  import '@testing-library/jest-dom/vitest'
  ```

## Optional: UI/CSS choice impacts tooling

Ask: "Do you want a UI/component library or CSS framework? If yes, which one?"

Then adjust only what is necessary:
- Tailwind/utility CSS: add its PostCSS tooling and ensure formatting/linting ignores generated files.
- Component libraries (framework-specific): add them only after confirming framework (React/Vue/etc).

## Optional: @testing-library

If the project uses React, Vue, or Svelte, install the matching testing-library:

```sh
# React example
pnpm add -D @testing-library/react @testing-library/jest-dom
```

Then wire up the setup file as described in the Vitest config section above.
