# Refactor Plan Template

Template for documenting refactoring plans in GitHub issues or PRs.

## Template

```markdown
## Problem Statement

[The problem being solved, from the developer's perspective. What pain points exist? What would ideal look like?]

## Solution

[The proposed solution, from the developer's perspective. What approach will be taken? Why this approach over alternatives?]

## Commits

[Detailed implementation plan broken into the tiniest commits possible. Each commit should leave the codebase in a working state.]

1. [Commit description]
2. [Commit description]
3. [Commit description]
...

## Decision Document

[List of implementation decisions made]

- Modules to be built/modified: [list]
- Interfaces that will change: [list]
- Technical clarifications: [any clarifications]
- Architectural decisions: [decisions]
- Schema changes: [if any]
- API contracts: [if any]

## Testing Decisions

[How testing will be handled]

- Test philosophy: [e.g., test external behavior, not implementation details]
- Modules to test: [list]
- Prior art: [similar existing tests to reference]

## Out of Scope

[What will NOT be changed in this refactor]

- [Item 1]
- [Item 2]
...

## Further Notes

[Any additional context or notes]

[Optional - only include if needed]
```

## Example

```markdown
## Problem Statement

The UserService has grown to handle multiple responsibilities: user CRUD, email notifications, and analytics tracking. This makes it hard to test, hard to modify, and changes in one area risk breaking others. Developers avoid touching it because it's unclear what effects their changes might have.

## Solution

Extract separate services for each responsibility using the Single Responsibility Principle. The UserService will focus only on user data operations. EmailService will handle all email notifications. AnalyticsService will track user events. Services will communicate through explicit interfaces.

## Commits

1. Add characterization tests for UserService (covers all current behavior)
2. Extract EmailService interface
3. Implement EmailService with SendGrid
4. Inject EmailService into UserService
5. Move email logic from UserService to EmailService
6. Update UserService tests to mock EmailService
7. Add unit tests for EmailService
8. Extract AnalyticsService interface
9. Implement AnalyticsService
10. Inject AnalyticsService into UserService
11. Move analytics tracking from UserService to AnalyticsService
12. Update UserService tests to mock AnalyticsService
13. Add unit tests for AnalyticsService
14. Remove unused imports and dead code from UserService
15. Update UserService documentation

## Decision Document

- Modules to be built/modified:
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

- Test philosophy: Test external behavior through public APIs, mock dependencies
- Modules to test:
  - UserService: integration tests with mocked services
  - EmailService: unit tests with mocked SendGrid client
  - AnalyticsService: unit tests with mocked HTTP client
- Prior art: AuthService tests show good pattern for service testing

## Out of Scope

- Changing the email templates
- Adding new analytics events
- Modifying the database schema
- Updating the API endpoints
- Changing error messages

## Further Notes

This refactor should be done incrementally. Each commit should pass all tests. If tests fail, fix before proceeding to next commit. The goal is to have zero downtime and no behavior changes during the refactor.
```

For the planning process that produces this template, see [02-planning-workflow.md](02-planning-workflow.md).
