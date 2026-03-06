# Planning Workflow

Use this workflow when the refactor is large enough to benefit from an explicit plan.

## Step 1: Translate The Request Into A Refactor Problem

Start by naming the actual problem in code terms.

- What is painful right now?
- What smell is dominant?
- What should stay behaviorally stable?
- Is this really a refactor, or is feature work mixed in?

Examples:

- "clean this up" -> long function with mixed responsibilities
- "make this easier to test" -> side effects block isolated tests
- "split this component" -> file is doing too much, not necessarily a design-system rewrite

## Step 2: Verify In Code

Do not trust the first narrative blindly.

- Read the relevant files
- Trace imports and dependencies
- Identify entry points and public contracts
- Look for existing patterns worth matching
- Confirm whether the suspected smell is real

## Step 3: Choose The Safest Useful Direction

Prefer a small first move over a grand design.

Common choices:

- Extract a pure helper
- Add guard clauses
- Isolate side effects behind a seam
- Rename to reveal responsibility
- Split one module boundary
- Add characterization tests first

If more than one approach is plausible, compare them in terms of risk, payoff, and reversibility.

## Step 4: Define Scope And Non-Goals

Write down exactly what changes and what does not.

### In Scope

- Specific files, modules, or functions
- The boundary being preserved or adjusted
- Tests or verification that must be updated

### Out Of Scope

- Related cleanup that can wait
- Product behavior changes
- Architectural redesign beyond the first seam

For vague requests, keep this section especially tight.

## Step 5: Decide The Safety Net

Choose a safety net that matches reality.

- Good tests: run and extend targeted tests.
- Weak tests: add characterization coverage at the boundary.
- No tests: define manual verification plus compiler, type, and lint checks.

For legacy code, list the behavior surface explicitly: inputs, outputs, side effects, ordering, fallbacks, and errors.

## Step 6: Break The Work Into Small Transformations

Plan a sequence of reversible moves.

```markdown
## Phases

1. Add characterization tests for UserService behavior surface
2. Extract `validateEmail` as a local pure helper
3. Replace inline validation with helper calls
4. Move helper to validation module if duplication still exists
5. Introduce `UserRepository` seam around direct database access
6. Update tests around the new seam
```

Each step should:

- Leave the codebase working
- Preserve behavior
- Be reviewable on its own
- Avoid mixing unrelated techniques unless there is a strong reason

## Step 7: Package Mixed Requests Honestly

If the user asked for a refactor plus a new feature or bug fix:

- separate the structural work from the behavior change
- prefer refactor first, then feature
- if they must land together, describe them as distinct phases or review sections
- keep phase one small and local

Do not frame combined work as a pure refactor.

## Planning Quality Checks

- **Problem fit**: names the dominant smell, not just a generic desire to clean up
- **Scope control**: preserves boundaries and lists non-goals
- **Safety**: uses a realistic safety net for the test situation
- **Proportionality**: first move is small, local, and reversible
- **Honesty**: separates refactor work from feature work

## Planning Output

After planning, you should have:

1. Dominant smell or pain point
2. Scope and non-goals
3. Chosen first move and why
4. Safety net and verification plan
5. Sequence of small transformations or phases
6. Assumptions, unknowns, and deferred work

Document the plan with [19-plan-template.md](19-plan-template.md).
