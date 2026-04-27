# Feedback Schema & Privacy Documentation

---

## feedback-log.jsonl

One JSON object per line. Appended by `codesec-learn record` and by the scan workflow when the user responds to finding prompts.

### Schema

```json
{
  "timestamp": "2025-04-27T10:30:00Z",
  "rule_id": "SECRETS_OPENAI_KEY",
  "file_hash": "sha256:a1b2c3d4...",
  "snippet_hash": "sha256:e5f6a7b8...",
  "decision": "dismiss",
  "reason": "Stripe publishable key — safe to expose",
  "project_id": "sha256:9c8d7e6f...",
  "scan_id": "sha256:1a2b3c4d..."
}
```

### Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | ISO-8601 string | When the feedback was recorded |
| `rule_id` | string | The rule that fired (e.g., `SECRETS_OPENAI_KEY`) |
| `file_hash` | string | SHA-256 of the file path (not the file contents) |
| `snippet_hash` | string | SHA-256 of the matched line/snippet (not the raw text) |
| `decision` | enum | `accept`, `dismiss`, `fix`, or `skip` |
| `reason` | string | Optional free-text reason the user gave |
| `project_id` | string | SHA-256 of the git remote URL (or "unknown") |
| `scan_id` | string | SHA-256 of scan timestamp+path (groups a scan session) |

### Why Hashing?

- `file_hash`: allows the learning system to recognize the same file across scans without storing the path (which may contain usernames or project names).
- `snippet_hash`: allows pattern matching on what code triggered the rule without storing the actual source code.
- `project_id`: allows per-project statistics without storing the repository URL.

**The raw file path and source code are never stored in feedback-log.jsonl.**

---

## findings-log.jsonl

One JSON object per line. Appended after each scan with the full finding output (including file paths). This log is local-only and never exported.

### Schema

```json
{
  "timestamp": "2025-04-27T10:30:00Z",
  "scan_id": "sha256:...",
  "rule_id": "BACKEND_PYTHON_EVAL",
  "severity": "critical",
  "file": "/absolute/path/to/file.py",
  "line": 42,
  "snippet": "eval(user_input)"
}
```

---

## research-cache.jsonl

One JSON object per line. Tracks all fetched advisory and research items to avoid re-fetching.

### Schema

```json
{
  "timestamp": "2025-04-27T09:00:00Z",
  "source": "https://github.com/advisories/GHSA-...",
  "id": "GHSA-xxxx-yyyy-zzzz",
  "title": "Remote code execution in example-lib",
  "relevant": true,
  "rule_proposed": "BACKEND_NODE_EXAMPLE_RCE",
  "stacks": ["node"],
  "cvss": 9.8
}
```

---

## Proposal Format (in-memory, not persisted)

When `codesec-learn analyze` generates proposals, they are numbered in-memory for the session. Each proposal has:

```json
{
  "id": 1,
  "type": "add-fp-pattern | disable-rule | lower-severity | add-ignore-path | propose-new-rule | elevate-confidence",
  "rule_id": "SECRETS_OPENAI_KEY",
  "evidence": {
    "total_occurrences": 13,
    "dismiss_rate": 0.82,
    "top_reasons": ["Stripe publishable key"]
  },
  "action": {
    "target_file": "memory/false-positive-patterns.yaml",
    "change": { ... }
  }
}
```

---

## Apply Pipeline

When `codesec-learn apply --proposal N` is called:

1. Locate proposal N in the in-memory proposal list.
2. Validate the action (check target file exists, YAML is valid after change).
3. Write the change.
4. Log the application to a `memory/applied-proposals.jsonl` file for auditability.
5. Print a diff-style summary of what changed.

**Nothing is auto-applied without explicit user confirmation.** Even with `auto_apply_high_confidence_suppressions: true` in settings, the user still sees the proposal and must say "apply" or "apply all".

---

## Privacy Guarantees

| What | Stored | Where |
|------|--------|-------|
| File paths | Hashed only | feedback-log.jsonl |
| Source code snippets | Hashed only | feedback-log.jsonl |
| Full findings (file path + snippet) | Plaintext | findings-log.jsonl (local only) |
| Git remote URL | Hashed only | feedback-log.jsonl |
| User's dismiss reasons | Plaintext | feedback-log.jsonl |
| Fetched advisories | Plaintext URLs + titles | research-cache.jsonl |

**Nothing is sent to any remote server.** The `community_rules_url` in settings is only fetched when the user explicitly runs `codesec-rules sync`. No telemetry, no analytics, no beaconing.

To review what would be shared before exporting:
```bash
bin/codesec-learn export --dry-run
```
