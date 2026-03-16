# Accessibility Fundamentals for Modern Pages

The goal is not bare-minimum compliance language. The goal is a page people can actually read, navigate, and operate.

## Structure

- Use semantic landmarks where appropriate: `header`, `main`, `nav`, `section`, `footer`
- Keep heading levels logical and scannable
- Prefer real buttons and links over clickable generic containers

## Text and Contrast

- Preserve readable contrast between text and background
- Avoid low-contrast decorative text in critical content
- Use large display text carefully; dramatic typography must still be legible

## Focus and Keyboard Use

- Interactive elements need visible focus styles
- Avoid interaction patterns that depend only on hover
- Keyboard users should be able to reach primary actions predictably

## Naming and Labels

- Button text should communicate action
- Form fields need clear labels
- Images that convey meaning need useful alt text
- Decorative imagery should stay decorative

## Motion and Effects

- Animation should support orientation, not distract from content
- Avoid effects that obscure text or delay access to important actions
- Strong visual direction is welcome, but content must remain readable during motion states

## Responsive Accessibility

- Verify small-screen readability
- Avoid collapsing important content into unclear affordances
- Preserve tap target clarity and spacing on touch devices

## Common Failure Modes

- Gorgeous hero with unreadable overlay text
- Tiny text used as a style choice
- Hidden focus rings
- Cards or tiles that look clickable but have ambiguous semantics
- Critical information revealed only on hover

## Final Check

Before handoff, ask:
- Can a new user understand the page without guessing?
- Can a keyboard user operate the meaningful controls?
- Is the main content readable without perfect eyesight or ideal display conditions?
