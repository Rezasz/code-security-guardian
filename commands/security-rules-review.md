---
description: Audit the current rule set for staleness, redundancy, and coverage gaps. Shows what rules are enabled, their false-positive rates, and what might be missing.
---

Perform a rules audit using the rule management and learning tools.

Steps:
1. Run `${CLAUDE_PLUGIN_ROOT}/bin/codesec-rules stats` to get counts by category, severity, and source.
2. Run `${CLAUDE_PLUGIN_ROOT}/bin/codesec-learn analyze` to get feedback summary (if feedback-log.jsonl is non-empty).
3. Read `rules/core/` to understand what's currently shipped.
4. Check `rules/community/` and `rules/local/` for any custom or proposed rules.

Then provide a structured audit covering:
- **Coverage gaps**: vulnerability classes not currently covered by any rule
- **High false-positive rules**: rules where dismiss rate exceeds 50% (from feedback)
- **Stale rules**: rules referencing deprecated APIs or patterns no longer seen in modern code
- **Redundant rules**: rules with overlapping patterns that could be merged
- **Suggested additions**: specific rule IDs that should be added based on common vulnerabilities in detected stacks

Conclude with a prioritized action list: rules to retire, consolidate, or add.
