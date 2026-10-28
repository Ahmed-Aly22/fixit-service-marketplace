# Skill: Feature Implementation

## Purpose

Implement an approved feature in small, verifiable phases while preserving business rules and architecture.

## Workflow

1. Read `AGENTS.md`, `CONTEXT.md`, `STATE.md`, and relevant Rules.
2. Read the relevant sections of `docs/business-rules.md`, `docs/modules-overview.md`, and `docs/requirements-traceability.md`.
3. Inspect existing code, contracts, tests, and configuration.
4. Define scope, acceptance criteria, dependencies, risks, and a short plan.
5. Request approval for the phase before creating or modifying project files.
6. Implement the smallest complete vertical slice within the approved phase.
7. Add or update shared contracts, validation, authorization, and tests as needed.
8. Update relevant documentation and traceability.
9. Run focused checks, then broader relevant checks when appropriate.
10. Review the diff and report results before asking approval to continue.

## Implementation Guidance

- Follow existing conventions and the approved architecture.
- Keep critical business logic on the server.
- Keep frontend, API, and worker responsibilities separate.
- Do not add dependencies without approval.
- Do not implement deferred features unless explicitly approved.
- If a missing decision affects correctness, stop at that part and ask instead of guessing.

## Completion Report

Report the behavior implemented, files changed, business rules affected, checks and results, remaining risks, and next proposed phase.
