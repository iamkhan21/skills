# TypeScript Refactoring Eval Plan

This eval set is designed to answer one question: does a revised version of the `typescript-refactoring` skill produce materially better refactoring behavior than the current version, or are we just making the skill longer and more self-important?

## Configurations

- `A` - current skill snapshot (`old_skill`)
- `B` - revised skill under test (`with_skill`)
- `C` - optional no-skill baseline (`without_skill`)

Use `A` vs `B` on every iteration. Add `C` for the first broad comparison and any later spot checks where you want to verify the skill still beats the default model behavior.

## Eval Coverage

The prompts in `evals.json` cover the refactor shapes this skill claims to handle:

- long functions and conditional complexity
- React/TypeScript component decomposition
- mixed side effects and low cohesion in services
- type-system complexity and unsafe `any`
- duplication that may or may not justify abstraction
- low-test legacy code
- module-boundary and architecture pressure
- vague cleanup requests that need scope discipline
- mixed refactor + feature requests
- naming and semantic clarity

If a proposed skill revision only looks better on one of these buckets, it is probably overfit.

## Test Protocol

1. Snapshot the current skill as configuration `A`.
2. Apply the proposed improvements to create configuration `B`.
3. For each eval prompt, run both configurations in the same batch.
4. Save artifacts per eval under separate config directories.
5. Grade assertions for each run.
6. Run a blind comparison between `A` and `B` outputs.
7. Aggregate assertion scores, blind wins, token use, duration, and churn notes.

## What Counts As Real Improvement

Treat `B` as genuinely better only if most of these are true:

- `B` wins the blind comparison on at least 65 percent of evals.
- `B` improves the assertion pass rate by at least 10 percentage points, or maintains pass rate while clearly reducing overengineering.
- `B` does not increase token use or duration by more than 25 percent unless the blind-comparison win rate also improves.
- `B` handles low-test and vague-scope prompts at least as well as `A`.
- `B` reduces fake-refactor behavior: abstraction churn, scope drift, or rewriting instead of sequencing.

If the new skill is more verbose, more expensive, and not clearly better in blind review, call it noise.

## Anti-Bullshit Checks

Flag the iteration as suspect if you see any of these patterns:

- the model talks at length about refactoring principles but edits code shallowly
- assertion gains come mostly from easy prompts while hard prompts stay flat
- the revised skill introduces more interfaces, layers, or files without clearer boundaries
- the output explains discipline well but still mixes refactors with feature work
- most "improvements" are naming, formatting, or file moves with little reduction in cognitive load

## Suggested Scoring Summary

For each configuration, report:

- assertion pass rate
- blind-comparison wins
- mean tokens and duration
- evaluator notes on overengineering, scope control, and safety

For each iteration, answer explicitly:

1. What got better?
2. What got worse?
3. Did we reduce real refactor risk, or just write a more elaborate prompt?
4. Should the next iteration target the top-level `SKILL.md`, the reference docs, or both?
