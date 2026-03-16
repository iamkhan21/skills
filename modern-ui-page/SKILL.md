---
name: modern-ui-page
description: Creates modern full-page UI with strong visual direction while preserving accessibility and UX fundamentals. Use when the user asks for a landing page, marketing page, dashboard page, homepage, redesign, or to make a page feel more polished, modern, or premium.
---

# Modern UI Page

## Quick Start

1) Confirm page scope
- This skill is for a **full page** or page-sized surface, not an entire app or design system.
- If the request is broader, implement one page first.

2) Understand the page before coding
- What is the page for?
- Who is it for?
- What is the primary action?
- What content or sections are required?

3) Choose a visual direction
- Pick a clear aesthetic instead of defaulting to generic SaaS UI.
- Favor memorable composition, typography, color, and spacing.
- Keep visuals subordinate to readability and navigation.

4) Build with the repo's existing conventions
- Inspect current framework, libraries, and component patterns first.
- Reuse installed dependencies and established patterns.

5) Self-audit before handoff
- Check accessibility and UX fundamentals.
- Summarize assumptions and tradeoffs.

Links:
- Workflow details: see `modern-ui-page/references/workflow.md`
- Accessibility checks: see `modern-ui-page/references/accessibility.md`
- UX checks: see `modern-ui-page/references/ux.md`
- Visual direction guidance: see `modern-ui-page/references/visual-direction.md`

## Scope

Use this skill for:
- Landing pages
- Marketing pages
- Homepage redesigns
- Dashboard pages
- Promotional or campaign pages
- Any page-level UI that should feel modern, polished, and intentional

Do not use this skill for:
- App bootstrapping
- Full design systems
- Test-heavy workflows
- Deep architecture setup

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

### 2) Pick a strong visual concept

Choose an intentional direction, for example:
- minimal and editorial
- premium and dark
- technical and dense
- playful and high-contrast
- product-marketing with narrative sections

Do not mix several weak directions. Commit to one clear idea.

### 3) Build the page

- Follow the repo's existing stack and coding conventions.
- Prefer composition over one giant component when the page is complex.
- Use existing design tokens, theme primitives, and components if available.
- Keep section hierarchy easy to scan.
- Make the primary CTA obvious.

### 4) Preserve accessibility fundamentals

Always check:
- semantic landmarks and heading order
- readable text and contrast-aware color usage
- visible focus styles
- keyboard-safe interaction patterns
- meaningful labels, alt text, and button text where relevant
- responsive behavior across small and large screens

### 5) Preserve UX fundamentals

Always check:
- the page communicates its purpose quickly
- information hierarchy is obvious
- important actions are easy to find
- spacing supports readability
- empty, loading, and error states exist where the feature needs them
- visuals do not obscure content or navigation

### 6) Handoff

At the end, briefly report:
- the visual direction chosen
- assumptions made from missing requirements
- major accessibility or UX tradeoffs, if any remain

## Guardrails

Always:
- inspect surrounding code before editing
- reuse installed libraries before adding new ones
- optimize for page quality, not just speed of implementation
- keep the page distinctive without sacrificing clarity

Never:
- invent a new stack when the repo already has conventions
- hide critical actions behind unusual interactions
- sacrifice readability for decoration
- claim accessibility compliance without checking the basics

## Verification

Before completing work with this skill:
- inspect repo scripts and run the relevant validators
- review the page against the accessibility and UX references
- confirm the result matches the user's requested page scope
- call out any unresolved tradeoffs explicitly

## Output Expectations

The final result should:
- feel visually intentional rather than generic
- read clearly on first scan
- guide the user toward the primary action
- respect accessibility and UX fundamentals
- fit naturally into the repo's existing frontend patterns
