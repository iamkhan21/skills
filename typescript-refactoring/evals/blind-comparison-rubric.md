# Blind Comparison Rubric

Use this rubric when comparing two outputs for the same eval prompt without revealing which output came from which skill version.

## Instructions For The Reviewer

- Read the prompt first.
- Review output `A` and output `B` without knowing which is the old skill or new skill.
- Judge the outputs based on refactoring quality, not writing style alone.
- If one output is more verbose, do not reward it unless the extra reasoning clearly improves decisions.
- If both outputs are weak, still pick the less harmful one and explain why.

## Questions To Answer

For each pair, answer these questions:

1. Which output better identified the real code smell or structural problem?
2. Which output chose a safer first move?
3. Which output did a better job preserving behavior or planning how to preserve it?
4. Which output was less likely to overengineer the solution?
5. Which output would leave the codebase easier for a teammate to change next month?

## Scoring Dimensions

Score each output from 1 to 5 on these dimensions:

- `problem_fit` - Did it target the actual problem instead of generic cleanup?
- `scope_control` - Did it define boundaries and avoid scope creep?
- `behavior_safety` - Did it preserve behavior or use a credible safety net?
- `refactor_quality` - Did the proposed or implemented moves reduce complexity, coupling, or confusion?
- `practicality` - Did it avoid unnecessary patterns, layers, or abstractions?
- `maintainability` - Would the result be easier to test, reason about, and extend?

## Tie-Break Rules

If scores are close, prefer the output that:

- makes a smaller but clearly justified change
- acknowledges uncertainty instead of bluffing
- avoids mixing feature work with refactoring
- leaves a reversible next step instead of demanding a rewrite

## Winner Format

Record results in this shape:

```json
{
  "winner": "A",
  "reason": "A isolates side effects without inventing unnecessary layers, while B adds abstractions that are not yet justified.",
  "scores": {
    "A": {
      "problem_fit": 5,
      "scope_control": 4,
      "behavior_safety": 4,
      "refactor_quality": 4,
      "practicality": 5,
      "maintainability": 4
    },
    "B": {
      "problem_fit": 4,
      "scope_control": 3,
      "behavior_safety": 4,
      "refactor_quality": 3,
      "practicality": 2,
      "maintainability": 3
    }
  }
}
```

## Interpretation Guide

- If the revised skill wins mainly on style but not on `problem_fit`, `practicality`, or `behavior_safety`, do not count that as meaningful improvement.
- If the revised skill loses because it overexplains, overabstracts, or widens scope, simplify the skill.
- If both outputs fail in similar ways, the issue is probably missing operational guidance in the skill rather than an isolated bad prompt.
