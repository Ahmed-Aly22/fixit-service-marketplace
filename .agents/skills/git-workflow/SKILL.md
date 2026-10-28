# Skill: Git Workflow

## Purpose

Keep changes reviewable and protect user-owned work.

## Workflow

1. Inspect `git status` and the current branch before changing files.
2. Review existing user changes and avoid overwriting or including unrelated work.
3. Keep each change set focused on the approved task.
4. Review the diff for accidental secrets, generated files, unrelated edits, and correctness.
5. Run relevant required checks before committing.
6. Create a local commit only after the changes have been reviewed and required checks pass.
7. Report the commit and verification results.

## Approval and Safety

- Never push without explicit user approval.
- Ask before reverting user changes, rolling back work, rebasing shared history, force-pushing, or performing other history-changing operations.
- Do not use destructive Git commands without explicit approval.
- Do not commit secrets, credentials, private documents, or local environment files.
