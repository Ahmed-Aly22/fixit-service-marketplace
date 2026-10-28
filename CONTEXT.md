# Project Context

## 1. Project Overview

A bilingual service marketplace that connects customers with service workers in Cairo and Giza, Egypt.

Customers create service requests, workers submit offers with prices and proposed appointment times, and customers choose the offer that suits them.

The initial MVP supports ordinary service requests only. Urgent requests and native mobile applications are deferred to future versions.

## 2. Users and Roles

- **Customer:** Creates and manages service requests, compares offers, manages bookings, submits reviews, and files complaints.
- **Worker:** Maintains a service profile, selects approved specialties and service areas, submits offers, and manages accepted jobs.
- **Admin:** Reviews worker identity documents, manages approved specialties and areas, and handles complaints and administrative operations.

Each account has exactly one role. A person who needs both customer and worker capabilities must use separate accounts. The same phone number may be used for separate accounts with different roles, subject to phone ownership verification.

## 3. MVP Scope

- Responsive website for desktop, tablet, and mobile.
- Arabic and English, with RTL support for Arabic and LTR support for English.
- Ordinary service requests, offers, bookings, reviews, and complaints.
- Worker identity verification and administrative approval.
- Role-based permissions and controlled access to protected operations.

## 4. Core Technology

- **Monorepo and package management:** pnpm workspaces.
- **Web application:** Next.js App Router, React, TypeScript, and Tailwind CSS.
- **Backend API:** NestJS with REST endpoints.
- **Database and ORM:** PostgreSQL and Prisma.
- **Background processing:** A dedicated worker using pg-boss.
- **Shared contracts:** `packages/contracts`, including shared request/response types and appropriate Zod schemas.
- **Web authentication:** Auth.js Credentials provider.
- **Testing:** Use the tools and commands supported by the actual project configuration.

Verify the installed versions and existing project configuration before relying on version-specific behavior or commands.

## 5. Architectural Constraints

- `apps/web` communicates with the backend through the API and must not access PostgreSQL directly.
- `apps/api` is the only application responsible for database access.
- `apps/worker` runs background jobs and reuses appropriate shared backend services without calling the project's own API over HTTP for internal operations.
- Cross-application contracts belong in `packages/contracts` where appropriate.
- Keep application boundaries clear and avoid duplicating business logic across applications.
- Preserve the approved architecture. Changes to approved architectural decisions require impact analysis and user approval.

## 6. Authoritative References

Use each document as the authoritative source for its information:

- `docs/architecture.md` — architecture details, technology choices, and architectural decisions.
- `docs/api-design.md` — API conventions and endpoint design.
- `docs/business-rules.md` — numbered business rules organized by module.
- `docs/modules-overview.md` — module responsibilities and relationships.
- `docs/ui-design.md` — approved UI design system and visual conventions.
- `docs/security.md` — security and data-protection decisions.
- `docs/testing-strategy.md` — testing approach and verification expectations.
- `docs/requirements-traceability.md` — mapping business rules to implementation and tests.
- `STATE.md` — current project state, module checklists, progress, blockers, and next steps.

## 7. Language and Documentation

All agent instruction and project documentation files must be written in clear, concise English. The product itself must support Arabic and English.

Keep this context file concise. Store detailed requirements, implementation procedures, and specialized instructions in their designated authoritative files.
