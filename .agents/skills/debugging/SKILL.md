# Skill: Debugging

## Purpose

Find and fix the root cause of a reported issue without introducing unrelated changes.

## Workflow

1. Read the relevant instructions and verify `STATE.md` against the actual project.
2. Reproduce the issue when possible and capture the exact error, steps, and expected behavior.
3. Inspect relevant code, logs, recent changes, configuration, and related tests.
4. Form a small set of hypotheses and test them with safe, focused checks.
5. Explain any risky or broad fix and request approval before changing scope or architecture.
6. Implement the smallest root-cause fix within the approved phase.
7. Add a regression test when practical.
8. Run focused checks and relevant wider verification.
9. Review the diff and report the cause, fix, checks, and remaining uncertainty.

## Safety

- Preserve user changes and existing data.
- Do not delete files, reset Git history, disable security, or change production data to make the error disappear.
- Do not mask an error with a broad catch or silent fallback unless that behavior is explicitly intended.
- Separate pre-existing failures from regressions caused by the fix.
