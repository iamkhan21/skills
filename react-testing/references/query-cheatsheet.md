# Query Cheatsheet

- [Query Types](#query-types)
- [Query Priority](#query-priority)
- [Within Queries](#within-queries)
- [Debugging](#debugging)
- [Async Queries](#async-queries)
- [Common Mistakes](#common-mistakes)

## Query Types

| Type | Throws on 0 | Throws on 1+ | Use Case |
|------|-------------|--------------|----------|
| `getBy*` | Error | No | Assert element exists |
| `getAllBy*` | Error | Error | Assert multiple elements |
| `queryBy*` | No (null) | No | Assert element doesn't exist |
| `queryAllBy*` | No ([]) | No | Assert no elements match |
| `findBy*` | Error (async) | No | Wait for async element |
| `findAllBy*` | Error (async) | Error | Wait for multiple async |

## Query Priority (Most to Least Preferred)

### 1. By Role

```typescript
screen.getByRole("button", { name: /submit/i })
screen.getByRole("textbox", { name: /email/i })
screen.getByRole("heading", { level: 1, name: /welcome/i })
screen.getByRole("checkbox", { checked: true })
screen.getByRole("link", { name: /learn more/i })
screen.getByRole("dialog")
screen.getByRole("alert")
screen.getByRole("navigation")
screen.getByRole("listitem")
screen.getByRole("option", { name: /blue/i })
```

**Common roles**: button, checkbox, link, textbox, searchbox, combobox, listbox, radio, switch, tab, menu, dialog, alert, status, navigation, main, banner, contentinfo

### 2. By Label Text

```typescript
screen.getByLabelText(/email address/i)
screen.getByLabelText(/remember me/i)
screen.getByLabelText(/^password$/i)
```

### 3. By Placeholder

```typescript
screen.getByPlaceholderText("Enter your email")
screen.getByPlaceholderText(/search/i)
```

### 4. By Text Content

```typescript
screen.getByText(/loading/i)
screen.getByText("Submit")
screen.getByText((content, element) => element.tagName === "SPAN" && content.includes("Total"))
```

### 5. By Display Value (form elements)

```typescript
screen.getByDisplayValue("user@example.com")
screen.getByDisplayValue(/selected/i)
```

### 6. By Alt Text (images)

```typescript
screen.getByAltText(/profile picture/i)
```

### 7. By Title

```typescript
screen.getByTitle(/close/i)
```

### 8. By Test ID (last resort)

```typescript
screen.getByTestId("submit-button")
screen.getByTestId(/^user-card-/)
```

## Within Queries

Scope queries to a container:

```typescript
const form = within(screen.getByRole("form"));
form.getByRole("button", { name: /submit/i });

const dialog = within(screen.getByRole("dialog"));
expect(dialog.getByText(/confirm/i)).toBeInTheDocument();
```

## Debugging

```typescript
screen.debug()                          // Print entire DOM
screen.debug(screen.getByRole("form"))  // Print specific element
screen.logTestingPlaygroundURL()        // Generate playground link
```

## Async Queries

```typescript
// findBy = getBy + waitFor
await screen.findByRole("alert")

// Custom timeout
await screen.findByText(/saved/i, {}, { timeout: 3000 })

// waitFor with matcher
await waitFor(() => {
  expect(screen.getByRole("status")).toHaveTextContent(/success/i)
})
```

## Common Mistakes

```typescript
// WRONG: Using queryBy when element should exist
expect(screen.queryByRole("button")).toBeInTheDocument() // Silent failure if null

// RIGHT: Use getBy to fail fast
expect(screen.getByRole("button")).toBeInTheDocument()

// WRONG: Using getBy for assertion that element is absent
expect(screen.getByRole("alert")).not.toBeInTheDocument() // Throws before expect

// RIGHT: Use queryBy for absence assertions
expect(screen.queryByRole("alert")).not.toBeInTheDocument()
```
