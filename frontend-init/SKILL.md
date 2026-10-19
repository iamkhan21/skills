---
name: frontend-init
description: Configure an already-scaffolded TypeScript frontend's tooling and conventions. Use for initial setup or alignment of package scripts, strict TypeScript, path aliases, Vitest, Oxc, and lightweight feature folders. Not for generating a new app or replacing an established toolchain without a migration request.
---

# Frontend Init

## Quick start

1. Inspect project instructions, package scripts, lockfile, framework, and configs.
2. Identify missing setup; preserve working conventions and dependencies.
3. For an unopinionated scaffold, default to pnpm, strict TypeScript, Vitest, and Oxc. State these defaults before applying them.
4. Configure only what the requested baseline needs, then verify it.

## Workflow

### 1. Establish scope

- Confirm the app is already scaffolded and check supported Node/tool versions.
- Use the package manager declared by the repo; do not generate a second lockfile.
- Ask only about decisions that affect the requested setup and cannot be inferred. A tooling-only task does not need a UI-library interview.
- Keep the existing CSS/UI approach. Ask before introducing a new one.

### 2. Fill tooling gaps

- Reuse a working test runner, linter, and formatter rather than installing alternatives alongside them.
- If missing, add the defaults and scripts for typecheck, test, lint, and format. Preserve existing script names where practical.
- Check the installed version's documentation before using version-sensitive config or CLI options.
- See [tooling](references/tooling.md) for setup examples.

### 3. Align TypeScript and imports

- Enable strict checking in the app config; assess existing errors before changing an established project.
- Preserve framework compiler settings and separate browser, server, and test environments.
- Add `@/*` only if useful or requested; verify both TypeScript and runtime resolution.
- See [TypeScript](references/typescript.md).

### 4. Keep structure proportional

- Preserve framework-owned routing and directory conventions.
- Use shallow feature folders when code needs a home; do not create empty layers or reorganize working code for symmetry.
- See [structure](references/structure.md).

## Guardrails

- Do not scaffold, migrate frameworks, upgrade unrelated dependencies, or replace existing tools as incidental setup.
- Do not overwrite config wholesale or remove framework-specific lint rules without an equivalent check.
- Keep tool preferences as defaults, not requirements that override project instructions.

## Verification

- Run the configured typecheck, non-watch tests, lint, format check, and build where available.
- If adding a test runner, run a small representative test; an empty suite does not prove the setup works.
- Verify an alias import through the build and test runner if aliases changed.
- Report changes, actual check results, and anything not verified.
