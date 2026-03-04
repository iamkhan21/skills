# Accessibility Matchers

`@testing-library/jest-dom` provides matchers that verify accessible properties. Use these to ensure components are not just present but usable.

## State Matchers

```typescript
// Visibility
expect(screen.getByRole("dialog")).toBeVisible();
expect(screen.getByRole("tooltip")).not.toBeVisible();

// Enabled/disabled
expect(screen.getByRole("button", { name: /submit/i })).toBeEnabled();
expect(screen.getByRole("button", { name: /submit/i })).toBeDisabled();

// Form validity
expect(screen.getByLabelText(/email/i)).toBeInvalid();
expect(screen.getByLabelText(/email/i)).toBeValid();

// Required
expect(screen.getByLabelText(/email/i)).toBeRequired();

// Checked
expect(screen.getByRole("checkbox")).toBeChecked();
expect(screen.getByRole("checkbox")).not.toBeChecked();
expect(screen.getByRole("checkbox")).toBePartiallyChecked();
```

## Accessible Name & Description

```typescript
// Name (what screen readers announce)
expect(screen.getByRole("button")).toHaveAccessibleName("Submit form");
expect(screen.getByRole("img")).toHaveAccessibleName(/profile/i);

// Description (aria-describedby content)
expect(screen.getByRole("textbox")).toHaveAccessibleDescription(/must be at least 8 characters/i);
```

## Content Matchers

```typescript
// Text content
expect(screen.getByRole("status")).toHaveTextContent(/3 items/i);

// Form values
expect(screen.getByLabelText(/email/i)).toHaveValue("test@example.com");
expect(screen.getByRole("combobox")).toHaveDisplayValue("Option 1");

// CSS class (use sparingly — prefer behavioral assertions)
expect(screen.getByRole("alert")).toHaveClass("alert-danger");
```

## ARIA Attributes

```typescript
// Expanded state (accordions, menus)
expect(screen.getByRole("button", { name: /menu/i })).toHaveAttribute("aria-expanded", "true");

// Live regions
expect(screen.getByRole("status")).toHaveAttribute("aria-live", "polite");

// Current (navigation)
expect(screen.getByRole("link", { name: /home/i })).toHaveAttribute("aria-current", "page");
```

## When to Use Which

| Want to verify... | Use |
|-------------------|-----|
| Element exists in DOM | `toBeInTheDocument()` |
| Element is visible to user | `toBeVisible()` |
| Button can be clicked | `toBeEnabled()` |
| Form field shows error | `toBeInvalid()` |
| What screen reader announces | `toHaveAccessibleName()` |
| Tooltip/help text content | `toHaveAccessibleDescription()` |

Prefer `toBeVisible()` over `toBeInTheDocument()` when the distinction matters — an element can be in the DOM but hidden via CSS.
