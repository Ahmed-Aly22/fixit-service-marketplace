# Skill: Testing and Review

## Purpose

Verify an approved change against its acceptance criteria and identify regressions before handoff.

## Workflow

1. Read `docs/testing-strategy.md` and the relevant numbered business rules.
2. Inspect project scripts and installed tooling; do not assume commands exist.
3. Identify changed behavior, edge cases, permissions, and affected integrations.
4. Run the smallest useful focused tests first.
5. Run relevant type checks, linting, integration tests, end-to-end tests, or builds when configured and warranted.
6. Review the diff for security, maintainability, architecture boundaries, and unrelated changes.
7. Check that documentation and requirements traceability remain accurate.
8. Report each check and its actual result.

## Rules

- Never claim a test passed if it was not run successfully.
- Do not delete or weaken a test just to get a passing result.
- If a check cannot run, state why and what remains unverified.
- Do not test against production data or use real sensitive customer documents.
