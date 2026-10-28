# Agent Instructions

## 1. Start Here

Before working on a task:

1. Read `CONTEXT.md` to understand the project scope and approved architecture.
2. Read `STATE.md` and verify the actual project state before resuming ongoing work.
3. Read the relevant business rules, documentation, and applicable agent rules.
4. Use relevant Skills when their workflows apply.
5. Inspect the existing code and configuration before deciding how to implement a change.

## 2. Instruction Priority

Follow this priority order:

1. System and platform instructions.
2. Explicit user instructions and approvals.
3. Applicable project Rules.
4. `CONTEXT.md` and approved project decisions.
5. Relevant Skills and task-specific guidance.

If instructions conflict, apply the priority order. If a conflict remains unresolved, stop and ask the user. Never bypass security requirements or explicit approval gates.

## 3. Plan Before Implementation

For each task:

1. Understand the goal, scope, dependencies, and acceptance criteria.
2. Prepare a short, clear implementation plan.
3. Identify important missing decisions or risks.
4. Request approval before starting each implementation phase.
5. Execute only the approved phase, verify the result, and report the outcome.
6. Request approval before proceeding to the next phase.

Do not create or modify project files until the user approves the applicable plan or phase.

## 4. Scope and Approval

- Stay within the approved task scope.
- Make routine technical decisions independently when they do not change approved decisions.
- Explain important risks and request approval before changing approved architecture, business rules, or significant UX behavior.
- Ask before adding, removing, or replacing dependencies.
- Never modify agent instruction files without explicit user approval.
- Do not perform destructive operations or push Git changes without explicit approval.
- Preserve existing user changes and unrelated work.

## 5. Verification and Review

- Inspect changes for correctness, security, maintainability, architecture compliance, and unintended side effects.
- Run relevant checks using commands verified against the actual project configuration.
- Fix failures introduced by the current changes when they are within scope.
- Do not claim that checks passed if they were not run successfully.
- Report relevant checks that could not be performed and any remaining issues.

## 6. Documentation and Progress

- Update relevant documentation when approved changes affect durable project knowledge.
- Update `STATE.md` at major phases and significant milestones.
- Keep authoritative information in its designated document and avoid unnecessary duplication.
- When resuming work, compare the recorded state with the actual project state.

## 7. Git Workflow

- Review the changes before committing.
- Create a local commit automatically only after the required checks pass and the changes are reviewed.
- Do not commit if required checks fail.
- Never push without explicit user approval.
- Ask before rollback or history-changing operations.

## 8. Completion Report

At the end of each phase, briefly report:

- What was completed.
- The key files changed.
- The checks performed and their results.
- Any blockers or remaining work.

Then request approval before continuing to the next phase.
