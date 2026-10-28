# Testing Strategy

## 1. Goals

Verify user-visible behavior, business rules, security boundaries, integration correctness, and regression safety. Use the tools actually configured in the repository; do not assume scripts or dependencies exist.

## 2. Test Layers

- **Unit:** focused services, utilities, validators, and business rules.
- **Integration:** API modules, persistence behavior, contracts, and background processing.
- **Frontend component tests:** use Vitest and React Testing Library where configured.
- **API tests:** use Jest and NestJS testing utilities where configured.
- **End-to-end:** use Playwright for important user journeys where configured.
- **Static checks:** type checking, linting, and build commands defined by the actual workspace.

## 3. Business Rule Coverage

Use `docs/business-rules.md` as the source of truth. Every implemented or changed rule should have appropriate tests. Cover:
- valid and invalid transitions;
- authorization, roles, and resource ownership;
- boundary values and malformed input;
- booking overlap and concurrent offer acceptance;
- active-job limits and cancellation;
- identity-document access and verification;
- complaint privacy and notifications;
- retries and duplicate execution for background jobs.

## 4. Test Data

- Use isolated development/test databases and controlled fixtures.
- Never use real customer identity documents or production credentials.
- Avoid dependence on test ordering or uncontrolled external services.
- Clean up test-created data without deleting unrelated data.
- Use mocks for external services where appropriate, while retaining critical integration coverage.

## 5. Verification Workflow

For each approved implementation phase:
1. Run focused tests for the changed behavior.
2. Run relevant type checks and linting.
3. Run integration or end-to-end tests when available and warranted.
4. Run a build when scope or risk warrants it.
5. Review the diff and ensure unrelated files were not changed.

## 6. Failure Reporting

- Fix failures caused by the current change when within scope.
- Separate pre-existing failures from new regressions.
- Do not remove or weaken tests solely to obtain a passing result.
- Report checks that could not run and explain what remains unverified.
