# Progressive FSD-Style Structure (Minimal Wrapping)

Goal: get the benefits of FSD-ish boundaries without deep nesting or premature layers.

## Default (day 1)

For a new, unopinionated source tree, consider these homes as code appears:

```
src/
  app/
  features/
  shared/
```

### What goes where

Do not create empty folders or move working code just to match this tree. Framework routing directories take precedence; this is an optional lightweight convention, not a required FSD migration.

- `src/app/`: app bootstrap, providers, routing, global styles, app-level config.
- `src/shared/`: shared UI primitives, shared libs/utils, shared API clients, shared config.
- `src/features/`: user-facing feature slices (each feature owns its UI + logic).

## Add layers only when needed

Introduce these when there is real pressure:

- `src/entities/`: reusable domain objects/models used by multiple features.
- `src/pages/`: route-level composition when routes become non-trivial.
- `src/widgets/`: reusable composed blocks that sit between features/pages.

## Minimal wrapping rules

- Keep folder depth shallow.
  - Prefer `shared/ui/button/` over `shared/ui/button/button/`.
- Co-locate code.
  - Keep component + styles + tests in the same folder.
- Export intentionally.
  - Use local `index.ts` only at boundaries you actually import from.
- Avoid global barrels.
  - Do not create a single `src/index.ts` that re-exports everything.

## Example feature slice

```
src/features/auth/
  ui/
    login-form.tsx
    login-form.test.tsx
  model/
    use-login.ts
  api/
    login.ts
  index.ts
```

Adapt file extensions to the project framework (React/Vue/etc). The layering idea stays the same.
