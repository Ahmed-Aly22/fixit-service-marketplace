# Backend and Worker Rules

## 1. Responsibilities and Boundaries

- Follow `CONTEXT.md`, `docs/architecture.md`, and `docs/api-design.md`.
- `apps/api` is the NestJS REST API and the only application responsible for database access.
- `apps/worker` processes background jobs and reuses appropriate backend services without calling the project's own API over HTTP for internal operations.
- Keep business logic in appropriate backend services rather than controllers, frontend components, or duplicated worker code.
- Put shared request/response types and appropriate Zod schemas in `packages/contracts`.
- Inspect existing modules and patterns before adding or changing backend functionality.

## 2. API Design

- Use the approved REST API prefix `/api/v1`.
- Follow the established response and error format.
- Document API endpoints using the project's Swagger/OpenAPI setup.
- Use the approved offset pagination convention with `page` and `limit`.
- Make filtering and sorting explicit and validate supported values.
- Use appropriate HTTP status codes and avoid exposing stack traces, secrets, or internal implementation details.
- Avoid breaking existing contracts without impact analysis and explicit user approval.

## 3. Validation and Business Rules

- Validate incoming request data at API boundaries using the established validation approach.
- Use shared Zod schemas from `packages/contracts` where appropriate.
- Enforce authorization, role permissions, ownership, and business rules on the server.
- Never rely on frontend checks to enforce important business constraints.
- Keep business rules in reusable services or domain-focused modules, not duplicated across controllers and jobs.
- Handle missing, invalid, conflicting, and stale data explicitly.

## 4. Database and Transactions

- Use PostgreSQL through Prisma within `apps/api`.
- Keep database operations in the established persistence/service boundaries.
- Use transactions when related writes must succeed or fail together.
- Use Prisma migrations for schema changes and review their effect on existing data.
- Consider concurrent requests when enforcing limits, booking conflicts, offer acceptance, or other shared-state rules.
- Do not assume that a prior availability check remains valid; enforce critical constraints atomically where required.
- Do not access the database directly from `apps/web` or create a second independent database access path in `apps/worker`.

## 5. Authentication and Authorization

- Follow the approved Auth.js Credentials integration for website authentication and the backend's established authentication contract.
- Verify authenticated identity and permissions on every protected operation.
- Enforce the user's role and resource ownership on the server.
- Never trust role, account identity, price, status, or permission values supplied by the client without server-side verification.
- Keep authentication and authorization behavior consistent across HTTP endpoints and background processing.

## 6. Background Jobs

- Use the approved pg-boss-based worker architecture.
- Keep jobs safe to retry where practical; avoid duplicate side effects.
- Design job payloads with validated, minimal data and avoid including secrets or unnecessary personal information.
- Handle failures, retries, and terminal errors according to the project's established job configuration.
- Make job processing observable through structured logs and useful operational status.
- Do not move work to the worker solely to avoid implementing correct API validation or authorization.

## 7. Errors, Logging, and Operations

- Use the approved Pino logging approach and include useful context without logging passwords, tokens, private identity data, or sensitive request contents.
- Return consistent, safe API errors while retaining appropriate diagnostic details in protected server logs.
- Apply configured rate limits to sensitive endpoints and authentication-related operations.
- Keep CORS restricted to approved origins before production.
- Follow the approved security requirements and `docs/security.md`.
- Handle external-service failures and timeouts without exposing internal details to users.

## 8. Configuration and Dependencies

- Read configuration from environment variables through the project's established configuration mechanism.
- Validate required configuration at startup where practical.
- Never commit secrets or put server-only credentials in frontend-visible variables.
- Inspect installed versions and existing scripts before using framework-specific features or commands.
- Request approval before adding, removing, or replacing dependencies.

## 9. Backend Verification

- Test validation, authorization, role boundaries, business rules, failure cases, and relevant concurrency scenarios.
- Verify API contracts and Swagger/OpenAPI output for changed endpoints.
- Run relevant type checks, linting, tests, and builds supported by the actual workspace configuration.
- Verify worker retry and idempotency behavior for changed jobs where practical.
- Report checks that were not run, known limitations, and unresolved risks.
