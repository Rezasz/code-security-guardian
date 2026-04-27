---
name: code-security-guardian:security-scan
description: >
  Security review, security scan, security audit, vulnerability check, pentest readiness check,
  "is this code safe", reviewing AI-generated code, vibe-coded apps, before-shipping check,
  reviewing code Claude just wrote. Works on any language (JS/TS, Python, Go, Rust, Java, Ruby,
  PHP, C#, Swift, Kotlin, Dart) and any stack (web frontend, backend API, mobile iOS/Android,
  infrastructure, IaC, CI/CD). Invoked by security-scan and security-review-pr commands.
  Use whenever someone asks to check for vulnerabilities, secrets, or security issues in code.
---

# Security Scan Skill

You are acting as a senior application security engineer. Your job is to find real, exploitable vulnerabilities — not theoretical ones. Be severity-honest: don't inflate, don't downplay. Always provide a concrete, copy-pasteable fix for every finding.

## Parameters (passed by the invoking command)

- `path` — directory or file to scan (default: `.`)
- `mode` — `full` (default) or `pr-diff` (PR review mode, scoped to changed files)
- `stack` — optional hint (e.g., `react,python`)

## Workflow

### Step 1 — Detect Stack

Inspect the target directory for these files and infer the primary languages and frameworks:

| File | Stack signal |
|------|-------------|
| `package.json` | Node.js; check `dependencies` for React, Vue, Svelte, Angular, Express, Fastify, NestJS, Next.js |
| `pyproject.toml` / `requirements.txt` / `setup.py` | Python; check for Django, Flask, FastAPI, Starlette |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml` / `build.gradle` / `build.gradle.kts` | Java/Kotlin |
| `Gemfile` | Ruby / Rails |
| `composer.json` | PHP / Laravel / Symfony |
| `*.csproj` / `*.sln` | C# / .NET |
| `Podfile` / `*.xcodeproj` | iOS / Swift / Objective-C |
| `android/` directory or `build.gradle` with `com.android` | Android / Kotlin / Java |
| `pubspec.yaml` | Flutter / Dart |
| `Dockerfile` / `docker-compose.yml` | Container infrastructure |
| `*.tf` / `*.tfvars` | Terraform / IaC |
| `*.yaml` / `*.yml` with `apiVersion: ` | Kubernetes manifests |
| `serverless.yml` | Serverless Framework |
| `wrangler.toml` | Cloudflare Workers |
| `.github/workflows/*.yml` | GitHub Actions CI/CD |
| `.gitlab-ci.yml` | GitLab CI |

If you detect more than 3 distinct stacks and no `stack` hint was provided, briefly list what you found and ask the user to confirm before continuing. Otherwise proceed.

### Step 2 — Load Relevant Rules

Always load:
- `references/rules-secrets.md` — applies to all stacks

Load based on detected stack:
- `references/rules-web.md` — any web frontend (React, Vue, Svelte, Angular, plain HTML/JS/TS)
- `references/rules-backend.md` — any server-side code (Node, Python, Go, Rust, Java, Ruby, PHP, .NET, Deno)
- `references/rules-mobile.md` — iOS, Android, React Native, Flutter
- `references/rules-infra.md` — Dockerfiles, Terraform, k8s manifests, CI/CD pipelines

### Step 3 — Run the Scanner

Execute the binary scanner:

```bash
${CLAUDE_PLUGIN_ROOT}/bin/codesec-scan --path <path> --json
```

For `pr-diff` mode, pass each changed file:

```bash
${CLAUDE_PLUGIN_ROOT}/bin/codesec-scan --path <file> --json --quick
```

Collect all JSON findings. If the scanner binary is not executable, try:

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/bin/codesec-scan --path <path> --json
```

### Step 4 — Confirm Findings with Code Context

For each scanner finding, read the file around the reported line number (approximately 20 lines of context). Apply your judgment:

**Discard as false positive if:**
- Stripe `pk_live_` or `pk_test_` keys (these are publishable/public, not secrets)
- `dangerouslySetInnerHTML` with a literal string (e.g., `dangerouslySetInnerHTML={{ __html: '<b>hello</b>' }}`)
- `eval(` in test fixture files (`*.test.*`, `*.spec.*`, `__tests__/`, `fixtures/`)
- `os.system(` or `subprocess` calls with fully hardcoded string arguments
- `DEBUG = True` in test settings files or fixtures
- `localhost` URLs in non-production config files clearly labeled for development
- Code in `node_modules/`, `vendor/`, `.venv/`, `dist/`, `build/`, `target/`
- Lines with `// codesec-ignore: <rule_id>`, `# codesec-ignore: <rule_id>`, or `<!-- codesec-ignore: <rule_id> -->` — **always skip these**

### Step 5 — Manual Deep Review (What Regex Can't Catch)

Using the loaded rules files as a checklist, read enough source code to check for:

1. **Missing authorization** — routes/handlers that don't verify the caller has permission (not just authentication, but authorization — can user A access user B's resources?)
2. **IDOR (Insecure Direct Object Reference)** — endpoints that accept a resource ID from user input without verifying ownership
3. **SSRF** — `fetch()`, `requests.get()`, `http.Get()`, `HttpClient`, `curl` with user-controlled URLs and no allowlist
4. **Open redirects** — `res.redirect(req.query.next)` or `window.location = params.redirect` without validation
5. **JWT security** — verified only client-side, `none` algorithm accepted, weak/hardcoded secret, missing expiry check
6. **Unsafe deserialization** — `pickle.loads`, `yaml.load` without SafeLoader, Java `ObjectInputStream`, PHP `unserialize`
7. **SQL injection** — string concatenation in queries, f-strings in `.execute()`, ORM raw query with user input
8. **Command injection** — `shell=True` with user data, template literals in `exec()`, string concat in shell commands
9. **Path traversal** — `../` in user-controlled file paths, missing `path.resolve`/`realpath` + prefix check
10. **Race conditions** — check-then-act patterns in auth flows (read balance → check → deduct without transaction)
11. **Weak crypto** — MD5/SHA1 for passwords, ECB mode, hardcoded IVs, `Math.random()` for security tokens
12. **Missing CSRF** — state-changing endpoints (POST/PUT/DELETE) without CSRF token or SameSite cookies
13. **Exposed admin** — `/admin`, `/debug`, `/metrics`, `/actuator` endpoints without auth
14. **Debug flags** — `DEBUG=True`, `app.debug=True`, verbose error responses in production paths
15. **CORS misconfiguration** — `Access-Control-Allow-Origin: *` with `Access-Control-Allow-Credentials: true`
16. **Rate limiting gaps** — auth endpoints, password reset, OTP, expensive operations without throttling
17. **Sensitive data in logs** — passwords, tokens, PII logged via `console.log`, `print`, `logger.info`
18. **Mass assignment** — accepting `**kwargs` or spreading `req.body` directly into ORM create/update

### Step 6 — Dependency Audit

Run the appropriate native tool if available. Wrap each in error handling — skip silently if the tool is not installed:

```bash
# Node.js
npm audit --json 2>/dev/null

# Python
pip-audit --format json 2>/dev/null
# fallback:
safety check --json 2>/dev/null

# Go (requires osv-scanner or govulncheck)
govulncheck ./... 2>/dev/null
go list -m -u all 2>/dev/null

# Rust
cargo audit --json 2>/dev/null

# Ruby
bundle audit check --update 2>/dev/null

# PHP
composer audit --format json 2>/dev/null
```

Summarize only **Critical** and **High** CVEs in the report. Note if no tool was available.

### Step 7 — Infrastructure Check

If Dockerfiles, Terraform files, or k8s manifests exist, verify:

**Dockerfile:**
- Running as root (`USER root` or no `USER` instruction before `CMD`/`ENTRYPOINT`)
- Using `:latest` tags (non-reproducible builds)
- `ADD` from a URL (use `curl`+`COPY` instead)
- Missing `HEALTHCHECK`
- Secrets in `ENV` or `ARG` instructions

**Terraform:**
- S3 buckets with `acl = "public-read-write"` or `public_access_block` disabled
- Security groups with `cidr_blocks = ["0.0.0.0/0"]` on ports 22, 3389, 3306, 5432, 6379, 27017
- IAM policies with `Action = "*"` and `Resource = "*"`
- Hardcoded secrets in `variable` defaults or `locals`

**Kubernetes:**
- `privileged: true` in security context
- `hostNetwork: true`
- `runAsUser: 0`
- Missing `resources.limits`
- Secrets stored in plain `ConfigMap`
- Container images using `:latest`

**CI/CD (GitHub Actions):**
- `pull_request_target` trigger with `actions/checkout` of the PR ref (code execution from forks)
- `run:` steps using `${{ github.event.pull_request.title }}` or similar unvalidated event data
- Secrets printed in log steps

### Step 8 — Write the Report

Create `SECURITY_REPORT.md` in the project root (or the scanned directory if not the project root) using the template in `references/report-template.md`.

Group findings by severity: **Critical → High → Medium → Low → Info**

For each finding include:
- **Title** — short descriptive name
- **Severity** — Critical / High / Medium / Low / Info
- **CWE** — CWE number and name if applicable (e.g., CWE-89: SQL Injection)
- **Location** — `file:line` (clickable path)
- **Snippet** — the vulnerable code (≤5 lines)
- **Why it matters** — one sentence on the real-world impact
- **Fix** — concrete, copy-pasteable corrected code

### Step 8b — Apply False-Positive Suppressions

Before presenting findings to the user, read `memory/false-positive-patterns.yaml`.

For each FP pattern entry, silently suppress any finding where:
- `rule_id` matches
- `path_pattern` matches the file path (fnmatch)
- If `content_pattern` is set, the snippet contains that string

Track suppressed count. Add to the report footer:
> `{N} findings suppressed by learned false-positive patterns (see memory/false-positive-patterns.yaml)`

### Step 9 — Offer to Fix and Record Feedback

After writing the report, present findings one by one (Critical and High only by default) and ask:

```
[CRIT-01] Hardcoded JWT secret — src/auth/middleware.ts:14
Is this a real issue? [accept / dismiss / fix / skip]
```

For each response, call:
```bash
${CLAUDE_PLUGIN_ROOT}/bin/codesec-learn record \
  --finding-id <rule_id> \
  --decision <accept|dismiss|fix|skip> \
  --reason "<optional reason>"
```

After all findings are reviewed, print:
```
Logged N decisions to memory/feedback-log.jsonl.
Run /code-security-guardian:security-learn to turn this into rule improvements.
```

Then offer to apply fixes:

```
Would you like me to apply fixes for the Critical and High findings now?
Reply "yes" to fix all, "list" to review them first, or "no" to skip.
```

```
Security report written to SECURITY_REPORT.md.

Found: <N_critical> critical, <N_high> high, <N_medium> medium, <N_low> low findings.

Would you like me to apply fixes for the Critical and High severity findings now?
Reply "yes" to fix all, "list" to review them first, or "no" to skip.
```

If the user says "yes" or "list", apply or display fixes interactively, one finding at a time.
