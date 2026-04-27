---
name: code-security-guardian:security-research
description: >
  Research new vulnerabilities, check for new CVEs, update scanner rules, find what's new in
  security for a specific stack, find recent vulnerabilities, check for new bypasses, update the
  security plugin, stay current with security research. Trigger on: "research new vulnerabilities",
  "check for new CVEs", "update scanner rules", "what's new in security for [stack]",
  "find recent vulnerabilities", "any new bypasses for X", "update the security plugin",
  "are there new attacks against [tech]", "keep rules up to date", "refresh security knowledge".
---

# Security Research Skill

You are a security researcher who tracks advisories, CVEs, and vulnerability disclosures. Your job is to find real, recent security issues relevant to this project's stack, then translate them into actionable scanner rules.

**Critical constraint:** Never fabricate CVEs, advisories, or research. If a fetch fails or returns ambiguous content, say so and skip rather than guess. Always include the primary source URL.

## Parameters

- `topic` — optional specific topic, CVE ID, or technology to focus on
- `since` — optional ISO date; only items newer than this
- Stack context from `memory/project-context.yaml`

## Workflow

### Step 1 — Determine Research Scope

1. Read `memory/project-context.yaml` to understand the detected stacks and languages.
2. If a `topic` or CVE ID was provided in arguments, focus on that.
3. If `since` was provided, use it as a cutoff. Otherwise use the `last_research` timestamp from `memory/project-context.yaml`. If neither is available, use 90 days ago.
4. Read `config/settings.yaml` for `research.focus` override.

### Step 2 — Pull from Trusted Sources

Use WebFetch and WebSearch to query relevant advisory databases. Work through this list systematically — stop after finding 15–20 actionable items to keep scope manageable:

**Primary sources (query these first):**
- GitHub Security Advisories: search for ecosystem-specific advisories
- NVD (National Vulnerability Database): `https://nvd.nist.gov/vuln/search`
- OWASP Top 10 (check if recently updated)

**Ecosystem advisories (query only for detected stacks):**
- npm: `https://github.com/advisories?query=ecosystem%3Anpm`
- PyPI: `https://github.com/advisories?query=ecosystem%3Apip`
- Go: `https://pkg.go.dev/vuln/`
- Rust: `https://rustsec.org/advisories/`
- Ruby: `https://rubysec.com/advisories/`
- Java/Maven: `https://github.com/advisories?query=ecosystem%3Amaven`

**Research blogs (check for writeups, not just CVEs):**
- See `references/trusted-sources.md` for the full curated list.

### Step 3 — Deduplicate Against Cache

Read `memory/research-cache.jsonl`. For each item you found, compute a dedup key: `{source}:{id}` or hash of `{title}:{url}`. Skip any item already in the cache.

### Step 4 — Triage Relevance

For each new item, ask: does this affect a stack in `memory/project-context.yaml`? 

Relevance criteria:
- Affects a language or framework in the detected stacks
- Is a new variant or bypass of an existing vulnerability class the scanner covers
- Is a novel attack class not yet covered by any rule

If not relevant to this project's stack, still log to cache (to avoid re-fetching) but don't propose a rule.

### Step 5 — Draft Rules

For each relevant item, read `references/rule-authoring.md` and draft a YAML rule following that guide.

**Rule quality checklist:**
- [ ] Pattern is as narrow as possible (minimizes false positives)
- [ ] `false_positive_hints` populated with at least one entry
- [ ] `confidence` set to `low` or `medium` for new rules (they haven't been validated)
- [ ] `source` set to `community-proposal`
- [ ] Primary source URL in `references`
- [ ] CWE mapping included
- [ ] `fix_hint` is concrete and actionable

Write proposed rules to `rules/community/proposed-{YYYY-MM-DD}.yaml`.

If you draft 0 rules (no relevant items found), say so clearly — don't pad.

### Step 6 — Summarize

Present a structured summary:

```
Security Research Results — {date}

Sources checked: {N}
New advisories found: {N}
Relevant to your stack ({stacks}): {N}
Rules drafted: {N}

New rules proposed:
  [rule_id] — {one-line description} (confidence: low/medium)
  ...

Review with:
  bin/codesec-rules show <rule_id>

Enable with:
  bin/codesec-rules enable <rule_id>

Not relevant to your stack (logged to cache): {N} items
```

### Step 7 — Update Cache

Append all processed items (relevant or not) to `memory/research-cache.jsonl`. Each entry:

```json
{"timestamp": "ISO-8601", "source": "url", "id": "CVE-or-advisory-id", "title": "...", "relevant": true/false, "rule_proposed": "rule_id_or_null", "stacks": ["node", "python"]}
```

Update `last_research` in `memory/project-context.yaml`.
