---
name: modern-ui-page
description: Creates modern full-page UI with strong visual direction while preserving accessibility and UX fundamentals. Use when the user asks for a landing page, marketing page, dashboard page, homepage, redesign, or to make a page feel more polished, modern, or premium.
---

# Modern UI Page

## Quick Start

1. Identify the page's audience, purpose, primary action, and required content.
2. Inspect the stack and existing design language, then choose a fitting visual direction.
3. Build the requested page and interactions using existing conventions.
4. Verify responsive behavior, accessibility, and UX; report assumptions and actual checks.

Keep the scope page-sized. For an app-wide request, agree on a first page or sequence rather than silently reducing scope.

Read only the references needed for the task:
- [Workflow](references/workflow.md): page narrative and implementation order
- [Accessibility](references/accessibility.md): interaction and readability checks
- [UX](references/ux.md): hierarchy, actions, and states
- [Visual direction](references/visual-direction.md): composition and styling choices

## Workflow

### 1) Understand the brief

Identify:
- audience
- goal
- primary CTA
- required sections
- brand/tone constraints
- device priorities

If the prompt is sparse, infer a direction from context and state the assumptions.
Ask before guessing about consequential product behavior, real-world claims, or missing data contracts.

### 2) Pick a strong visual concept

Choose an intentional direction, for example:
- minimal and editorial
- premium and dark
- technical and dense
- playful and high-contrast
- product-marketing with narrative sections

Choose a direction that fits the audience and existing brand. Consistency can be more valuable than novelty, especially in an established product.

### 3) Build the page

- Follow the repo's existing stack and coding conventions.
- Prefer composition over one giant component when the page is complex.
- Use existing design tokens, theme primitives, and components if available.
- Keep section hierarchy easy to scan.
- Make the primary CTA obvious.
- Make controls work: use real destinations and implement requested interactions. Label demo-only behavior and placeholders; do not invent testimonials, customer logos, or product claims.
- Handle relevant loading, empty, error, and success states for data-driven pages.

### 4) Check usability

- Check landmarks, headings, labels, contrast, focus, and keyboard operation against the accessibility reference.
- Check hierarchy, action clarity, responsive layout, and relevant states against the UX reference.
- Keep typography and composition strong without relying on decoration or motion.

### 5) Handoff

At the end, briefly report:
- the visual direction chosen
- assumptions made from missing requirements
- major accessibility or UX tradeoffs, if any remain

## Guardrails

- inspect surrounding code before editing
- reuse installed libraries before adding new ones
- preserve design tokens and established interaction patterns unless redesigning them is in scope
- avoid unusual interactions that hide critical actions or sacrifice readability
- avoid new dependencies or remote assets just for decoration; respect licensing and project privacy constraints

## Verification

Before completing work with this skill:
- inspect repo scripts and run the relevant validators
- review the page against the accessibility and UX references
- if browser tools are available, inspect narrow and wide viewports, keyboard focus, overflow, and the primary action; fix observed issues
- use real-browser tests for layout and interaction behavior that DOM simulators cannot verify
- distinguish code review from browser checks; do not claim accessibility compliance or visual verification without evidence
- confirm the result matches the user's requested page scope
- call out any unresolved tradeoffs explicitly
