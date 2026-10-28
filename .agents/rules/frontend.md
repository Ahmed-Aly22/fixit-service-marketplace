# Frontend Rules

## 1. Framework and Boundaries

- Use Next.js App Router, React, TypeScript, and Tailwind CSS as approved in `CONTEXT.md`.
- Keep `apps/web` focused on presentation and user interaction; access application data through the backend API.
- Never access PostgreSQL or Prisma directly from the frontend.
- Follow the approved API contracts and use shared types from `packages/contracts` where appropriate.
- Inspect existing components, routing, styling conventions, and dependencies before implementing changes.

## 2. Server and Client Components

- Prefer React Server Components by default.
- Add `"use client"` only when browser APIs, state, effects, event handlers, or client-only libraries are required.
- Keep Client Components as small and focused as practical.
- Do not move an entire page or layout to the client when only a small interactive part needs it.
- Fetch data on the server when appropriate and pass serializable props to Client Components.
- Avoid duplicate fetching and unnecessary client-side effects.

## 3. TypeScript and Data Handling

- Use explicit types and shared contracts where appropriate.
- Avoid `any`; use a safe alternative or document a narrow, justified exception.
- Validate API data and handle loading, empty, success, and error states.
- Do not assume API responses are valid merely because they are typed.
- Avoid duplicating data models or business rules in frontend code.
- Keep secrets and server-only configuration out of client bundles.

## 4. UI Design System

Follow `docs/ui-design.md` as the source of truth for visual design.

- Use the approved Modern & Minimal style.
- Use the approved medium clear blue, white, and light-gray palette.
- Use Tajawal for the product interface.
- Support Arabic RTL and English LTR correctly.
- Build responsive layouts for mobile, tablet, and desktop.
- Use the approved hamburger side menu on mobile.
- Separate page sections primarily with clear dividers rather than wrapping every section in a card.
- Use very rounded buttons, filled icons, and light-gray dividers.
- Implement Light Mode only unless the user approves a change.
- Follow the approved home-page structure: search, categories, popular services, and a clear new-service-request action.
- Use local images, icons, illustrations, or approved placeholders. Avoid runtime dependencies on external image URLs.

## 5. Components and Styling

- Do not introduce a UI component library or additional styling library without explicit user approval.
- Prefer reusable components when they represent a real repeated pattern.
- Keep components focused, accessible, and easy to maintain.
- Use Tailwind CSS and the project's existing conventions consistently.
- Avoid arbitrary one-off styling when an approved design token or reusable pattern exists.
- Do not redesign unrelated areas during a focused task.

## 6. Forms and Interaction

- Provide clear labels, validation feedback, and useful error messages.
- Keep client-side validation aligned with server-side validation, but never treat client validation as a security boundary.
- Prevent duplicate submissions where appropriate and show submission progress.
- Make interactive controls usable by keyboard and assistive technology.
- Preserve user-entered data when recoverable errors occur.

## 7. Accessibility and Motion

- Follow accessibility basics: semantic HTML, keyboard access, visible focus, suitable contrast, meaningful labels, and useful alternative text.
- Use noticeable but purposeful animations without harming performance or usability.
- Respect `prefers-reduced-motion`.
- Do not use animation as the only way to communicate important state changes.
- Avoid unnecessary layout shifts and heavy visual effects.

## 8. Localization

- Support Arabic and English throughout user-facing flows.
- Apply RTL for Arabic and LTR for English, including alignment, spacing, icons, and navigation behavior where direction matters.
- Keep user-facing text in the project's approved localization structure rather than hardcoding text throughout components.
- Avoid concatenating translated sentence fragments when complete localized messages are needed.
- Test layouts with both short and long Arabic and English strings.

## 9. Performance

- Prefer framework-supported image optimization for local assets.
- Avoid unnecessary client-side JavaScript, re-renders, large dependencies, and duplicate requests.
- Use pagination or other approved API mechanisms for large lists.
- Keep loading feedback responsive and avoid blocking the whole page for small independent operations.
- Verify performance-sensitive changes using available project tooling when practical.

## 10. Frontend Verification

- Run the relevant type checks, linting, tests, and build commands defined by the actual workspace configuration.
- Check responsive behavior and both language directions for affected screens.
- Review keyboard access, focus states, loading/error/empty states, and console errors where applicable.
- Report any checks that were not run and any known limitations.
