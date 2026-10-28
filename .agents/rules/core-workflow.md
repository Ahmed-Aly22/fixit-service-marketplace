# Core Workflow Rules

## 1. Before Starting

- Read `AGENTS.md`, `CONTEXT.md`, and the relevant parts of `STATE.md`.
- Verify the actual project state before resuming existing work.
- Inspect relevant code, configuration, documentation, and applicable Rules.
- Identify task scope, dependencies, acceptance criteria, and important uncertainties.
- Do not assume that documented progress matches the actual codebase without verification.

## 2. Plan and Approval Gates

- Prepare a concise plan with clear, verifiable steps before implementation.
- Request approval before starting each phase.
- Implement only the currently approved phase.
- After each phase, review the changes, run relevant checks, and report the results.
- Request approval before starting the next phase.
- Approval of a plan does not automatically authorize every later phase.

## 3. File Creation and Instruction Changes

- Do not create or modify project files before the user approves the applicable plan or phase.
- Never create or modify `AGENTS.md`, `CONTEXT.md`, Rules, or Skills without explicit approval for the specific file and proposed content.
- Present each new instruction file for review and obtain approval before creating it.
- Do not silently rewrite approved instructions or decisions.

## 4. Scope and Decisions

- Stay within the approved scope and avoid unrelated changes.
- Make routine technical decisions independently when they remain within approved requirements and architecture.
- If a requirement or approved decision needs to change, analyze the impact and request approval before implementation.
- For important missing business, architecture, or UX decisions, explain the options and wait for the user's decision.
- For incomplete requirements, proceed only with clear, independent parts; stop at the affected part if clarification is required.
- If a conflict remains unresolved after applying instruction priority, stop and ask the user.

## 5. Dependencies and Commands

- Inspect existing dependencies, scripts, workspace configuration, and tool versions before choosing commands or solutions.
- Request approval before adding, removing, or replacing dependencies.
- Run safe, non-destructive commands when appropriate.
- Obtain explicit approval before destructive operations, data-loss commands, or risky environment changes.
- Never use a destructive command to resolve uncertainty about project state.

## 6. Implementation and Parallel Work

- Follow the approved architecture and use existing patterns where suitable.
- Break large tasks into small, verifiable subtasks.
- Verify prerequisites before starting dependent work.
- Parallelize only independent tasks when doing so is safe.
- Preserve existing user changes and avoid unrelated refactoring.
- If a blocker prevents safe progress, stop, preserve changes, and explain what is needed.

## 7. Testing and Review

- Inspect the impact of each phase and run relevant checks based on the actual project configuration.
- Fix failures introduced by the current changes when they are within scope.
- Do not expand the task to fix unrelated legacy issues; report them separately.
- Review changes for correctness, security, architecture compliance, maintainability, and unintended side effects.
- Never claim that a test or check passed unless it actually ran successfully.
- Report checks that could not be performed and any remaining risks.

## 8. Documentation and State

- Keep authoritative information in its designated document.
- Update relevant documentation when approved changes alter durable project knowledge.
- Update `STATE.md` at major phases and significant milestones, including completed work, current phase, next steps, blockers, and verification status.
- On resuming work, compare `STATE.md` with the actual project state.
- Do not duplicate detailed business rules, architecture, or UI specifications across multiple documents.

## 9. Git Workflow

- Review the intended changes and check for unrelated or user-owned modifications before committing.
- Create a local commit automatically only after relevant required checks pass and the changes have been reviewed.
- Do not commit when required checks fail.
- Never push without explicit user approval.
- Ask before reverting user changes, rolling back work, or changing Git history.
- Do not run destructive Git commands without explicit approval.

## 10. Completion and Handoff

At the end of each phase, report concisely:

- What was completed.
- Key files changed.
- Checks performed and their results.
- Remaining work, blockers, and verification limits.

Then request approval before continuing to the next phase.
