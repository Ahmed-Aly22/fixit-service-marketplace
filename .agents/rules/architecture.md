# Architecture Rules

## 1. Architecture First

- Read `CONTEXT.md` and `docs/architecture.md` before making architectural decisions.
- Inspect the existing repository structure, package configuration, and implementation before proposing changes.
- Follow the approved monorepo architecture and keep application responsibilities clearly separated.
- Do not introduce a new architectural pattern or replace an approved technology without impact analysis and explicit user approval.

## 2. Application Boundaries

- `apps/web` contains the Next.js user interface and communicates with the backend through the API.
- `apps/api` contains the NestJS REST API and is the only application responsible for database access.
- `apps/worker` handles background jobs and reuses appropriate shared backend services without calling the project's own API over HTTP for internal operations.
- `packages/contracts` contains shared request/response types and appropriate Zod schemas.
- Do not access PostgreSQL directly from `apps/web`.
- Do not duplicate business logic across the web application, API, and worker.
- Keep shared code in the appropriate package instead of copying it between applications.

## 3. Data and Database Integrity

- Use PostgreSQL and Prisma according to the approved architecture.
- Keep database access and persistence logic within the backend's established boundaries.
- Use transactions when multiple database operations must succeed or fail together.
- Enforce critical business constraints on the server; do not rely on frontend validation alone.
- Validate input at system boundaries and handle database errors safely.
- Use Prisma migrations for schema changes; do not modify production data or schema manually without an approved procedure.
- Review the effects of schema changes on existing data, contracts, API consumers, and background jobs before implementation.

## 4. API and Contracts

- Follow the conventions in `docs/api-design.md`.
- Keep API request and response contracts consistent with `packages/contracts` where appropriate.
- Validate incoming data with the project's established validation approach.
- Use consistent error handling and avoid exposing internal implementation details in API responses.
- Do not silently introduce breaking API or contract changes.
- Consider authorization, validation, pagination, and failure cases when implementing endpoints.

## 5. Security and Privacy

- Apply authentication and authorization on the server for protected operations.
- Enforce role-based access and resource ownership; never trust client-supplied roles or permissions.
- Validate and normalize untrusted input where appropriate.
- Store secrets in environment variables or an approved secrets mechanism; never commit credentials, tokens, or private keys.
- Protect sensitive identity documents, personal information, and uploaded files from unauthorized access.
- Do not expose private files through public static directories or unrestricted URLs.
- Use safe defaults for CORS, HTTP security headers, rate limiting, and error responses.
- Follow `docs/security.md` for detailed security and data-protection requirements.
- Request approval before weakening or bypassing an approved security control.

## 6. Code Quality and Maintainability

- Prefer clear, typed, modular code that follows existing project conventions.
- Keep modules focused and avoid unnecessary abstractions, duplication, and oversized components or services.
- Use explicit types; avoid `any` unless justified and narrowly scoped.
- Handle loading, empty, success, and failure states where relevant.
- Use consistent naming and organize code by responsibility.
- Do not refactor unrelated code as part of a focused task.
- Add comments when they explain intent or non-obvious constraints, not when they merely repeat the code.

## 7. Dependencies and Configuration

- Reuse existing dependencies and utilities when suitable.
- Request approval before adding, removing, or replacing dependencies.
- Verify package versions and supported APIs before using version-specific features.
- Keep environment-specific configuration outside source code.
- Update relevant setup documentation when an approved configuration change affects development or deployment.

## 8. Architecture Change Gate

Before proposing a significant architecture change:

1. Explain the problem the change solves.
2. Describe the proposed change and reasonable alternatives.
3. Identify affected applications, packages, contracts, data, security, and tests.
4. Explain migration risks and compatibility implications.
5. Request explicit user approval before implementation.

Minor implementation choices that preserve the approved architecture do not require a separate architecture approval.

## 9. Verification

- Verify architectural boundaries and affected integrations after implementation.
- Run relevant type checks, linting, tests, and build commands supported by the actual project configuration.
- Review changes for security issues, unintended coupling, duplicated business logic, and breaking contracts.
- Report checks that were not run and any unresolved risks.
