# Testing Hooks

- [Basic Usage](#basic-usage)
- [Testing State Updates](#testing-state-updates)
- [Testing with Props](#testing-with-props)
- [Testing with Context](#testing-with-context)

## Basic Usage

```typescript
import { renderHook } from "@testing-library/react";

test("returns initial count", () => {
  const { result } = renderHook(() => useCounter(0));
  expect(result.current.count).toBe(0);
});
```

## Testing State Updates

Wrap state changes in `act`:

```typescript
import { renderHook, act } from "@testing-library/react";

test("increments count", () => {
  const { result } = renderHook(() => useCounter(0));
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

Async updates use `waitFor`:

```typescript
test("fetches data", async () => {
  const { result } = await waitFor(() => renderHook(() => useFetchUser(1)));
  
  expect(result.current.user).toEqual({ id: 1, name: "Test" });
});
```

## Testing with Props

```typescript
test("updates when prop changes", () => {
  const { result, rerender } = renderHook(
    ({ id }) => useFetchUser(id),
    { initialProps: { id: 1 } }
  );
  
  expect(result.current.user.id).toBe(1);
  
  rerender({ id: 2 });
  
  expect(result.current.user.id).toBe(2);
});
```

## Testing with Context

```typescript
const wrapper: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <UserContext.Provider value={{ user: mockUser }}>
    {children}
  </UserContext.Provider>
);

test("accesses context", () => {
  const { result } = renderHook(() => useCurrentUser(), { wrapper });
  expect(result.current).toEqual(mockUser);
});
```
