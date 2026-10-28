# Skill: Database Changes

## Purpose

Safely evolve the Prisma data model and database behavior without losing data or breaking application boundaries.

## Workflow

1. Read `docs/architecture.md`, relevant business rules, and the current Prisma schema.
2. Inspect migrations, data access patterns, contracts, API consumers, and worker jobs affected by the change.
3. Define the schema change, compatibility needs, rollout concerns, and tests.
4. Request approval for the implementation phase before changing project files.
5. Create a Prisma migration using the project's verified workflow.
6. Update backend services, validation, contracts, and tests in the approved scope.
7. Consider existing rows, constraints, indexes, transaction behavior, and concurrent operations.
8. Run relevant migration validation and tests against an isolated development/test database.
9. Review the migration and report risks and verification results.

## Safety

- Never manually alter production schema or data without an approved procedure.
- Never reset or drop a database to solve a migration issue without explicit approval.
- Do not put database access in `apps/web`.
- Ensure schema changes do not silently break API contracts or background jobs.
