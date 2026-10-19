# Tooling Baseline (pnpm + Vitest + Oxc)

Use these defaults only for missing tooling in an already-scaffolded project. Keep existing tools, package manager, scripts, and version constraints unless migration is requested. Commands below assume pnpm has been selected.

## pnpm baseline

- Use pnpm when the repo selects it, or as the default when no package manager exists.
- Preserve the existing version range policy. If exact versions are requested, merge this into `.npmrc` rather than replacing the file:

  - Add `.npmrc`:

    ```
    save-exact=true
    ```

## Install dev dependencies

For missing tools, install only the needed packages:
- `vitest`
- `jsdom` or `happy-dom` for DOM tests (choose one supported by the app)
- `oxlint`
- `oxfmt`

```sh
pnpm add -D vitest jsdom oxlint oxfmt
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
  - Use the config format supported by the installed version; avoid competing configs.

- Keep framework-specific ESLint checks that Oxlint does not replace. Tool migration is separate work.

## Vitest config

Extend the existing test config. If adding a separate `vitest.config.ts`, it takes precedence over the Vite config; explicitly merge needed aliases and plugins with `mergeConfig` rather than assuming inheritance. A standalone test config can start with:

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom', // use node for non-DOM tests; happy-dom is another option
  },
})
```

- Install the selected environment package. Use browser tests for real layout or browser behavior.
- `globals: true` makes `describe`, `it`, `expect` available without imports.
  If enabled, add `"vitest/globals"` to the test config's existing `compilerOptions.types` without dropping other entries. Keep test globals out of production configs when possible. If globals are disabled, import runner APIs and explicitly register RTL cleanup in the setup file.
- Co-locate tests using the repo's conventions; React tests containing JSX use `.test.tsx` / `.spec.tsx`.
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

Keep the existing UI/CSS approach. Ask about a new library only when it is part of the requested setup.

Then adjust only what is necessary:
- Tailwind/utility CSS: follow the installed version's framework integration; do not assume every version needs PostCSS configuration.
- Component libraries (framework-specific): add them only after confirming framework (React/Vue/etc).

## Optional: @testing-library

If the project uses React, Vue, or Svelte, install the matching testing-library:

```sh
# React example
pnpm add -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

Then wire up the setup file as described in the Vitest config section above.

## Verify

Run a representative test and the project's typecheck, lint, format check, and build. Verify scripts use installed tools and do not silently pass an empty suite. Consult the installed versions' official docs for config syntax and compatibility before adopting examples.
