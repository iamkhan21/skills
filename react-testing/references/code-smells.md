# Code Smells

Use test struggles as design feedback.

## Smell Detection Table

| Smell | Meaning |
|-------|---------|
| Too many tests per module | Violates SRP—split the module |
| Complex test setup | Leaked abstractions—extract dependency |
| Need to mock your own code | Wrong boundaries—use adapter or refactor |
| Test never fails | Useless—remove it |
| Testing private methods | Test public behavior instead |
| Test name mentions implementation | Rename to describe behavior |
| Fragile tests (break on refactor) | Testing implementation details |
| Slow test suite | Too many mocks, consider MSW |

## Refactoring Triggers

When tests signal design issues:

1. **Hard to set up** → Accept dependencies via props/context
2. **Need internal mocks** → Extract to adapter pattern
3. **Multiple concerns in one test** → Split component/hook
4. **Testing state instead of output** → Return results via callbacks

## Coverage Guidance

- ~70% coverage is sufficient
- Critical paths > coverage percentage
- 100% mandate creates perverse incentives
- Every line counts equally—doesn't distinguish importance
