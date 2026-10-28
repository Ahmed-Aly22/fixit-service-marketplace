# Service Marketplace

Bilingual service marketplace MVP for Cairo and Giza, Egypt.

## Project Documentation

Start with:
1. `AGENTS.md` — agent workflow and approval rules.
2. `CONTEXT.md` — project overview and approved architecture.
3. `STATE.md` — progress and next steps.
4. `docs/architecture.md` — application boundaries and technical direction.
5. `docs/business-rules.md` — approved business rules.
6. `docs/ui-design.md` — approved visual design system.

## Intended Workspace

- `apps/web`: Next.js App Router frontend.
- `apps/api`: NestJS REST API.
- `apps/worker`: pg-boss background worker.
- `packages/contracts`: shared contracts and schemas.

This documentation does not prove that the application scaffold or all folders already exist. Inspect the repository before running setup or development commands.

## Setup

1. Install a compatible Node.js version and pnpm version based on the repository configuration.
2. Review the root `package.json`, `pnpm-workspace.yaml`, package-level scripts, and environment examples.
3. Configure local environment variables using documented placeholders; never commit secrets.
4. Start a local PostgreSQL instance using the project's approved development setup.
5. Follow the actual Prisma migration workflow and package scripts.
6. Run checks using commands defined by the repository.

Do not guess scripts or run destructive database commands. Do not run migrations against production as part of local setup.

## Environment Variables

Document required variable names and safe example values in an `.env.example` file when the application scaffold exists. Keep real credentials in local ignored environment files or an approved secrets manager.

## Testing

Use the commands configured in the actual workspace. See `docs/testing-strategy.md`. Do not report checks as passing unless they have been executed successfully.

## Security

See `docs/security.md`. Production launch requires a separate deployment and security-readiness review.
