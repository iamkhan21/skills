# Refactor Plan Template

Template for documenting refactoring plans in GitHub issues or PRs.

## Template

```markdown
## Dominant Smell

[What kind of refactor problem is this really? Long function, mixed responsibilities, duplication, side effects, type pressure, module boundary issue, legacy behavior uncertainty, etc.]

## Problem Statement

[The problem being solved, from the developer's perspective. What pain points exist? What would ideal look like?]

## Scope

- In scope: [specific files, modules, functions, seams]
- Out of scope: [related but deferred items]
- Boundary to preserve: [public APIs, contracts, behavior that must remain stable]

## Solution

[The proposed solution. What is the first safe move? Why this approach over alternatives?]

## Safety Net

- Existing tests to rely on: [list]
- Characterization tests to add: [list or n/a]
- Manual verification checkpoints: [list or n/a]
- Compiler, type, lint checks: [list]

## Behavior Surface To Preserve

- Inputs and outputs: [list]
- Side effects: [list]
- Ordering and timing expectations: [list]
- Fallbacks and defaults: [list]
- Errors and edge cases: [list]

## Phases Or Commits

[Detailed implementation plan broken into small, reversible steps. Each step should leave the codebase in a working state.]

1. [Step description]
2. [Step description]
3. [Step description]
...

## Decision Document

- Modules to be built or modified: [list]
- Interfaces that will change: [list]
- Technical clarifications: [any clarifications]
- Architectural decisions: [decisions]
- Schema changes: [if any]
- API contracts: [if any]

## Testing Decisions

- Test philosophy: [for example: test external behavior, not implementation details]
- Modules to test: [list]
- Prior art: [similar existing tests to reference]

## Mixed Work Packaging

[If feature work or bug fixes are included, explain how they are separated from the refactor. Otherwise state that this is refactor-only work.]

## Out of Scope

- [Item 1]
- [Item 2]

## Further Notes

- Assumptions: [list]
- Unknowns and risks: [list]
- Follow-up work: [list]
```

## Example

```markdown
## Dominant Smell

Mixed responsibilities and side effects in a service that has become hard to test and risky to change.

## Problem Statement

The UserService has grown to handle multiple responsibilities: user CRUD, email notifications, and analytics tracking. This makes it hard to test, hard to modify, and changes in one area risk breaking others. Developers avoid touching it because it's unclear what effects their changes might have.

## Scope

- In scope: UserService, the notification seam, analytics seam, related tests
- Out of scope: email template content, analytics event definitions, API contract changes
- Boundary to preserve: existing UserService public methods and externally visible behavior

## Solution

Start by protecting the current UserService behavior with characterization tests, then extract the notification and analytics seams one at a time. This keeps the first move local and preserves the existing contract while responsibilities are untangled.

## Safety Net

- Existing tests to rely on: current UserService integration tests
- Characterization tests to add: notification side effect, analytics side effect, duplicate-email failure path
- Manual verification checkpoints: smoke test user creation in development
- Compiler, type, lint checks: `npm test`, `npm run typecheck`, `npm run build`

## Behavior Surface To Preserve

- Inputs and outputs: `createUser`, `findById`, `updateUser` signatures and return values
- Side effects: welcome email sent after successful creation, analytics event emitted once
- Ordering and timing expectations: analytics only after persistence succeeds
- Fallbacks and defaults: existing default role assignment
- Errors and edge cases: duplicate email rejection behavior and message

## Phases Or Commits

1. Add characterization tests for UserService behavior surface
2. Extract a local notification helper behind the existing flow
3. Introduce EmailService seam and swap the helper to use it
4. Update UserService tests around the email seam
5. Extract a local analytics helper behind the existing flow
6. Introduce AnalyticsService seam and swap the helper to use it
7. Update UserService tests around the analytics seam
8. Remove dead code and tighten names once seams are stable

## Decision Document

- Modules to be built or modified:
  - UserService (modified)
  - EmailService (new)
  - AnalyticsService (new)

- Interfaces that will change:
  - UserService constructor will accept EmailService and AnalyticsService
  - New IEmailService interface
  - New IAnalyticsService interface

- Technical clarifications:
  - EmailService uses SendGrid (existing account)
  - AnalyticsService posts to /api/analytics endpoint
  - Services will be singleton instances

- Architectural decisions:
  - Dependency injection for all services
  - Services depend on interfaces, not implementations
  - Each service in its own module

## Testing Decisions

- Test philosophy: test external behavior through public APIs, mock dependencies
- Modules to test:
  - UserService: integration tests with mocked services
  - EmailService: unit tests with mocked SendGrid client
  - AnalyticsService: unit tests with mocked HTTP client
- Prior art: AuthService tests show good pattern for service testing

## Mixed Work Packaging

This is refactor-only work. Any bug fixes or new notification behavior should land separately.

## Out of Scope

- Changing the email templates
- Adding new analytics events
- Modifying the database schema
- Updating the API endpoints
- Changing error messages

## Further Notes

- Assumptions:
  - Existing notification and analytics behavior is correct enough to preserve
- Unknowns and risks:
  - Some callers may rely on current duplicate-email error wording
- Follow-up work:
  - Consider cleanup of UserService naming once the seams are extracted
```

For the planning process that produces this template, see [02-planning-workflow.md](02-planning-workflow.md).
