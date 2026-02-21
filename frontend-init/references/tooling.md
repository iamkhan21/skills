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
- `oxlint`
- `oxfmt`

Example:

```sh
pnpm add -D vitest oxlint oxfmt
```

## Add `package.json` scripts

Add (or align with existing scripts):

- `typecheck`: `tsc --noEmit`
- `test`: `vitest`
- `test:run`: `vitest run`
- `lint`: `oxlint`
- `lint:fix`: `oxlint --fix`
- `format`: `oxfmt`
- `format:check`: `oxfmt --check`

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

## Vitest defaults

- Start with file-based tests: `**/*.test.ts` / `**/*.spec.ts`.
- If a browser-like environment is needed, set `environment: "jsdom"` and add `jsdom` as a dev dependency.

## Optional: UI/CSS choice impacts tooling

Ask: "Do you want a UI/component library or CSS framework? If yes, which one?"

Then adjust only what is necessary:
- Tailwind/utility CSS: add its postcss tooling and ensure formatting/linting ignores generated files.
- Component libraries (framework-specific): add them only after confirming framework (React/Vue/etc).
