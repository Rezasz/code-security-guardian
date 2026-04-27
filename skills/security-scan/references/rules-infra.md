# Infrastructure Security Rules

Applies to: Dockerfiles, Docker Compose, Terraform, Kubernetes manifests, GitHub Actions, GitLab CI, serverless configs

---

## Docker / Container Security

### Dockerfile

- [ ] **Running as root — `USER root` or no `USER` instruction**
  - CWE-250
  - Severity: **High** — if the container is compromised, the attacker has root inside the container, which can aid breakout
  - Fix: Add a non-root user:
    ```dockerfile
    RUN addgroup --system appgroup && adduser --system --group appuser
    USER appuser
    ```

- [ ] **Using `:latest` tags** — `FROM node:latest`, `FROM python:latest`
  - Severity: **Medium** — non-reproducible builds, you may unintentionally pull a vulnerable image version
  - Fix: Pin to a specific version: `FROM node:20.11.1-alpine3.19`

- [ ] **`ADD` from a URL** — `ADD https://example.com/script.sh /app/`
  - Severity: **Medium** — `ADD` from URL bypasses build cache and downloads at build time without checksum verification
  - Fix: Use `RUN curl -sSL --fail https://... | sha256sum -c - && ...` with a pinned checksum, then `COPY`.

- [ ] **Missing `HEALTHCHECK`**
  - Severity: **Low**
  - Fix: Add `HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1` <!-- codesec-ignore: GENERAL_LOCALHOST_IN_PROD -->

- [ ] **Secrets in `ENV` or `ARG` instructions** — `ENV DATABASE_PASSWORD=secret123`
  - CWE-798
  - Severity: **Critical** — secrets in `ENV` are visible in `docker inspect` and image layers
  - Fix: Use Docker secrets (`--secret`), environment variable injection at runtime, or a secrets manager.

- [ ] **Secrets in `RUN` instructions** — `RUN git clone https://user:token@github.com/...`
  - Severity: **Critical** — stored in image layer history
  - Fix: Use `--mount=type=secret` (BuildKit) to inject secrets at build time without storing them.

- [ ] **`--privileged` flag in `docker run` or Docker Compose** — `privileged: true`
  - Severity: **Critical** — grants all Linux capabilities, effectively gives root on the host
  - Fix: Use specific capabilities with `--cap-add` instead.

- [ ] **Sensitive ports exposed without binding restriction** — `EXPOSE 22`, `EXPOSE 3306` — verify `docker run -p 0.0.0.0:3306:3306` isn't used
  - Severity: **Medium**

### Docker Compose

- [ ] **`privileged: true`** in service definition
  - Severity: **Critical**

- [ ] **Secrets as plain environment variables** — `environment: DB_PASSWORD: secret`
  - Severity: **High**
  - Fix: Use Docker Compose secrets: `secrets:` block with `file:` pointing to a secret file not committed to git.

- [ ] **`network_mode: "host"`** without justification
  - Severity: **High** — bypasses Docker network isolation

---

## Kubernetes Security

### Pod / Container Security Context

- [ ] **`privileged: true`** in container securityContext
  - Severity: **Critical** — effectively root on the node
  - Fix:
    ```yaml
    securityContext:
      privileged: false
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      capabilities:
        drop: ["ALL"]
    ```

- [ ] **`runAsUser: 0`** — running container as root
  - Severity: **High**

- [ ] **`hostNetwork: true`** — container shares the host's network namespace
  - Severity: **High** — bypasses network policies, can reach host services

- [ ] **`hostPID: true`** — container shares host PID namespace
  - Severity: **Critical**

- [ ] **Missing `resources.limits`** — no CPU/memory limits set
  - Severity: **Medium** — enables resource exhaustion (DoS)
  - Fix:
    ```yaml
    resources:
      limits:
        memory: "256Mi"
        cpu: "500m"
      requests:
        memory: "128Mi"
        cpu: "250m"
    ```

- [ ] **`:latest` image tags in k8s manifests** — `image: myapp:latest`
  - Severity: **Medium** — non-deterministic deployments
  - Fix: Use immutable image digests: `image: myapp@sha256:abc123...`

### Secrets Management

- [ ] **Secrets in plain `ConfigMap`** — credentials or API keys in ConfigMap instead of Secret
  - Severity: **High** — ConfigMaps are not access-controlled the same way as Secrets
  - Fix: Use Kubernetes `Secret` objects. Better: use External Secrets Operator with Vault/AWS Secrets Manager.

- [ ] **Secrets stored as base64 in `Secret` objects without encryption at rest** — note that base64 is not encryption
  - Severity: **Medium**
  - Fix: Enable etcd encryption at rest, or use a sealed secrets solution.

### Network Policies

- [ ] **No NetworkPolicy defined** — all pods can communicate with all other pods
  - Severity: **Medium**
  - Fix: Implement a default-deny NetworkPolicy and explicitly allow required traffic.

### RBAC

- [ ] **`ClusterRole` with wildcard verbs or resources** — `verbs: ["*"]` or `resources: ["*"]`
  - Severity: **High**
  - Fix: Follow least privilege — specify only the verbs and resources needed.

- [ ] **`automountServiceAccountToken: true`** (default) on pods that don't need it
  - Severity: **Low**
  - Fix: Set `automountServiceAccountToken: false` on the ServiceAccount or Pod spec.

---

## Terraform / IaC

### AWS

- [ ] **S3 bucket with public ACL** — `acl = "public-read"` or `acl = "public-read-write"`
  - Severity: **Critical** — exposes bucket contents to the internet
  - Fix: Use `acl = "private"` and configure bucket policies explicitly.

- [ ] **S3 `block_public_access` not configured** — missing `aws_s3_bucket_public_access_block` resource
  - Severity: **High**
  - Fix:
    ```hcl
    resource "aws_s3_bucket_public_access_block" "example" {
      bucket                  = aws_s3_bucket.example.id
      block_public_acls       = true
      block_public_policy     = true
      ignore_public_acls      = true
      restrict_public_buckets = true
    }
    ```

- [ ] **Security group open to `0.0.0.0/0` on sensitive ports** — port 22 (SSH), 3389 (RDP), 3306 (MySQL), 5432 (PostgreSQL), 6379 (Redis), 27017 (MongoDB)
  - CWE-732
  - Severity: **Critical**
  - Fix: Restrict to specific IP ranges or use VPN/bastion.

- [ ] **IAM policy with `Action = "*"` and `Resource = "*"`** — wildcard IAM permissions
  - CWE-732
  - Severity: **Critical**
  - Fix: Apply least privilege. Specify exact actions and resource ARNs.

- [ ] **RDS instance with `publicly_accessible = true`**
  - Severity: **Critical**
  - Fix: Set `publicly_accessible = false` and use VPC peering or bastion for access.

- [ ] **Hardcoded secrets in Terraform `variable` defaults or `locals`** — `default = "my-db-password"`
  - Severity: **Critical**
  - Fix: Use `sensitive = true` on the variable and inject value via Terraform Cloud variables or environment variables (`TF_VAR_*`), never in `.tf` files.

- [ ] **Missing encryption for storage resources** — RDS without `storage_encrypted = true`, S3 without server-side encryption, EBS without `encrypted = true`
  - Severity: **Medium**

- [ ] **Unrestricted egress on security groups** — `0.0.0.0/0` on all outbound ports
  - Severity: **Low** — unnecessary but common; restricting egress limits blast radius

### Cloudflare / Other IaC

- [ ] **Hardcoded API tokens in Terraform** — `cloudflare_api_token = "..."`
  - Severity: **Critical**
  - Fix: Use input variables with `sensitive = true` and inject at runtime.

---

## CI/CD Pipeline Security

### GitHub Actions

- [ ] **`pull_request_target` with `actions/checkout` of the PR ref**
  - CWE-829
  - Severity: **Critical** — code from a fork PR runs with repository secrets in scope
  - Fix: Never check out PR code in `pull_request_target` workflows, or if you must, run the code in a sandboxed step without access to secrets.
  - Example dangerous pattern:
    ```yaml
    on: pull_request_target
    jobs:
      build:
        steps:
          - uses: actions/checkout@v4
            with:
              ref: ${{ github.event.pull_request.head.sha }}  # DANGEROUS
    ```

- [ ] **Unquoted expression in `run:` step** — `run: echo ${{ github.event.pull_request.title }}`
  - CWE-78
  - Severity: **High** — attacker-controlled PR title could inject shell commands
  - Fix: Assign to an env variable first:
    ```yaml
    env:
      PR_TITLE: ${{ github.event.pull_request.title }}
    run: echo "$PR_TITLE"
    ```

- [ ] **Secrets logged in CI steps** — `run: echo ${{ secrets.API_KEY }}` or debug steps that print env vars
  - Severity: **Critical** — secrets appear in public CI logs
  - Fix: Remove logging of secrets. GitHub Actions masks `${{ secrets.* }}` but not env vars that hold secrets.

- [ ] **Using `@main` or `@master` for third-party actions** — `uses: some-action@main`
  - Severity: **Medium** — supply chain risk; ref can be moved
  - Fix: Pin to a commit SHA: `uses: some-action@abc123def456`

- [ ] **`GITHUB_TOKEN` with `write-all` permissions** — `permissions: write-all`
  - Severity: **Medium**
  - Fix: Specify minimum required permissions:
    ```yaml
    permissions:
      contents: read
      pull-requests: write
    ```

### GitLab CI

- [ ] **`CI_JOB_TOKEN` passed to untrusted scripts**
  - Severity: **High**

- [ ] **Variables printed in CI logs** — `echo $CI_SECRET_VAR`
  - Severity: **High**

- [ ] **`artifacts: paths` including sensitive files** — uploading `.env`, `*.pem`, or credentials as artifacts
  - Severity: **High**
