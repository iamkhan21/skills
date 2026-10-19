# Skills

This repository is a small library of reusable, task-focused "skills".
Each skill lives in its own folder and includes:

- `SKILL.md`: the playbook / prompt for how to execute the task
- `references/`: supporting notes, checklists, and cheat sheets

## Skills in this repo

- [frontend-init](frontend-init/SKILL.md): configure missing tooling and conventions in a scaffolded TypeScript frontend without replacing a working setup
- [modern-ui-page](modern-ui-page/SKILL.md): build or polish a page with intentional visual direction, accessible interactions, and responsive behavior
- [react-testing](react-testing/SKILL.md): write and debug React tests with Testing Library, including async UI, providers, and mocking boundaries
- [typescript-refactoring](typescript-refactoring/SKILL.md): simplify TypeScript or JavaScript while preserving behavior through small, verified changes

## Using a skill

Load the relevant `SKILL.md` in your agent, or point the agent to it with your task. Read linked references only as needed. Installation and discovery depend on the host; these playbooks do not require a particular model or agent framework.

## Adding a new skill

1. Create a new folder at the repo root (e.g. `my-skill/`).
2. Add a `SKILL.md` that explains:
   - goal and non-goals
   - workflow / steps
   - default choices and when to ask for clarification
   - links to any `references/` docs
3. Put supporting material in `my-skill/references/` as Markdown.
