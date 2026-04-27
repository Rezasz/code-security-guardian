---
description: Review recent scan feedback and propose updates to scanner rules and false-positive suppressions. Makes the scanner smarter based on your accept/dismiss decisions.
---

Invoke the `code-security-guardian:security-learn` skill.

This command analyzes your recent scan feedback stored in `memory/feedback-log.jsonl` and proposes:
- Narrowing rules that fire too many false positives
- Elevating confidence on rules you consistently accept
- Adding path-scoped suppressions for patterns you always dismiss
- New rules based on issues you manually flagged as missed

No arguments required. Run after accumulating several scan sessions worth of feedback.
