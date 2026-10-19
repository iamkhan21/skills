# Query Cheatsheet

- [Query Types](#query-types)
- [Query Priority](#query-priority)
- [Within Queries](#within-queries)
- [Debugging](#debugging)
- [Async Queries](#async-queries)
- [Common Mistakes](#common-mistakes)

## Query Types

| Type | No matches | One match | Multiple matches | Use case |
|------|------------|-----------|------------------|----------|
| `getBy*` | Throws | Element | Throws | Element exists now |
| `getAllBy*` | Throws | Array | Array | One or more exist now |
| `queryBy*` | `null` | Element | Throws | Assert absence |
| `queryAllBy*` | `[]` | Array | Array | Assert a collection or absence |
| `findBy*` | Rejects after timeout | Resolves element | Rejects if ambiguity persists | Await an element |
| `findAllBy*` | Rejects after timeout | Resolves array | Resolves array | Await one or more elements |

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

### Fallback: By Placeholder

```typescript
screen.getByPlaceholderText("Enter your email")
screen.getByPlaceholderText(/search/i)
```

### 3. By Text Content

```typescript
screen.getByText(/loading/i)
screen.getByText("Submit")
screen.getByText((content, element) => element.tagName === "SPAN" && content.includes("Total"))
```

### 4. By Display Value (form elements)

```typescript
screen.getByDisplayValue("user@example.com")
screen.getByDisplayValue(/selected/i)
```

### 5. By Alt Text (images)

```typescript
screen.getByAltText(/profile picture/i)
```

### Fallback: By Title

```typescript
screen.getByTitle(/close/i)
```

### Last Resort: By Test ID

```typescript
screen.getByTestId("submit-button")
screen.getByTestId(/^user-card-/)
```

## Within Queries

Scope queries to a container:

```typescript
// A form needs an accessible name to expose the form landmark role.
const form = within(screen.getByRole("form", { name: /sign in/i }));
form.getByRole("button", { name: /submit/i });

const dialog = within(screen.getByRole("dialog"));
expect(dialog.getByText(/confirm/i)).toBeInTheDocument();
```

## Debugging

```typescript
screen.debug()                          // Print entire DOM
screen.debug(screen.getByRole("form", { name: /sign in/i }))
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
// Less helpful diagnostics: this assertion still fails when null
expect(screen.queryByRole("button")).toBeInTheDocument()

// RIGHT: Use getBy to fail fast
expect(screen.getByRole("button")).toBeInTheDocument()

// WRONG: Using getBy for assertion that element is absent
expect(screen.getByRole("alert")).not.toBeInTheDocument() // Throws before expect

// RIGHT: Use queryBy for absence assertions
expect(screen.queryByRole("alert")).not.toBeInTheDocument()
```

Choose queries by semantics, not a rigid ranking. Password inputs have no implicit textbox role; use their label. Placeholder text is not a substitute for a label.
