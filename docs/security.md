# Security and Data Protection

## 1. Core Principles

- Treat security and privacy as requirements throughout implementation, not as a final step.
- Enforce critical controls on the server.
- Follow applicable law and platform requirements; obtain appropriate review before production launch.
- Do not claim legal compliance without a qualified review.

## 2. Authentication and Authorization

- Authenticate protected requests and enforce role and resource ownership on the server.
- Do not trust client-provided role, identity, status, price, or permission fields.
- Apply least privilege to administrative actions and internal services.
- Protect account recovery and phone verification against abuse.
- Follow the approved Auth.js Credentials integration and backend authentication contract.

## 3. Secrets and Configuration

- Keep passwords, tokens, API keys, database credentials, and private keys out of source control.
- Use environment variables or an approved secrets mechanism.
- Never expose server-only secrets through client-visible configuration.
- Validate required configuration and fail safely when critical settings are missing.

## 4. Personal and Sensitive Data

- Minimize collection, retention, logging, and exposure of personal information.
- Protect worker identity documents and other private uploads from public access.
- Store private files outside public static directories and use authorized access paths.
- Restrict admin and staff access to sensitive data by role and operational need.
- Do not include sensitive documents or unnecessary personal information in background-job payloads.

## 5. Input and API Security

- Validate all untrusted input.
- Enforce authorization and business rules for every protected operation.
- Configure CORS to approved origins before production.
- Use Helmet or the project's approved HTTP security-header setup.
- Apply rate limits to authentication, recovery, and other sensitive operations.
- Use safe error responses; never expose stack traces or internal secrets.

## 6. Database and Concurrency

- Use Prisma migrations for schema changes.
- Use transactions and database constraints where appropriate to preserve consistency.
- Consider concurrent requests for offers, bookings, active-job limits, and status transitions.
- Never use production data for tests unless a separately approved, controlled process explicitly permits it.

## 7. Logging and Monitoring

- Use structured Pino logging.
- Do not log passwords, tokens, identity-document contents, or unnecessary sensitive request bodies.
- Keep diagnostic information in protected server logs.
- Log meaningful security and operational events without exposing private information.

## 8. Uploads

- Validate allowed file types, size limits, and upload metadata on the server.
- Do not trust file extensions or client-supplied MIME types alone.
- Generate safe storage names and prevent path traversal.
- Serve private files only after authorization.
- Apply additional scanning or processing before production if required by the final threat assessment.

## 9. Verification and Incident Handling

- Test authorization failures, ownership boundaries, input validation, upload access, and safe error handling.
- Review dependencies and configuration for known risks using available tooling.
- Document security-sensitive incidents and follow the approved response process.
- Request approval before weakening or bypassing a security control.
