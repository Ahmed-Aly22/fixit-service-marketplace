# Project State

## Purpose

This file records the actual implementation progress. Update it at significant milestones and verify it against the repository whenever work resumes.

## Current Status

- **Project planning:** approved product scope, architecture direction, and UI design decisions have been recorded in project context and documentation.
- **Instruction bundle:** initial Rules and Skills are prepared as guidance; review the actual repository before assuming they have been installed.
- **Application implementation:** not verified by this document. Inspect the repository and update this section based on actual code.
- **Tests and builds:** not verified here.

## Approved Technical Direction

- pnpm workspaces monorepo.
- `apps/web`: Next.js App Router, React, TypeScript, Tailwind CSS.
- `apps/api`: NestJS REST API and sole database-access application.
- `apps/worker`: pg-boss background processing, reusing appropriate backend services without internal HTTP calls to the project's API.
- `packages/contracts`: shared types and suitable Zod schemas.
- PostgreSQL and Prisma.
- Auth.js Credentials for website authentication.
- Arabic/English, RTL/LTR; responsive website only for the MVP.
- Ordinary service requests only in the MVP; urgent requests and native apps deferred.

## Module Checklist

- [ ] Inspect the repository and confirm the monorepo structure.
- [ ] Confirm workspace package scripts and installed dependency versions.
- [ ] Confirm environment-variable names and safe local setup instructions.
- [ ] Implement and verify account/authentication flows.
- [ ] Implement and verify worker profiles and identity review.
- [ ] Implement and verify service requests.
- [ ] Implement and verify offers and booking/scheduling rules.
- [ ] Implement and verify reviews and complaints.
- [ ] Implement and verify notifications/background jobs.
- [ ] Establish automated testing and requirements traceability.
- [ ] Review deployment readiness and security requirements before launch.

## Current Phase

Instruction and documentation preparation. Confirm the actual repository state before transitioning into implementation.

## Blockers and Open Decisions

- Check prior project discussion and `docs/business-rules.md` before implementing any unresolved business behavior.
- BR-063 is explicitly unresolved and must be clarified before implementing a customer-non-response completion rule.
- Production deployment, data retention, and legal review must be finalized before launch.

## Verification Log

No application checks are claimed as passed by this file. Add dated verification results only after running the relevant commands successfully.
