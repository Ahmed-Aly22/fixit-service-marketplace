# Architecture

## 1. Overview

The product is a bilingual service marketplace delivered as a responsive website. The approved architecture is a pnpm-workspaces monorepo containing a Next.js web application, a NestJS REST API, a background worker, and shared contracts.

## 2. Monorepo Layout

- `apps/web`: Next.js App Router, React, TypeScript, and Tailwind CSS user interface.
- `apps/api`: NestJS REST API and the only application responsible for database access.
- `apps/worker`: background processing using pg-boss.
- `packages/contracts`: shared request/response types and appropriate Zod schemas.
- `docs`: authoritative project documentation.
- `.agents/rules`: project-wide and domain-specific implementation rules.
- `.agents/skills`: repeatable task workflows.

Inspect the actual repository before assuming this structure is already implemented.

## 3. Technology Choices

- Package manager and workspace: pnpm workspaces.
- Web: Next.js App Router, React, TypeScript, Tailwind CSS.
- API: NestJS REST API.
- Database: PostgreSQL.
- ORM and migrations: Prisma.
- Background jobs: pg-boss.
- Website authentication: Auth.js Credentials provider.
- API documentation: Swagger/OpenAPI.
- Logging: Pino.
- Validation/contracts: shared TypeScript types and appropriate Zod schemas.
- Testing: configured Jest, Vitest, React Testing Library, and Playwright where present.

Verify actual installed versions and project scripts before using version-specific features or commands.

## 4. Application Responsibilities

### Web (`apps/web`)

Owns rendering, navigation, forms, client interactions, and user experience. It communicates with the backend through the API and must not access PostgreSQL or Prisma directly.

### API (`apps/api`)

Owns HTTP endpoints, authentication integration boundaries, authorization, business rules, validation, database access, and API documentation. Keep controllers thin and place reusable business behavior in appropriate services/modules.

### Worker (`apps/worker`)

Owns asynchronous/background jobs. It should reuse suitable shared backend services without making internal HTTP calls to the project's own API. Jobs should be observable and safe to retry where practical.

### Contracts (`packages/contracts`)

Holds cross-application request/response types and appropriate schemas. Avoid coupling the shared package to framework-specific runtime details unless explicitly approved.

## 5. Data and Consistency

- PostgreSQL is the primary relational database.
- Prisma manages the schema and migrations.
- Use transactions for related writes that must remain consistent.
- Consider race conditions for booking acceptance, worker capacity, and schedule conflicts.
- Enforce critical constraints on the server and, where appropriate, in database constraints or atomic transactional logic.
- Keep persistence access in the API boundary; do not introduce direct web-to-database access.

## 6. API Conventions

- REST API prefix: `/api/v1`.
- Document endpoints with Swagger/OpenAPI.
- Use consistent response and error formats.
- Use offset pagination with `page` and `limit`.
- Define supported filters and sorts explicitly.
- Avoid breaking contracts without impact analysis and user approval.

See `docs/api-design.md`.

## 7. Background Processing

Use pg-boss for approved asynchronous work. Validate job payloads, minimize personal data in payloads, handle retries and failures, and prevent duplicate side effects where possible.

## 8. Authentication and Security Boundaries

- The website uses the approved Auth.js Credentials approach.
- Protected operations require server-side authentication, authorization, role checks, and ownership checks.
- Secrets remain in environment configuration or an approved secrets mechanism.
- Identity documents and private uploads must not be publicly accessible.
- CORS, rate limits, security headers, and safe error handling must be configured for deployment.

See `docs/security.md`.

## 9. Architecture Change Process

For significant changes, document the problem, alternatives, affected applications/contracts/data/security/tests, migration risks, and compatibility impact. Obtain explicit user approval before implementation.

## 10. Verification

Verify boundaries and affected integrations after changes. Use the repository's actual scripts for type checking, linting, tests, migrations, and builds. Report any checks not performed.
