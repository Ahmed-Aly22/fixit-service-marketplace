# UI Design System

This document is the source of truth for approved product UI decisions. Follow it unless the user explicitly approves a change.

## 1. Visual Direction

- Style: Modern & Minimal.
- Palette: medium clear blue, white, and light gray.
- Typography: Tajawal.
- Theme: Light Mode only.
- Overall layout: responsive, clean, clear, and easy to scan.

## 2. Layout and Navigation

- Support desktop, tablet, and mobile.
- On mobile, use a hamburger button that opens a side menu.
- Separate page sections primarily with clear dividers rather than enclosing every section in a card.
- Use light-gray dividers with clear visual distinction.
- Buttons should have very rounded corners.
- Icons should use a filled style where suitable.

## 3. Home Page

The approved homepage structure includes:
1. Search.
2. Service categories.
3. Popular services.
4. A clear action to create a new service request.

Choose sensible spacing and responsive arrangements consistent with the approved style.

## 4. Localization and Direction

- Support Arabic and English.
- Arabic must use RTL; English must use LTR.
- Check direction-sensitive alignment, spacing, navigation, icons, forms, and page structure.
- Use Tajawal consistently for product-facing text.

## 5. Images and Illustrations

- Use a mix of real photos, icons, and illustrations where appropriate.
- Prefer local assets and placeholders.
- Avoid runtime dependencies on external image URLs.
- Do not add image libraries or external services without approval if they introduce dependencies or architectural changes.

## 6. Components and Libraries

- Do not introduce a UI component library; implement components manually using the existing stack.
- Do not add dependencies without user approval.
- Reuse components when a repeated pattern warrants it, but avoid unnecessary abstraction.

## 7. Animation and Accessibility

- Animations may be noticeable, but must not harm performance or usability.
- Respect `prefers-reduced-motion`.
- Follow accessibility basics: semantic structure, keyboard access, visible focus, meaningful labels, alternative text, and suitable contrast.
- Do not use animation as the only way to communicate important state.

## 8. Design Flexibility

- The agent may make reasonable decisions for screens and details not yet specified.
- Explain meaningful UI decisions briefly when useful.
- Do not change the approved overall design language.
- Ask for approval before significant UX changes that alter a previously approved flow.
