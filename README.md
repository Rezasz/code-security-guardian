```
  ██████╗ ██████╗ ██████╗ ███████╗    ███████╗███████╗ ██████╗
 ██╔════╝██╔═══██╗██╔══██╗██╔════╝    ██╔════╝██╔════╝██╔════╝
 ██║     ██║   ██║██║  ██║█████╗      ███████╗█████╗  ██║
 ██║     ██║   ██║██║  ██║██╔══╝      ╚════██║██╔══╝  ██║
 ╚██████╗╚██████╔╝██████╔╝███████╗    ███████║███████╗╚██████╗
  ╚═════╝ ╚═════╝ ╚═════╝ ╚══════╝    ╚══════╝╚══════╝ ╚═════╝

  ██████╗ ██╗   ██╗ █████╗ ██████╗ ██████╗ ██╗ █████╗ ███╗   ██╗
 ██╔════╝ ██║   ██║██╔══██╗██╔══██╗██╔══██╗██║██╔══██╗████╗  ██║
 ██║  ███╗██║   ██║███████║██████╔╝██║  ██║██║███████║██╔██╗ ██║
 ██║   ██║██║   ██║██╔══██║██╔══██╗██║  ██║██║██╔══██║██║╚██╗██║
 ╚██████╔╝╚██████╔╝██║  ██║██║  ██║██████╔╝██║██║  ██║██║ ╚████║
  ╚═════╝  ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚═════╝ ╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝
```

# code-security-guardian

**A language-agnostic security review plugin for Claude Code.**

Built by [Reza Sahebozamani](https://github.com/Rezasz) — sahebzamanii@gmail.com

A free gift to the software developer community. If Claude Code wrote it, this plugin reviews it.

---

## Disclaimer

**This software is provided "as is", without warranty of any kind, express or implied.**

Code Security Guardian is a best-effort static analysis tool. It is not a substitute for a professional security audit, penetration test, or legal compliance review. The author makes no representations or warranties that this tool will identify all vulnerabilities, prevent security breaches, or make your software secure.

By using this tool, you agree that:

- The author accepts **no responsibility or liability** for any security incidents, data breaches, damages, or losses arising from use of this tool or reliance on its output.
- Findings are informational only. You are solely responsible for evaluating, validating, and acting on any output.
- This tool does not guarantee your code is free of security vulnerabilities, even if no findings are reported.

For production systems handling sensitive or regulated data, always engage qualified security professionals.

---

## What It Is

Code Security Guardian combines a fast, dependency-free regex scanner (`codesec-scan`) with LLM-powered semantic analysis. The scanner does a broad sweep in milliseconds; Claude confirms each finding with code context, filters false positives, and catches the vulnerability classes no regex can touch — IDOR, SSRF, missing auth checks, and business logic flaws.

It learns from your feedback. The more you use it, the better it gets at knowing what's a real issue in *your* codebase.

---

## Why

AI-generated code ships fast. Security review doesn't always keep up.

This plugin exists because:
- Vibe-coded apps skip security review entirely
- LLMs are fluent at writing SQL injection, XSS sinks, and hardcoded secrets
- Every developer deserves a second set of eyes before shipping

It's not a replacement for a penetration test. It's the thing you run before you even think about a pentest — the fast, always-available, no-excuses security check.

---

## License

MIT License — Copyright (c) 2025 Reza Sahebozamani (sahebzamanii@gmail.com)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

---

## Install

### From the marketplace (recommended)

```
/plugin marketplace add Rezasz/code-security-guardian
/plugin install code-security-guardian@code-security-guardian
```

To get updates later:

```
/plugin marketplace update code-security-guardian
```

### From the zip

```bash
unzip code-security-guardian-plugin.zip
claude --plugin-dir ./code-security-guardian
```

### From source

```bash
git clone https://github.com/Rezasz/code-security-guardian
claude --plugin-dir ./code-security-guardian
```

**Requirements:** Python 3.8+ (stdlib only — zero `pip install` needed)

---

## Quick Start

```
/code-security-guardian:security-scan ./my-app
```

That's it. Claude detects your stack, loads the right rules, runs the scanner, confirms every finding with code context, and writes `SECURITY_REPORT.md`.

---

## How to Use

### 1 — Run a scan

Once installed (see [Install](#install)), inside Claude Code type:

```
/code-security-guardian:security-scan
```

Or scan a specific folder with a stack hint for faster detection:

```
/code-security-guardian:security-scan ./src --stack react,python
```

### 2 — Review what Claude just wrote (before committing)

```
/code-security-guardian:security-review-pr
```

Scans only the current git diff — fast and focused on what changed.

### 3 — Respond to findings

After each scan, Claude walks you through Critical and High findings one by one:

```
[CRIT-01] Hardcoded JWT secret — src/auth/middleware.ts:14
Is this a real issue? [accept / dismiss / fix / skip]
```

Your decisions are logged locally and used to improve the scanner over time.

### 4 — Make it smarter

```
/code-security-guardian:security-learn
```

Clusters your past dismissals and proposes narrowing noisy rules or adding suppressions. Run this every few scans.

### 5 — Stay current on CVEs

```
/code-security-guardian:security-research
```

Fetches new advisories for your detected stack and drafts new scanner rules. To research a specific CVE:

```
/code-security-guardian:security-research CVE-2024-1234
```

### 6 — Optional: inline hook

To get security warnings on every file Claude edits, add the post-edit hook to your Claude Code `settings.json`. See the **Enabling the post-edit hook** section under Configuration for the exact snippet. It runs silently on clean files and only prints Critical/High findings inline.

---

## Commands

| Command | What it does |
|---------|-------------|
| `/code-security-guardian:security-scan [path]` | Full security scan of a directory or file |
| `/code-security-guardian:security-review-pr` | Scan only the current git diff (review what Claude just wrote) |
| `/code-security-guardian:security-research [topic]` | Fetch new CVEs/advisories and draft new scanner rules |
| `/code-security-guardian:security-learn` | Analyze your feedback, propose rule improvements |
| `/code-security-guardian:security-rules-review` | Audit rule coverage gaps and staleness |

### Examples

```
# Scan current directory
/code-security-guardian:security-scan

# Scan a specific path
/code-security-guardian:security-scan ./api

# Scan with stack hint (faster detection)
/code-security-guardian:security-scan ./app --stack react,python

# Review what Claude just wrote before committing
/code-security-guardian:security-review-pr

# Check for new CVEs affecting your stack
/code-security-guardian:security-research

# Research a specific CVE
/code-security-guardian:security-research CVE-2024-1234

# Turn feedback into smarter rules
/code-security-guardian:security-learn
```

---

## CLI Tools

All three tools are pure Python 3, stdlib only, no installation required.

### `bin/codesec-scan`

```bash
# Full scan, human-readable output
bin/codesec-scan --path ./my-app

# Full scan, JSON output (for CI/CD)
bin/codesec-scan --path ./my-app --json

# Quick mode: secrets + injection only, fast
bin/codesec-scan --path ./my-app --quick

# Filter by minimum severity
bin/codesec-scan --path ./my-app --severity high

# Exclude paths
bin/codesec-scan --path ./my-app --exclude "tests/**"

# Skip dependency audit
bin/codesec-scan --path ./my-app --no-deps

# List all 67 built-in rules
bin/codesec-scan --list-rules
```

**Exit codes for CI gating:**

| Code | Meaning |
|------|---------|
| `0` | No findings at medium or above |
| `1` | Medium findings present |
| `2` | High findings present |
| `3` | Critical findings present |

```yaml
# GitHub Actions example
- name: Security scan
  run: python3 bin/codesec-scan --path . --severity high --no-deps
  # exits 2 or 3 if high/critical findings → fails the build
```

### `bin/codesec-rules`

```bash
codesec-rules list [--category xss] [--severity critical] [--source core|community|local]
codesec-rules show <rule-id>
codesec-rules disable <rule-id>   # writes override to rules/local/
codesec-rules enable <rule-id>
codesec-rules add <yaml-file>     # validate + copy to rules/local/
codesec-rules sync                # pull community rules
codesec-rules diff                # show recently synced rule files
codesec-rules validate <yaml-file>
codesec-rules stats
codesec-rules test <rule-id> --on <path>
```

### `bin/codesec-learn`

```bash
codesec-learn record --finding-id <rule-id> --decision accept|dismiss|fix|skip [--reason "text"]
codesec-learn analyze
codesec-learn propose-suppressions
codesec-learn propose-rules
codesec-learn apply --proposal <N>|all
codesec-learn export [--plain]
codesec-learn import <file>
```

---

## What It Scans

| Category | Languages / Stacks |
|----------|-------------------|
| Secrets & credentials | All files (AWS, GCP, GitHub, Stripe, OpenAI, Anthropic, Slack, SendGrid, DB connection strings, JWT secrets, private keys) |
| XSS sinks | JS, TS, JSX, TSX, Vue, Svelte, Angular, HTML |
| SQL injection | Python, Node.js, Go, Java, Ruby, PHP |
| Command injection | Python, Node.js, Go, Java, Ruby, PHP |
| Unsafe deserialization | Python (pickle/yaml), Ruby (Marshal), PHP (unserialize), Java (ObjectInputStream) |
| Weak crypto | Python (MD5/SHA1 for passwords), Node.js |
| Auth / JWT issues | Node.js (`none` algorithm, hardcoded secrets) |
| CORS misconfiguration | Node.js |
| Debug flags | Python (Django/Flask), Node.js |
| TLS validation bypass | Python, Go |
| Container security | Dockerfiles, Docker Compose |
| IaC misconfigurations | Terraform (AWS S3, SGs, IAM), Kubernetes manifests |
| CI/CD pipeline | GitHub Actions, GitLab CI |
| Dependency CVEs | npm, pip (pip-audit), Cargo (cargo audit) |

### Semantic checks (LLM-powered, not regex)

- Missing authorization / IDOR (can user A access user B's resources?)
- SSRF in HTTP client calls with user-controlled URLs
- Open redirects
- Race conditions in auth flows
- JWT verified client-side only
- Mass assignment via `req.body` spread into ORM
- Sensitive data in log statements
- Missing rate limiting on auth/expensive endpoints
- CSRF on state-changing routes

---

## Self-Improvement Loop

The plugin gets smarter as you use it:

```
Scan → respond to prompts → feedback recorded → /security-learn → proposals → apply
```

1. **Scan** your project
2. For each Critical/High finding: respond `accept`, `dismiss`, `fix`, or `skip`
3. Feedback is logged to `memory/feedback-log.jsonl` (hashed paths/snippets — no raw source stored)
4. After several scans, run `/code-security-guardian:security-learn`
5. Claude clusters your dismissals and proposes narrowing noisy rules or adding suppressions
6. You review and apply what makes sense

Rules proposed by research and learning land in `rules/community/proposed-*.yaml` (disabled by default). You enable them with `codesec-rules enable <id>` after reviewing.

---

## Rule Layers

Rules load in three layers, with later layers overriding earlier ones by `id`:

| Layer | Path | Who writes it | Override? |
|-------|------|---------------|-----------|
| Core | `rules/core/` | Plugin maintainer | Never edit |
| Community | `rules/community/` | Refreshed by `codesec-rules sync` | Enable/disable |
| Local | `rules/local/` | You, or `codesec-learn apply` | Full control |

67 built-in core rules ship with the plugin. Add your own:

```yaml
# rules/local/my-rules.yaml
rules:
  - id: LOCAL_COMPANY_SECRET_FORMAT
    severity: critical
    category: secrets
    cwe: CWE-798
    languages: []
    extensions: []
    pattern: 'MYCO_[A-Z0-9]{32}'
    message: "Internal MYCO API key detected."
    fix_hint: "Store in vault. Remove from source."
    false_positive_hints:
      - "In .gitignored config files"
    enabled: true
    source: local
    added: 2025-04-27
```

```bash
codesec-rules add rules/local/my-rules.yaml
```

---

## Suppressing False Positives

Add an inline ignore comment on the flagged line:

```js
// codesec-ignore: SECRETS_OPENAI_KEY
const STRIPE_PK = 'pk_live_abc123';  // publishable key — safe to expose
```

```python
# codesec-ignore: BACKEND_PYTHON_EVAL
eval(trusted_ast_node)  # pre-validated AST from internal tooling only
```

```html
<!-- codesec-ignore: WEB_INNERHTML_ASSIGN -->
<div id="sanitized"></div>
```

Use the exact rule ID from `bin/codesec-scan --list-rules`.

---

## Configuration

Edit `config/settings.yaml`:

```yaml
learning:
  enabled: true
  min_occurrences_for_proposal: 5     # need 5+ occurrences before proposing
  dismiss_rate_threshold_for_narrowing: 0.7  # 70%+ dismiss rate triggers proposal

privacy:
  hash_file_paths: true       # never store raw paths
  hash_snippets: true         # never store raw code
  share_anonymized_feedback: false  # opt-in only

scanner:
  load_community_rules: true
  load_local_rules: true
  default_excludes:
    - node_modules
    - .git
    - dist
    - build
    - vendor
```

### Enabling the post-edit hook

The plugin includes an optional hook that runs a quick scan on every file Claude edits and surfaces Critical/High findings inline. It is silent on clean files and targets under 2 seconds per file.

To enable, add the following to your Claude Code `settings.json` (find it via **Claude Code → Settings → Edit Config**):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"${CLAUDE_PLUGIN_ROOT}/bin/codesec-scan\" --path \"${TOOL_RESULT_FILE_PATH}\" --quick --json --severity high 2>/dev/null | python3 -c \"\nimport json, sys\ntry:\n    findings = json.load(sys.stdin)\nexcept Exception:\n    sys.exit(0)\nif not findings:\n    sys.exit(0)\ncrits = [f for f in findings if f.get('severity') in ('critical', 'high')]\nif not crits:\n    sys.exit(0)\nprint('\\n⚠️  Security findings in edited file:')\nfor f in crits:\n    sev = f.get('severity','?').upper()\n    rule = f.get('rule_id','?')\n    line = f.get('line','?')\n    msg = f.get('message','?')\n    print(f'  [{sev}] Line {line} ({rule}): {msg}')\nprint('Run /code-security-guardian:security-scan to see full report and fixes.')\n\""
          }
        ]
      }
    ]
  }
}
```

---

## Privacy

All learning is local:

- `memory/feedback-log.jsonl` — file paths are SHA-256 hashed, snippet content is SHA-256 hashed, raw source code is **never** stored
- `memory/project-context.yaml` — git remote URL is SHA-256 hashed
- No telemetry, no analytics, no remote calls unless you explicitly run `codesec-rules sync`
- To review exactly what an export contains before sharing: `bin/codesec-learn export --plain`

---

## Limitations

This is a static analysis tool, not a penetration test:

- **Regex first pass.** The scanner catches known patterns quickly. Novel or obfuscated code may be missed.
- **No execution.** Dynamic vulnerabilities (timing attacks, complex deserialization gadget chains, real SSRF to internal services) require dynamic testing to confirm.
- **Dependency audit requires native tools.** `npm audit`, `pip-audit`, `cargo audit` must be installed. The scanner skips gracefully if they're absent.
- **No threat modeling.** Authentication flow design, trust boundary violations, and business logic attacks require domain knowledge beyond static analysis.

For production systems handling sensitive data: supplement with DAST, manual penetration testing, and threat modeling.

---

## Contributing

1. Fork the repo at https://github.com/Rezasz/code-security-guardian
2. Add rules to `rules/core/` following the schema in `skills/security-research/references/rule-authoring.md`
3. Test against a fixture: `bin/codesec-rules test <rule-id> --on ./fixture`
4. Validate: `bin/codesec-rules validate rules/core/my-new-rules.yaml`
5. Open a PR

Rule guidelines:
- Narrow patterns beat broad ones — a rule with 30% false positive rate is noise
- Every rule needs `false_positive_hints`
- Default new rules to `confidence: medium` until validated at scale
- Document the CWE mapping

---

## Acknowledgements

Built with love for the developer community by [Reza Sahebozamani](https://github.com/Rezasz).

If this saved you from shipping a vulnerability, please star the repo and share it with a friend.

Security is everyone's responsibility — especially when the code writes itself.
