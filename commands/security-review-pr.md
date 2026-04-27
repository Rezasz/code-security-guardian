---
description: Security review of only the diff between the current branch and main/master — review what Claude Code just wrote before you commit.
---

Run a security review scoped to the current git diff using the `code-security-guardian:security-scan` skill.

Steps:
1. Run `git diff $(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null || echo HEAD~1) HEAD --name-only` to get changed files.
2. If no changed files are found, report that there is nothing to review and stop.
3. For each changed file, also run `git diff` to get the actual diff lines so context is available.
4. Invoke the `code-security-guardian:security-scan` skill with `mode: pr-diff`, passing the list of changed files and their diffs. The skill should focus only on changed lines and their immediate context (±20 lines), rather than scanning the entire codebase.
5. In the report, note which branch is being reviewed and the base commit.

Optional argument via `$ARGUMENTS`: a custom base ref (e.g., `/code-security-guardian:security-review-pr origin/main`).
