# API Design

## 1. Principles

- Use REST endpoints under `/api/v1`.
- Keep controllers focused on HTTP concerns and move reusable business behavior into services.
- Validate every untrusted input at the API boundary.
- Enforce authentication, authorization, role, ownership, and business constraints on the server.
- Document endpoints with the project's Swagger/OpenAPI setup.
- Use shared request/response contracts from `packages/contracts` where appropriate.

## 2. URL and Resource Conventions

- Use resource-oriented nouns for paths.
- Use HTTP methods consistently: GET to read, POST to create or trigger an action, PATCH for partial updates, and DELETE only where deletion is an approved behavior.
- Use explicit action endpoints when a business transition cannot be expressed clearly as a normal resource update.
- Avoid exposing internal database structure as a public contract.

## 3. Versioning

The initial API prefix is `/api/v1`. Breaking changes require impact analysis and explicit approval. Prefer additive changes when compatible with the existing contract.

## 4. Responses and Errors

- Use the response envelope and error format established by the implementation.
- Use appropriate HTTP status codes.
- Return stable, actionable error codes/messages without stack traces, secrets, or internal implementation details.
- Keep diagnostic details in protected server logs.
- Do not invent a second response convention in a new module.

## 5. Pagination, Filtering, and Sorting

- Use offset pagination with `page` and `limit`.
- Validate pagination values and enforce a reasonable maximum page size.
- Define supported filters and sort fields explicitly.
- Do not interpolate untrusted filter or sort input into raw database queries.
- Return pagination metadata consistently according to the established API contract.

## 6. Validation and Authorization

- Validate request parameters, query values, and bodies.
- Use shared Zod schemas where appropriate and align them with backend validation.
- Never trust client-provided roles, owner IDs, status values, or permissions.
- Check resource ownership and role permissions for every protected operation.
- Keep critical business rules on the server.

## 7. Documentation

- Add or update Swagger/OpenAPI documentation for changed endpoints.
- Document required authentication, request parameters, response shapes, and expected errors.
- Keep `packages/contracts` and endpoint documentation aligned.
- Update tests and `docs/requirements-traceability.md` when behavior changes.
