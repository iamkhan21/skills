## Purpose

This repository is a library of reusable prompt-based skills. Each top-level skill directory contains a `SKILL.md` playbook plus optional supporting material in `references/`.

## Repository Shape

- `frontend-init/` — baseline conventions for scaffolded TypeScript frontend apps
- `modern-ui-page/` — page-level UI implementation guidance
- `react-testing/` — React Testing Library guidance and patterns
- `typescript-refactoring/` — refactoring workflow and reference material
- `README.md` — short repo overview

Ignore local tool state such as `.factory/`, `.claude/`, `.opencode/`, `.idea/`, and other editor/runtime artifacts unless the task explicitly targets them.

## Skill Folder Contract

Each skill should follow the existing layout:

```text
skill-name/
  SKILL.md
  references/
```

Keep skill folders minimal unless the task explicitly requires more structure.

## `SKILL.md` Conventions

Match the established format used in this repo:

1. YAML frontmatter with at least:
   - `name`
   - `description`
   - optional trigger metadata when useful
2. A clear H1 title matching the skill name
3. Practical sections such as:
   - quick start
   - workflow
   - guardrails / scope
   - verification
   - links to `references/`

Descriptions should be operational and trigger-oriented. Write them the way a routing system or agent can use directly, not as marketing copy.

## Prompt Quality

- Give each skill a clear job and exclusions so related skills do not all trigger on generic keywords.
- Keep the main playbook short; link to references with guidance on when to read them. Do not require loading every reference.
- Prefer conditional defaults over universal tool choices, mandatory interviews, response templates, or approval gates for routine work.
- Preserve project conventions and user authorization. Examples do not authorize installing tools, committing, publishing, or editing global configuration.
- Check version-sensitive advice against official documentation. Avoid model-release claims and unverified performance or quality promises.
- Check that examples preserve the behavior they claim to preserve, including edge cases and error paths.

## Writing Style

Follow the tone already present in the repo:

- concise, direct, and action-oriented
- prefer short sections and bullet lists
- emphasize scope, guardrails, and verification
- prefer practical defaults over theory
- use Markdown headings consistently
- keep examples minimal and relevant

## Reference Material

When adding `references/*.md` files:

- keep each file focused on one topic
- use imperative, scannable headings
- favor checklists, heuristics, and examples over long prose
- link from `SKILL.md` to the relevant references

## Editing Rules

- Preserve the existing structure unless the task requires a broader reorganization.
- Prefer updating the relevant skill folder over adding repo-wide docs.
- Do not rename skills casually; top-level folder names are part of the repo interface.
- Keep changes narrow and consistent with surrounding files.
- Do not add new dependencies or tooling unless the task explicitly requires it.

## Verification

For this repository, verification is usually structural and editorial rather than build-based:

- ensure new files are placed in the correct skill directory
- confirm links and referenced filenames are valid
- check that headings, frontmatter, and file names match repo conventions
- review realistic trigger and near-miss requests; distinguish editorial/static checks from actual agent evaluations
- if a task changes code or tooling, then run the relevant project validators

## Non-Goals

- Do not turn this repo into a general application codebase.
- Do not introduce broad process documentation unless requested.
- Do not add boilerplate files that are not used by a specific skill.
