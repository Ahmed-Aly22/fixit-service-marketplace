# Testing Rules

## 1. Testing Principles

- Follow `docs/testing-strategy.md` and inspect the actual workspace scripts before selecting test commands.
- Test behavior and acceptance criteria, not only implementation details.
- Prefer focused tests that provide clear feedback, then run broader relevant checks.
- Do not claim that tests pass unless they were actually executed successfully.
- Do not change production behavior merely to satisfy a test without understanding the intended behavior.

## 2. Test Layers

- **Unit tests:** Verify focused functions, services, validation, and business rules in isolation.
- **Integration tests:** Verify interactions between modules, persistence, API endpoints, and background-job behavior where appropriate.
- **End-to-end tests:** Verify important user journeys across the website and backend using the project's configured tools.
- **Frontend tests:** Use the configured Vitest and React Testing Library setup where present.
- **API tests:** Use the configured Jest and NestJS testing setup where present.
- **Browser tests:** Use Playwright for critical end-to-end journeys where configured.

Do not assume a tool or script exists; verify the current repository configuration first.

## 3. Business Rule Coverage

- Use `docs/business-rules.md` as the authoritative list of numbered business rules.
- Add or update tests for every affected rule when implementing a relevant change.
- Include boundary conditions, invalid input, permission failures, and error paths.
- For booking and offer flows, consider concurrent requests, schedule conflicts, capacity limits, and state transitions.
- For authentication and account flows, test role boundaries, ownership, and recovery-related cases.
- For complaints and identity verification, test access control and protection of sensitive information.
- Keep `docs/requirements-traceability.md` aligned with implementation and test coverage when approved changes affect the mapping.

## 4. Test Data and Isolation

- Use controlled test data and isolated test environments.
- Never run tests against production data or use real customer identity documents.
- Avoid tests that depend on execution order or external services unless the dependency is deliberately controlled.
- Mock external services when appropriate; retain integration tests for critical integration behavior.
- Ensure tests clean up their own data and do not destroy unrelated developer data.

## 5. Reliability and Security

- Test both successful and unsuccessful paths.
- Verify that protected operations reject unauthorized users and users with the wrong role.
- Check that logs and errors do not expose passwords, tokens, private documents, or unnecessary personal data.
- Test retries and duplicate delivery for background jobs where relevant.
- Avoid weakening validation, authorization, or security controls to make tests pass.

## 6. Required Verification

For each implementation phase:

1. Run focused tests for the changed behavior.
2. Run relevant type checks and linting when configured.
3. Run related integration or end-to-end tests when practical and available.
4. Run a build when the scope or risk warrants it.
5. Review the diff and verify that no unrelated changes were introduced.

Use only commands verified against the actual project configuration.

## 7. Failure Handling and Reporting

- Investigate failures introduced by the current change and fix them when within scope.
- Separate new failures from known pre-existing failures.
- Do not hide, skip, or delete failing tests without a justified and approved reason.
- If a required check cannot run, explain why and state what remains unverified.
- Report the checks performed, their results, and any unresolved risks.
