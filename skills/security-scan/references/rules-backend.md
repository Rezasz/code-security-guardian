# Backend Security Rules

Applies to: Node.js, Python, Go, Rust, Java, Ruby, PHP, C#/.NET, Deno, Bun

---

## Injection

### SQL Injection

- [ ] **String concatenation in SQL queries**
  - Python: `cursor.execute("SELECT * FROM users WHERE id = " + user_id)`
  - Python f-string: `cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")`
  - Python `.format()`: `cursor.execute("SELECT * FROM users WHERE id = {}".format(user_id))`
  - Python `%` interpolation: `cursor.execute("SELECT * FROM users WHERE id = %s" % user_id)` (note: `%s` with a tuple is parameterized and **safe**)
  - Node.js template literal: `` db.query(`SELECT * FROM users WHERE id = ${req.params.id}`) ``
  - Java: `stmt.executeQuery("SELECT * FROM users WHERE id = " + userId)`
  - Ruby: `User.where("id = #{params[:id]}")`
  - PHP: `mysqli_query($conn, "SELECT * FROM users WHERE id = " . $_GET['id'])`
  - CWE-89
  - Severity: **Critical**
  - Fix: Use parameterized queries / prepared statements:
    ```python
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    ```
    ```js
    db.query('SELECT * FROM users WHERE id = $1', [userId])
    ```

- [ ] **ORM raw queries with user input** — `Model.query(raw_sql + user_input)`, `db.raw(userInput)`, Django `extra(where=[user_input])`
  - Severity: **Critical**
  - Fix: Use ORM query builders, not raw SQL with user data.

- [ ] **`$where` in MongoDB** — `Model.find({ $where: userInput })`
  - CWE-943
  - Severity: **High**
  - Fix: Never use `$where`. Use standard query operators.

### Command Injection

- [ ] **Python `os.system(` with user input** — `os.system(cmd + user_input)`
  - CWE-78
  - Severity: **Critical**
  - Fix: Use `subprocess.run([...], shell=False)` with a list of arguments.
  - False positives: `os.system(` with a fully hardcoded string.

- [ ] **Python `subprocess` with `shell=True`** — `subprocess.run(user_input, shell=True)`
  - CWE-78
  - Severity: **Critical**
  - Fix: Use `shell=False` with a list: `subprocess.run(['ls', '-la'], shell=False)`

- [ ] **Node.js `child_process.exec(` with user input or template literals**
  - CWE-78
  - Severity: **Critical**
  - Fix: Use `execFile` or `spawn` with argument arrays instead of `exec` with a shell string.

- [ ] **Node.js `child_process.execSync(` with template literals** — `` execSync(`convert ${filename}`) ``
  - Severity: **Critical**

- [ ] **Ruby backtick execution** — `` `#{user_input}` `` or `system("#{user_input}")`
  - CWE-78
  - Severity: **Critical**

- [ ] **PHP `system(`, `exec(`, `passthru(`, `shell_exec(` with user input**
  - CWE-78
  - Severity: **Critical**

- [ ] **Go `exec.Command` with string concatenation** — `exec.Command("sh", "-c", "cmd "+userInput)`
  - CWE-78
  - Severity: **Critical**
  - Fix: Pass arguments as separate strings: `exec.Command("convert", inputFile, outputFile)`

### Path Traversal

- [ ] **`../` in user-controlled file paths** without normalization
  - CWE-22
  - Severity: **High**
  - Python: `open(base_dir + user_path)` — fix: `os.path.realpath(os.path.join(base, user_path))` and assert it starts with `base_dir`
  - Node.js: `fs.readFile(path.join(baseDir, userPath))` — must verify the resolved path is within `baseDir`
  - Go: `os.Open(filepath.Join(baseDir, userPath))` — check `strings.HasPrefix(resolved, baseDir)`

- [ ] **Missing prefix check after `path.resolve`** — resolving but not verifying the result is within the allowed directory
  - Severity: **High**

### SSRF (Server-Side Request Forgery)

- [ ] **`requests.get(user_url)` / `fetch(userInput)` / `http.Get(userInput)` without allowlist**
  - CWE-918
  - Severity: **High**
  - Fix: Validate the URL against an allowlist of trusted hosts. Block private IP ranges (127.x, 10.x, 192.168.x, 169.254.x for metadata).

- [ ] **URL-fetching with `verify=False` (Python)** — `requests.get(url, verify=False)`
  - Severity: **High** — disables TLS certificate validation (MITM risk)

- [ ] **Go `tls.Config{InsecureSkipVerify: true}`**
  - Severity: **High**

### XML External Entities (XXE)

- [ ] **Python `lxml` / `xml.etree` without disabling external entities**
  - CWE-611
  - Severity: **High**
  - Fix: Use `defusedxml` library for all XML parsing.

- [ ] **Java `DocumentBuilder`, `SAXParser`, `XMLReader` without `setFeature` to disable XXE**
  - Severity: **High**
  - Fix:
    ```java
    factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
    ```

---

## Unsafe Deserialization

- [ ] **Python `pickle.loads(` with user data** — `pickle.loads(user_bytes)`
  - CWE-502
  - Severity: **Critical** — arbitrary code execution
  - Fix: Use JSON or another safe serialization format. Never unpickle untrusted data.

- [ ] **Python `yaml.load(` without SafeLoader** — `yaml.load(user_input)` (without `Loader=yaml.SafeLoader`)
  - CWE-502
  - Severity: **Critical**
  - Fix: `yaml.safe_load(user_input)` or `yaml.load(user_input, Loader=yaml.SafeLoader)`

- [ ] **Ruby `Marshal.load(` with user input**
  - CWE-502
  - Severity: **Critical**

- [ ] **PHP `unserialize($userInput)`**
  - CWE-502
  - Severity: **Critical**
  - Fix: Use `json_decode` instead.

- [ ] **Java `ObjectInputStream`** reading from user-controlled data
  - CWE-502
  - Severity: **Critical**
  - Fix: Use a serialization allowlist or switch to JSON/Protobuf.

- [ ] **Java `XMLDecoder`** from user input
  - CWE-502
  - Severity: **Critical**

---

## Authentication & Session

### Password Hashing

- [ ] **MD5 or SHA1 for passwords** — `hashlib.md5(password)`, `hashlib.sha1(password)`, `MD5.digest(password)`
  - CWE-916
  - Severity: **Critical**
  - Fix: Use `bcrypt`, `argon2`, or `scrypt`. Never use raw hash functions for passwords.

- [ ] **SHA256 for passwords without salt or work factor** — `hashlib.sha256(password)`
  - Severity: **High**
  - Fix: Use `argon2-cffi` or `bcrypt`. SHA-256 is too fast for password hashing.

- [ ] **`Math.random()` for security tokens** — used for session IDs, CSRF tokens, password reset tokens
  - CWE-338
  - Severity: **High**
  - Fix: Use `crypto.randomBytes(32)` (Node.js) or `secrets.token_hex(32)` (Python).

### JWT

- [ ] **Hardcoded JWT secret** — `jwt.sign(payload, 'mysecret')`, `jwt.verify(token, 'hardcoded')`
  - Severity: **Critical**
  - Fix: Use environment variable: `process.env.JWT_SECRET` (min 256 bits of entropy).

- [ ] **JWT `none` algorithm accepted** — `jwt.verify(token, secret, { algorithms: ['none'] })`
  - CWE-347
  - Severity: **Critical** — attacker can forge any token
  - Fix: Always specify allowed algorithms and never include `none`.

- [ ] **Missing `algorithms` option in jwt.verify** — `jwt.verify(token, secret)` without `algorithms` — some libraries default to accepting `none`
  - Severity: **High**
  - Fix: `jwt.verify(token, secret, { algorithms: ['HS256'] })`

- [ ] **No expiry (`exp`) on JWT** — tokens that never expire
  - Severity: **Medium**

### Session Fixation / Hijacking

- [ ] **Session not regenerated after login** — not calling `req.session.regenerate()` (Express) after authentication
  - CWE-384
  - Severity: **Medium**

---

## Authorization

- [ ] **Missing authorization check on route** — handler reads user data or performs privileged action without verifying `req.user` matches the resource owner
  - CWE-285
  - Severity: **High**
  - Example: `GET /api/orders/:orderId` that returns the order without checking `order.userId === req.user.id`

- [ ] **Mass assignment** — `User.create(req.body)`, `user.update_attributes(params)`, spreading request body into ORM create
  - CWE-915
  - Severity: **High**
  - Fix: Use an explicit allowlist of permitted fields.

---

## CSRF

- [ ] **State-changing endpoints (POST/PUT/DELETE/PATCH) without CSRF protection**
  - CWE-352
  - Severity: **High** (lower if API-only with no cookies)
  - Fix: Use `csurf` middleware (Express) or Django's `{% csrf_token %}`. Alternatively, use `SameSite=Strict` or `SameSite=Lax` on session cookies.

- [ ] **CORS configured to allow all origins with credentials** — `Access-Control-Allow-Origin: *` + `Access-Control-Allow-Credentials: true`
  - CWE-942
  - Severity: **High**
  - Fix: Specify exact trusted origins instead of `*`.

- [ ] **`app.use(cors())` with no options (Express)** — defaults to `*`
  - Severity: **Medium**
  - Fix: `app.use(cors({ origin: ['https://example.com'] }))`

---

## Security Headers

- [ ] **Missing `helmet` in Express** — no `app.use(helmet())`
  - Severity: **Medium**
  - Fix: `npm install helmet` and `app.use(helmet())`.

- [ ] **Verbose error messages** — `res.json(err)` or `res.send(err.stack)` — stack traces exposed to clients
  - CWE-209
  - Severity: **Medium**

---

## Rate Limiting

- [ ] **No rate limiting on authentication endpoints** — `/login`, `/register`, `/forgot-password`, `/verify-otp` without `express-rate-limit` or equivalent
  - CWE-307
  - Severity: **High**
  - Fix: Add rate limiting middleware, ideally backed by Redis for distributed deployments.

- [ ] **No rate limiting on expensive operations** — endpoints that trigger email sends, file processing, external API calls
  - Severity: **Medium**

---

## Logging

- [ ] **Passwords or tokens in log statements** — `logger.info('Login attempt', { password })`, `console.log('Auth token:', token)`
  - CWE-532
  - Severity: **High**
  - Fix: Remove sensitive fields from logs. Use a log sanitizer.

---

## Debug Flags

- [ ] **`DEBUG = True` in Django `settings.py`** (non-test file)
  - Severity: **High** — exposes stack traces and SQL queries to end users
  - Fix: Use `DEBUG = os.environ.get('DEBUG', 'False') == 'True'`

- [ ] **`app.run(debug=True)` in Flask** (non-test file)
  - Severity: **High** — enables interactive debugger accessible from browser

- [ ] **Node.js `NODE_ENV !== 'production'` used to skip security checks**
  - Severity: **Medium** — verify NODE_ENV is set correctly in production

---

## Cryptography

- [ ] **ECB mode** — `AES.new(key, AES.MODE_ECB)` or equivalent
  - CWE-327
  - Severity: **High** — ECB mode leaks patterns in plaintext
  - Fix: Use AES-GCM or AES-CBC with a random IV.

- [ ] **Hardcoded IV** — `iv = b'\x00' * 16` or fixed IV value
  - CWE-330
  - Severity: **High**
  - Fix: Generate a random IV per encryption operation and store it alongside the ciphertext.

- [ ] **Weak TLS version** — `ssl.PROTOCOL_TLSv1` or `ssl.PROTOCOL_TLSv1_1` in Python, or equivalent
  - Severity: **High**
  - Fix: Use `ssl.PROTOCOL_TLS_CLIENT` with `minimum_version=ssl.TLSVersion.TLSv1_2`.
