# Planning Workflow

Detailed process for planning a refactoring effort.

## Step 1: Gather Problem Description

Ask user for:

- Long, detailed description of the problem
- Pain points they're experiencing
- Any potential solution ideas

Example prompts:
- "What problem are you trying to solve?"
- "What's frustrating about the current code?"
- "What would ideal look like?"

## Step 2: Explore Codebase

Verify user's assertions and understand current state:

- Read relevant files
- Trace dependencies
- Identify coupling points
- Find existing patterns in codebase

Don't assume - verify with code.

## Step 3: Consider Options

Ask whether they considered alternatives. Present options:

```ts
// Option A: Extract to separate service
// Option B: Use composition pattern
// Option C: Introduce adapter layer
```

Trade-offs for each option should be discussed.

## Step 4: Interview About Implementation

Be extremely detailed and thorough:

- What modules will be affected?
- What interfaces will change?
- What's the data flow?
- What external dependencies exist?
- What's the rollback plan?

## Step 5: Define Exact Scope

Work out what will change and what won't:

### In Scope

- List specific modules, functions, files
- Define interface changes
- Identify test updates needed

### Out of Scope

- Explicitly list what won't change
- Document related but deferred items
- Note dependencies that stay unchanged

## Step 6: Check Test Coverage

Look for test coverage of affected areas:

- [ ] Unit tests exist for core logic
- [ ] Integration tests cover module boundaries
- [ ] Edge cases tested

If insufficient coverage:

1. Ask user about testing plans
2. Recommend adding characterization tests first
3. Consider test-first refactoring approach

## Step 7: Break Into Tiny Commits

Martin Fowler's advice: "Make each refactoring step as small as possible, so that you can always see the program working."

### Commit Planning Example

```markdown
## Commits

1. Add characterization tests for UserService
2. Extract validateEmail to separate function
3. Move validateEmail to validation module
4. Add stricter types to UserService
5. Extract UserRepository interface
6. Implement UserRepository with current logic
7. Inject UserRepository into UserService
8. Add in-memory UserRepository for tests
9. Remove direct database calls from UserService
```

Each commit:

- Leaves codebase in working state
- Tests pass
- One logical change
- Easy to revert if needed

## Planning Output

After planning, you should have:

1. Problem statement
2. Proposed solution
3. List of tiny commits
4. Scope boundaries (in/out)
5. Testing decisions
6. Key implementation decisions

See [19-plan-template.md](19-plan-template.md) for documenting the plan.
