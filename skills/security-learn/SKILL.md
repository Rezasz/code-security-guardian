---
name: code-security-guardian:security-learn
description: >
  Review feedback, improve the scanner, analyze why a rule keeps flagging, suppress a noisy rule,
  tune rules, make it learn, self-improve, reduce false positives, the scanner is too noisy,
  make the scanner smarter. Trigger on: "review feedback", "improve the scanner", "why does X
  keep flagging", "suppress this rule", "scanner is too noisy", "make it learn", "self-improve",
  "tune the rules", "reduce false positives", "learn from my decisions".
---

# Security Learn Skill

You analyze scan feedback to make the scanner smarter. You cluster dismiss patterns, identify rules with high false-positive rates, and generate concrete proposals to narrow or suppress them — without touching `rules/core/` (which is immutable).

## Workflow

### Step 1 — Read Recent Feedback

Run `${CLAUDE_PLUGIN_ROOT}/bin/codesec-learn analyze` and capture its output.

If the feedback log is empty, tell the user:
> No feedback recorded yet. Run a scan and respond to the "Is this a real issue?" prompts at the end, then run this command again.

### Step 2 — Cluster by Rule

For each rule with ≥ `min_occurrences_for_proposal` entries (from `config/settings.yaml`, default 5):

| Metric | Meaning |
|--------|---------|
| accept_rate | Fraction of times this rule was accepted as a real issue |
| dismiss_rate | Fraction of times dismissed |
| top_dismiss_reasons | Most common user-provided reasons for dismissal |

### Step 3 — Generate Proposals

Based on the clusters, generate numbered proposals:

**For rules with dismiss_rate > 0.7 (70%):**
- If dismiss reasons suggest a specific pattern (e.g., "Stripe publishable key"): propose adding to `memory/false-positive-patterns.yaml` with a path or content pattern.
- If dismiss reasons suggest the rule is too broad: propose adding to `false_positive_hints` in the rule YAML.
- If 90%+ of firings are dismissed with no clear exception: propose disabling the rule (`codesec-rules disable`).
- If the rule fires correctly sometimes but noisily: propose lowering severity.

**For rules with accept_rate > 0.9 (90%) and ≥10 occurrences:**
- Propose elevating `confidence` to `high` in the rule YAML.
- If the rule is currently `medium` severity but always leads to a fix: propose elevating to `high`.

**For path-based patterns (e.g., user always dismisses a rule in `tests/`):**
- Propose adding the path to `memory/project-context.yaml` → `conventions.ignored_paths`.

**For missed findings (user recorded a finding with `decision: accept` that no rule caught):**
- Propose a new rule in `rules/local/`. Draft it using the pattern from the recorded reason.

### Step 4 — Present Proposals

Present proposals as a numbered list. For each:

```
[1] Narrow SECRETS_OPENAI_KEY — dismiss rate 82% (11/13 occurrences)
    Top reason: "Stripe publishable key in src/billing/checkout.tsx"
    Proposed action: Add false-positive pattern for pk_live_ prefix
    Command: codesec-learn apply --proposal 1
    Preview:
      memory/false-positive-patterns.yaml:
        - rule_id: SECRETS_OPENAI_KEY
          path_pattern: "src/billing/*"
          content_pattern: "pk_live_"
          reason: "Stripe publishable key"
          added: 2025-04-27
          source: auto-learned
```

### Step 5 — Apply on Confirmation

When the user says "apply 1", "apply 1, 3", or "apply all":

Run `${CLAUDE_PLUGIN_ROOT}/bin/codesec-learn apply --proposal <N>` for each confirmed proposal.

Confirm what was changed after applying.

### Step 6 — Update Project Context

If you found consistent path-based dismissals (e.g., `tests/`, `examples/`, `fixtures/`), propose adding those to `memory/project-context.yaml` → `conventions.ignored_paths`. This will cause the scanner to skip those directories entirely in future scans.

---

## Privacy Note

All analysis is local. `feedback-log.jsonl` stores hashed file paths and snippet hashes — never raw source code. Project ID is a hashed git remote URL. No data leaves the machine unless the user explicitly runs `codesec-learn export`.
