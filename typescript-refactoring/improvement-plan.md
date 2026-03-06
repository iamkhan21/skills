# TypeScript Refactoring Skill Improvement Plan

This plan is for improving the whole `typescript-refactoring` skill and verifying that changes produce real gains rather than more elaborate prompt text.

## Current Assessment

The skill is strong on principles and reference material, but the top-level `SKILL.md` behaves more like an index than an operational guide.

Main risks:

- the skill routes to references well but does not always tell the model what to do first
- the output shape is underspecified, so refactor responses may vary too much
- the skill assumes ideal conditions too often: clear scope, tests available, commit-based workflow
- the skill does not explicitly defend against fake refactors such as abstraction churn or mixing refactor work with feature work

## Improvement Goals

We want the skill to do a better job of:

- identifying the real smell before changing structure
- choosing a safe first move
- preserving behavior under uncertainty
- keeping refactors proportional to the problem
- avoiding overengineering and scope creep
- producing more consistent, reviewable refactor responses

## Planned Changes

### 1. Strengthen the top-level skill shell

Revise `SKILL.md` so it does more than route into references.

Changes:

- improve the frontmatter description with real trigger language users actually type
- add a short "when this skill should take over" section
- add a concrete working pattern: diagnose -> define scope -> choose one strategy -> state safety net -> make incremental changes -> verify
- rephrase "one technique per commit" as "one conceptual transformation at a time" so it still applies when no commit is made

### 2. Add operational guidance for messy real-world cases

Add guidance for situations where the current skill is too idealized.

Changes:

- low-test codebase fallback: characterization tests, narrow regression tests, or explicit manual verification checkpoints
- vague requests: translate vague cleanup requests into a concrete target smell before changing code
- mixed requests: separate refactoring from feature work when both are requested together
- uncertain legacy behavior: document assumptions and avoid silently converting suspicious behavior into bug fixes

### 3. Make the symptom guide more actionable

Expand the symptom guide so each smell has a likely first move.

Examples:

- long function -> extract pure helper and add guard clauses
- hard to test -> isolate side effects behind a seam
- tangled modules -> identify dependency direction and extract a boundary
- duplication -> classify identical vs similar vs conceptual duplication before extracting
- complex types -> simplify contracts before inventing generic frameworks

### 4. Add explicit anti-overengineering guidance

The skill should say what not to do.

Add a section that warns against:

- renaming and redesigning at the same time unless necessary
- introducing abstractions before proving real duplication or coupling pressure
- widening types just to get code compiling
- doing architectural rewrites when a local seam would solve the problem
- presenting combined feature + refactor work as if it were a pure refactor

### 5. Add a response/output contract

The skill should guide the model toward a more consistent shape for refactor work.

Recommended output pattern:

1. Identify the dominant smell or pain point
2. State scope and non-goals
3. Explain the chosen refactoring move and why it fits
4. State the behavior-preservation plan
5. Make or describe incremental changes
6. Verify and mention follow-up risks or next steps

### 6. Add practical recipe references if needed

If the top-level shell improvements are not enough, add short recipe docs for recurring situations:

- extracting business logic from a React/TypeScript component
- isolating async side effects in service code
- reducing type complexity without losing safety
- untangling module boundaries with a safe first seam

Keep these recipes short and operational, not theoretical.

## Evaluation Strategy

The skill should not be judged by feel alone. Use A/B testing.

### Configurations

- `A`: current skill snapshot
- `B`: revised skill candidate
- `C`: optional no-skill baseline

Run `A` vs `B` for every iteration. Use `C` periodically to confirm the skill still beats default behavior.

### Eval Set

The eval prompts live in `typescript-refactoring/evals/evals.json`.

They cover:

- long functions with nested conditionals
- React/TypeScript components doing too much
- services mixing business logic and side effects
- type-heavy utilities with unsafe `any`
- duplication that may or may not justify abstraction
- legacy low-test code
- architecture and module-boundary problems
- vague cleanup requests
- mixed refactor + feature requests
- declarative pipeline decisions
- naming consistency and semantic clarity

### Assertions

The assertions are designed to catch fake improvement.

Examples of what they test:

- did the output identify the actual smell before changing code?
- did it preserve public boundaries unless change was justified?
- did it separate refactor work from feature work?
- did it choose a safe incremental move instead of a rewrite?
- did it preserve behavior under weak test coverage?
- did it avoid unnecessary abstractions?

### Blind A/B Review

Use `typescript-refactoring/evals/blind-comparison-rubric.md`.

For each prompt, compare the old and new outputs without labels and judge:

- problem fit
- scope control
- behavior safety
- refactor quality
- practicality
- maintainability

This is the best defense against fooling ourselves with nicer prose.

## Success Criteria

Treat a revision as a real improvement only if most of these are true:

- the revised skill wins blind comparison on at least 65 percent of evals
- assertion pass rate improves by at least 10 percentage points, or stays flat while clearly reducing overengineering
- token use and duration do not grow by more than 25 percent unless the quality win is also clear
- the revised skill performs at least as well on vague prompts and low-test legacy prompts
- the revised skill reduces abstraction churn, scope drift, and rewrite-heavy behavior

If the new skill is longer, more expensive, and not clearly better in blind review, treat that as failure.

## Iteration Order

Recommended order:

1. Revise only the top-level `SKILL.md`
2. Run A/B on the eval set
3. Analyze failures and regressions
4. Add recipe/reference improvements only if needed
5. Rerun A/B
6. Optimize the description for triggering after behavior is in good shape

## Key Question For Every Iteration

Do not ask only whether the output sounds smarter.

Ask:

- did it make a better refactoring decision?
- did it reduce risk?
- did it keep scope under control?
- did it avoid fake architectural sophistication?
- would a teammate actually prefer to receive this refactor?
