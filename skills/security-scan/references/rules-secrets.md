# Secrets Detection Rules

These rules apply to **all file types** across all stacks. Secrets exposed in code or config files can lead to account takeover, data breaches, and infrastructure compromise.

---

## Rule Checklist

### AWS Credentials
- [ ] **AWS Access Key ID** — pattern `AKIA[0-9A-Z]{16}` in any file
  - CWE-798 (Hardcoded Credentials)
  - Severity: **Critical**
  - Fix: Use IAM roles, environment variables, or AWS Secrets Manager. Rotate the key immediately if found.
  - False positives: None — this pattern is highly specific to AWS.

- [ ] **AWS Secret Access Key** — 40-char base64 string appearing near `aws_secret`, `AWS_SECRET`, `SecretAccessKey`
  - Severity: **Critical**
  - Fix: Same as above. Also run `aws iam list-access-keys` to audit which keys are active.

- [ ] **AWS Session Token** — long base64 string near `aws_session_token` or `X-Amz-Security-Token`
  - Severity: **High** (short-lived but still dangerous)

### GCP / Google Cloud
<!-- codesec-ignore: SECRETS_PRIVATE_KEY,SECRETS_GCP_SERVICE_ACCOUNT -->
- [ ] **GCP Service Account JSON** — `"private_key": "-----BEGIN RSA PRIVATE KEY-----"` or `"private_key": "-----BEGIN PRIVATE KEY-----"` in a JSON file
  - Severity: **Critical**
  - Fix: Store service account JSON in Secret Manager. Use Workload Identity for GKE.

- [ ] **GCP API Key** — `AIza[0-9A-Za-z\-_]{35}` pattern
  - Severity: **High**
  - Fix: Restrict the API key by IP and API scope in the GCP console. Rotate it.

### Private Keys / Certificates
- [ ] **Generic private key** — `-----BEGIN (RSA |EC |OPENSSH |DSA |PGP )?PRIVATE KEY-----`
  - Severity: **Critical**
  - Fix: Remove from repository. Add to `.gitignore`. Generate a new key pair.
  - Note: `-----BEGIN CERTIFICATE-----` is usually a public cert — not a secret.

### Stripe
- [ ] **Stripe live secret key** — `sk_live_[0-9a-zA-Z]{24,}`
  - Severity: **Critical**
  - Fix: Rotate via Stripe dashboard. Move to environment variable.
  - **NOT a secret (skip):** `pk_live_` (publishable key — safe to expose in frontend), `pk_test_` (test publishable), `sk_test_` (test secret — medium severity at most)

- [ ] **Stripe test secret key** — `sk_test_[0-9a-zA-Z]{24,}`
  - Severity: **Medium** (test only, but establishes bad practices)

### GitHub
- [ ] **GitHub Personal Access Token (classic)** — `ghp_[A-Za-z0-9]{36}`
  - Severity: **Critical**
- [ ] **GitHub OAuth token** — `gho_[A-Za-z0-9]{36}`
  - Severity: **Critical**
- [ ] **GitHub App token** — `(ghs_|ghu_)[A-Za-z0-9]{36}`
  - Severity: **High**
- [ ] **GitHub fine-grained PAT** — `github_pat_[A-Za-z0-9_]{82}`
  - Severity: **Critical**

### Slack
- [ ] **Slack bot token** — `xoxb-[0-9]{10,}-[0-9]{10,}-[A-Za-z0-9]{24}`
  - Severity: **High**
- [ ] **Slack app token** — `xapp-[0-9]-[A-Za-z0-9]{10,}`
  - Severity: **High**
- [ ] **Slack webhook URL** — `https://hooks.slack.com/services/T[A-Za-z0-9]+/B[A-Za-z0-9]+/[A-Za-z0-9]+`
  - Severity: **Medium** (can post messages to channel, phishing risk)
- [ ] **General Slack tokens** — `xox[baprs]-` prefix
  - Severity: **High**

### Discord
- [ ] **Discord webhook URL** — `https://discord(app)?.com/api/webhooks/[0-9]+/[A-Za-z0-9_-]+`
  - Severity: **Medium**
- [ ] **Discord bot token** — `[MN][A-Za-z0-9]{23}\.[A-Za-z0-9-_]{6}\.[A-Za-z0-9-_]{27}`
  - Severity: **High**

### OpenAI / Anthropic / AI Services
- [ ] **OpenAI API key** — `sk-[A-Za-z0-9]{32,}` (not in test files)
  - Severity: **High** — billing impact
  - False positives: SSH private key headers can start similarly; verify the full pattern.
- [ ] **Anthropic API key** — `sk-ant-[A-Za-z0-9_-]{32,}`
  - Severity: **High**

### Twilio / SendGrid / Communications
- [ ] **Twilio Account SID** — `AC[a-z0-9]{32}`
  - Severity: **Medium** (SID alone is not a secret, but flag if paired with auth token)
- [ ] **Twilio Auth Token** — 32-char hex string near `auth_token` or `TWILIO_AUTH`
  - Severity: **High**
- [ ] **SendGrid API key** — `SG\.[A-Za-z0-9_-]{22}\.[A-Za-z0-9_-]{43}`
  - Severity: **High**

### Database Connection Strings
- [ ] **PostgreSQL URL with password** — `postgres(ql)?://[^:]+:[^@]+@` <!-- codesec-ignore: SECRETS_DB_URL_POSTGRES -->
  - Severity: **Critical**
- [ ] **MySQL URL with password** — `mysql://[^:]+:[^@]+@` or `mysql\+[a-z]+://[^:]+:[^@]+@` <!-- codesec-ignore: SECRETS_DB_URL_MYSQL -->
  - Severity: **Critical**
- [ ] **MongoDB URL with password** — `mongodb(\+srv)?://[^:]+:[^@]+@` <!-- codesec-ignore: SECRETS_DB_URL_MONGODB -->
  - Severity: **Critical**
- [ ] **Redis URL with password** — `redis://:[^@]+@` or `rediss://:[^@]+@` <!-- codesec-ignore: SECRETS_DB_URL_REDIS -->
  - Severity: **High**
- [ ] **Generic DSN** — password appearing in connection string variables (`DATABASE_URL`, `DB_URL`, `CONN_STRING`)
  - Severity: **Critical**

### JWT Secrets
- [ ] **Hardcoded JWT secret** — short string assigned to `jwt_secret`, `JWT_SECRET`, `jwtSecret`, `secret` in auth config
  - Severity: **Critical** — attackers can forge any token
  - Fix: Use a cryptographically random 256-bit secret from a secrets manager.
- [ ] **JWT `none` algorithm** — `algorithms: ['none']` or algorithm not validated on verify
  - See `rules-backend.md` for full JWT checklist.

### OAuth Client Secrets
- [ ] **OAuth client secret** — string assigned to `client_secret`, `OAUTH_CLIENT_SECRET`, `CLIENT_SECRET`
  - Severity: **High**
  - Note: `client_id` alone is not a secret.

### Generic High-Entropy Strings
- [ ] Variables named `*_secret`, `*_password`, `*_passwd`, `*_api_key`, `*_token`, `*_private_key` assigned a literal string value (not an env var reference)
  - Severity: **High** (confirm entropy before reporting — don't flag placeholder strings like `"your-secret-here"` or `"changeme"`)
  - Common false positives to skip: `"secret"`, `"password"`, `"token"`, `"example"`, `"your_*"`, `"<your_*>"`, `"TODO"`, `"CHANGEME"`, empty strings

---

## Suppression

Add `# codesec-ignore: SECRETS_<RULE>` (or `// codesec-ignore:` / `<!-- codesec-ignore: -->`) on the flagged line to suppress a known false positive.

## Common False Positives

| Pattern | Why it's fine |
|---------|--------------|
| `pk_live_` | Stripe publishable key — intentionally public |
| `pk_test_` | Stripe test publishable key — public |
| `-----BEGIN CERTIFICATE-----` | Public certificate, not a private key |
| `example.com`, `test@example.com` | Reserved test domain |
| `password: "{{ vault_db_password }}"` | Ansible vault reference, not a literal |
| `secret: $(SECRET_FROM_ENV)` | Shell variable reference |
| `AWS_SECRET_ACCESS_KEY=` with no value | Export without assignment |
| Any value in `*.example`, `*.sample`, `*.template` files | Template files |
