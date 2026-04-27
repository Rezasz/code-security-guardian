---
description: Research new vulnerabilities, CVEs, and security techniques relevant to this project's stack, then propose new scanner rules. Usage: /code-security-guardian:security-research [topic|CVE-ID|since:YYYY-MM-DD]
---

Invoke the `code-security-guardian:security-research` skill.

Parse `$ARGUMENTS` to extract:
- An optional topic (e.g., "JWT bypass", "prototype pollution")
- An optional CVE ID (e.g., "CVE-2024-1234")
- An optional date filter (e.g., "since:2025-01-01")

Pass these as context when invoking the skill. If no arguments are provided, the skill will infer the relevant stack from project context and research broadly.
