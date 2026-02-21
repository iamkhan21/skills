# Skills

This repository is a small library of reusable, task-focused "skills".
Each skill lives in its own folder and includes:

- `SKILL.md`: the playbook / prompt for how to execute the task
- `references/`: supporting notes, checklists, and cheat sheets

## Skills in this repo

- `frontend-init/`: baseline a scaffolded TypeScript frontend (tooling, strict TS, scripts, aliases, basic folder structure)
- `react-testing/`: write and debug React tests (Testing Library patterns, queries, utilities, mocking boundaries)
- `refactoring/`: systematic refactoring workflow and technique catalog (small steps, guardrails, templates)

## Adding a new skill

1. Create a new folder at the repo root (e.g. `my-skill/`).
2. Add a `SKILL.md` that explains:
   - goal and non-goals
   - workflow / steps
   - default choices and when to ask for clarification
   - links to any `references/` docs
3. Put supporting material in `my-skill/references/` as Markdown.
