# Process Heuristics

Use these heuristics while refactoring, not just when planning it.

## Prefer Small, Reversible Moves

Each step should be easy to review and easy to back out.

```bash
# Too big
git commit -m "refactor: rewrite user module"

# Better
git commit -m "refactor: extract validateEmail helper"
git commit -m "refactor: replace inline validation with helper"
git commit -m "test: add characterization coverage for validation"
```

Small steps make it easier to spot regressions, understand intent, and stop early when the main pain point is gone.

## One Conceptual Transformation At A Time

Do not mix unrelated refactor moves unless keeping them together clearly lowers risk.

- rename or repartition
- extract logic or change types
- isolate side effects or redesign a boundary

The goal is not perfect purity. The goal is understandable progress.

## Keep Behavior Stable

During refactoring, avoid quietly changing product behavior.

- No feature additions unless the user asked for them
- No bug fixes hidden inside cleanup
- No changed return contracts, ordering, or error behavior unless called out

If you find a real bug, note it and fix it separately unless the user asked for both.

## Verify After Meaningful Steps

After each meaningful change:

1. Run the most relevant checks.
2. Compare against the preserved behavior surface.
3. Stop and investigate if anything unexpected changes.

```bash
npm test -- --related
npm run typecheck
npm run build
```

If automated tests are weak, keep a short manual verification list and reuse it after each risky step.

## Keep Test Refactors Separate When Practical

Prefer this order:

1. Add or adjust tests that protect behavior.
2. Refactor production code.
3. Clean up test structure if it still needs work.

Mixing all three at once makes review harder.

## Stay Local Before Going Architectural

Start with the smallest seam that buys safety.

- local helper before shared utility
- explicit parameter before new service layer
- one extracted boundary before full module redesign

If the first useful step is already architectural, explain why a local move is not enough.

## Mixed Refactor + Feature Work

If the task includes both cleanup and new behavior:

- package them as separate phases, commits, or review sections
- keep the first phase structural and local
- avoid speculative target architecture language unless it is necessary

Combined work can be valid. Just do not disguise it as a pure refactor.

## When To Stop

Stop when:

- the original pain point is materially reduced
- the next step would mostly be polish
- risk starts rising faster than payoff
- you are about to broaden scope without a clear reason

New problems discovered during refactoring usually belong in follow-up work, not in the current change.

## Rollout Tactics For Risky Boundaries

For refactors near production-critical seams, use temporary safety mechanisms when justified.

- feature flag
- adapter or facade that preserves the old contract
- dual-read or compare mode for critical migrations

Only add these when rollout risk warrants the extra complexity.

## Pull Request Guidance

Good refactor PRs are small enough to review and explicit about safety.

```markdown
## Why
[dominant smell and why it hurts]

## Scope
[what changed and what did not]

## How
[first move, sequence, and key decisions]

## Verification
[tests, manual checks, or both]
```

## Checklist

- [ ] The main smell stayed in focus
- [ ] Each step is understandable and reversible
- [ ] Feature work is separate or clearly labeled
- [ ] Verification matches the actual risk level
- [ ] The refactor stopped once the main pain point was addressed
