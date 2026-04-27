# Rule Authoring Guide

How to convert a vulnerability writeup or advisory into a `codesec-scan` YAML rule.

---

## YAML Rule Schema

```yaml
rules:
  - id: CATEGORY_LANG_SHORT_NAME          # ALL_CAPS_SNAKE_CASE, unique
    severity: critical|high|medium|low|info
    category: secrets|xss|sql-injection|command-injection|unsafe-deserialization|
              weak-crypto|authentication|authorization|cors|tls|debug-mode|
              insecure-storage|code-injection|nosql-injection|ci-cd|
              container-security|cloud-misconfiguration|mobile|todo
    cwe: CWE-89                            # or null
    languages: [python, js, go]           # empty list = all
    extensions: [.py, .js]                # empty list = all files
    filename_pattern: '^Dockerfile'       # optional regex on filename only
    pattern: 'regex_here'                 # Python re syntax
    pattern_type: regex                   # always regex for now
    pattern_flags: [IGNORECASE, MULTILINE] # optional
    message: "One sentence: what was found."
    fix_hint: "Concrete fix, ideally with code snippet."
    confidence: high|medium|low           # how often this fires correctly
    false_positive_hints:
      - "Describe a common scenario where this is a false positive"
    references:
      - https://primary-source-url
    enabled: true
    source: core|community-proposal|local  # use community-proposal for new research
    added: 2025-04-27
```

---

## Authoring Principles

### 1. Prefer narrow patterns

Bad (high FP rate):
```yaml
pattern: 'password'
```

Good (narrow):
```yaml
pattern: 'password\s*=\s*["\'][^"\']{8,}["\']'
```

Better (scoped to assignment context):
```yaml
pattern: '(?i)(password|passwd|pwd)\s*=\s*["\'][^"\']{8,}["\']'
```

### 2. Require multiple indicators when possible

Instead of flagging any `exec(`, flag `exec(` with a variable argument:
```yaml
pattern: '\bexec\s*\([^"''\)][^)]*\)'   # not a string literal
```

### 3. Always populate false_positive_hints

Every rule must have at least one. Think: what valid code looks like the vulnerability?

### 4. Default to confidence: low for new research rules

Until a rule has been validated against real codebases and reviewed, keep confidence low. Elevate via `codesec-learn` after feedback.

### 5. Include CWE

Map every rule to a CWE from https://cwe.mitre.org/. Common ones:
- CWE-79: XSS
- CWE-89: SQL Injection
- CWE-78: OS Command Injection
- CWE-798: Hardcoded Credentials
- CWE-502: Unsafe Deserialization
- CWE-295: Certificate Validation
- CWE-347: JWT Signature Verification
- CWE-916: Password Hashing Weakness
- CWE-918: SSRF
- CWE-22: Path Traversal

### 6. Never write to rules/core/

Rules proposed by research go to `rules/community/proposed-{date}.yaml`. Users enable them. Only the plugin maintainer promotes to `rules/core/`.

---

## Worked Examples

### Example 1: Advisory → Rule

**Advisory:** "node-serialize < 0.0.4 allows RCE via IIFE in serialized objects (npm advisory GHSA-...)"

**Pattern to detect:**
```
{"rce":"_$$ND_FUNC$$_function(){require('child_process').exec(...)}()"}
```
The sink is `serialize.unserialize()` called on untrusted data.

**Drafted rule:**
```yaml
- id: BACKEND_NODE_SERIALIZE_RCE
  severity: critical
  category: unsafe-deserialization
  cwe: CWE-502
  languages: [js, ts]
  extensions: [.js, .ts, .mjs]
  pattern: 'require\([''"]node-serialize[''"\])\s*'
  pattern_type: regex
  pattern_flags: []
  message: "node-serialize is known to allow RCE via IIFE deserialization. Replace with a safe serializer."
  fix_hint: "Uninstall node-serialize. Use JSON.parse/stringify or a validated schema library."
  confidence: high
  false_positive_hints:
    - "If the input is fully server-controlled and never user-supplied"
  references:
    - https://github.com/advisories/GHSA-...
  enabled: false
  source: community-proposal
  added: 2025-04-27
```

---

### Example 2: Research Blog → Rule

**Writeup:** "PortSwigger: New prototype pollution gadgets in Express.js apps using `res.json()` with `__proto__` in query params"

**Drafted rule:**
```yaml
- id: WEB_PROTOTYPE_POLLUTION_QUERY
  severity: high
  category: xss
  cwe: CWE-1321
  languages: [js, ts]
  extensions: [.js, .ts]
  pattern: '(?:req\.query|req\.body|req\.params)\s*\.\s*__proto__'
  pattern_type: regex
  pattern_flags: []
  message: "Direct __proto__ access from request data — prototype pollution vector."
  fix_hint: "Use Object.create(null) as base object. Sanitize keys: delete obj.__proto__"
  confidence: medium
  false_positive_hints:
    - "Checking for __proto__ as a guard (if ('__proto__' in obj)) is safe"
  references:
    - https://portswigger.net/research/...
  enabled: false
  source: community-proposal
  added: 2025-04-27
```

---

### Example 3: CVE with PoC → Rule

**CVE-2023-45311:** Python `requests` library — SSRF via `@` in URL allowing credential injection to attacker-controlled host.

Pattern: `requests.get("https://trusted.com@evil.com/")` — the `@` causes the resolver to connect to `evil.com` but display `trusted.com`.

```yaml
- id: BACKEND_PYTHON_SSRF_AT_SIGN
  severity: high
  category: ssrf-variant
  cwe: CWE-918
  languages: [python]
  extensions: [.py]
  pattern: 'requests\.\w+\s*\(\s*[^)]*@[^)]*\)'
  pattern_type: regex
  pattern_flags: []
  message: "URL with @ character in requests call — may allow SSRF via credential injection."
  fix_hint: "Parse the URL with urllib.parse.urlparse() and validate the netloc before fetching."
  confidence: low
  false_positive_hints:
    - "@ in email address used as query parameter, not in hostname"
    - "@ in basic auth for a trusted internal service"
  references:
    - https://nvd.nist.gov/vuln/detail/CVE-2023-45311
  enabled: false
  source: community-proposal
  added: 2025-04-27
```

---

## Anti-Patterns to Avoid

| Anti-pattern | Why bad | Fix |
|-------------|---------|-----|
| `pattern: 'password'` | Fires on comments, variable names | Add context: assignment operator, string literal |
| `pattern: 'http://'` | Too broad | Scope to context where HTTPS is required |
| No `false_positive_hints` | Can't tune FP rate | Always add at least one |
| `confidence: high` on new rule | Overconfident, will confuse users | Default to `low` |
| No `references` | Can't verify | Always link primary source |
| `enabled: true` on proposed rule | Gets applied without review | Default to `enabled: false` |
