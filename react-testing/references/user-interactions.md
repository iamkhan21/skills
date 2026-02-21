# User Interactions

- [Setup](#setup)
- [Common Interactions](#common-interactions)
- [Keyboard](#keyboard)
- [Pointer](#pointer)
- [Clipboard](#clipboard)
- [userEvent vs fireEvent](#userevent-vs-fireevent)

## Setup

Always create a user instance:

```typescript
const user = userEvent.setup();
```

Or with options:

```typescript
const user = userEvent.setup({
  delay: 50, // Simulate typing delay (ms)
  pointerEventsCheck: PointerEventsCheckLevel.Strict,
});
```

## Common Interactions

```typescript
// Click
await user.click(screen.getByRole("button"));

// Type text
await user.type(screen.getByLabelText(/email/i), "test@example.com");

// Clear input
await user.clear(screen.getByLabelText(/email/i));

// Select option
await user.selectOptions(screen.getByRole("listbox"), "option-value");

// Upload file
const file = new File(["content"], "test.txt", { type: "text/plain" });
await user.upload(screen.getByLabelText(/upload/i), file);

// Hover
await user.hover(screen.getByText(/menu/i));
await user.unhover(screen.getByText(/menu/i));

// Tab navigation
await user.tab(); // Move to next focusable
await user.tab({ shift: true }); // Move to previous
```

## Keyboard

```typescript
// Single key
await user.keyboard("{Enter}");
await user.keyboard("{Escape}");
await user.keyboard("{Tab}");

// Key combinations
await user.keyboard("{Control>}c{/Control}"); // Ctrl+C
await user.keyboard("{Shift>}a{/Shift}"); // Shift+A
await user.keyboard("{Meta>}Enter{/Meta}"); // Cmd+Enter

// Type with keyboard
await user.keyboard("Hello World");
```

**Special keys**: `{Enter}`, `{Escape}`, `{Tab}`, `{Backspace}`, `{Delete}`, `{ArrowUp}`, `{ArrowDown}`, `{Control}`, `{Shift}`, `{Alt}`, `{Meta}`

## Pointer

For complex mouse interactions:

```typescript
await user.pointer([
  { target: screen.getByText(/item/i) }, // Move to element
  { keys: "[MouseLeft]" }, // Press left button
  { keys: "[/MouseLeft]" }, // Release
]);

// Drag and drop
await user.pointer([
  { target: screen.getByText(/drag me/i) },
  { keys: "[MouseLeft>]" },
  { coords: { x: 100, y: 200 } },
  { keys: "[/MouseLeft]" },
]);
```

## Clipboard

```typescript
// Write to clipboard
await user.clipboard("copied text");

// Read from clipboard (requires permissions)
const text = await navigator.clipboard.readText();
```

## userEvent vs fireEvent

| userEvent | fireEvent |
|-----------|-----------|
| High-level, user-centric | Low-level, DOM-centric |
| Fires all related events | Fires single event |
| Handles focus, timing | Manual focus management |
| **Preferred for tests** | Use only when userEvent lacks API |

```typescript
// userEvent: one call, multiple events
await user.click(button); // focus, mouseover, mousedown, mouseup, click

// fireEvent: one event only
fireEvent.click(button); // Just click event
```
