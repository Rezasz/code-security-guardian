---
description: Run a full security scan on a directory or file. Usage: /code-security-guardian:security-scan [path] [--stack <hint>]
---

Run a comprehensive security review on the target path using the `code-security-guardian:security-scan` skill.

Target path: `$ARGUMENTS` (defaults to `.` if not provided — the current working directory).

Parse `$ARGUMENTS` to extract:
- The target path (first positional argument, or `.` if absent)
- An optional `--stack <hint>` flag (e.g., `--stack react,python`) that narrows stack detection

Then invoke the `code-security-guardian:security-scan` skill, passing the resolved path and any stack hint.
