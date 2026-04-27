# Web Frontend Security Rules

Applies to: `.js`, `.jsx`, `.ts`, `.tsx`, `.vue`, `.svelte`, `.html`, `.ejs`, `.hbs`, `.pug`

---

## XSS (Cross-Site Scripting)

### DOM XSS Sinks

- [ ] **`innerHTML` assignment** — `element.innerHTML = userInput`
  - CWE-79
  - Severity: **High**
  - Fix: Use `textContent` for plain text, or sanitize with DOMPurify before using `innerHTML`.
  - False positives: `innerHTML` assigned a **literal string** with no user data.

- [ ] **`dangerouslySetInnerHTML` (React)** — `dangerouslySetInnerHTML={{ __html: expr }}`
  - CWE-79
  - Severity: **High** if `expr` is dynamic; **Info** if literal
  - Fix: Sanitize with `DOMPurify.sanitize(expr)` before passing. Consider React's built-in text rendering instead.

- [ ] **`v-html` (Vue)** — `<div v-html="userContent">`
  - CWE-79
  - Severity: **High**
  - Fix: Sanitize with DOMPurify. Use `{{ userContent }}` (auto-escaped) for text-only content.

- [ ] **`{@html}` (Svelte)** — `{@html content}`
  - CWE-79
  - Severity: **High**
  - Fix: Sanitize before rendering: `{@html DOMPurify.sanitize(content)}`

- [ ] **`bypassSecurityTrust*` (Angular)** — `bypassSecurityTrustHtml`, `bypassSecurityTrustScript`, `bypassSecurityTrustUrl`, `bypassSecurityTrustResourceUrl`
  - CWE-79
  - Severity: **High**
  - Fix: Avoid these methods. If necessary, sanitize the content first and document why it is safe.

- [ ] **`document.write()`** — `document.write(userInput)`
  - CWE-79
  - Severity: **High**
  - Fix: Use DOM manipulation APIs instead.

- [ ] **`outerHTML` assignment** — `element.outerHTML = userInput`
  - CWE-79
  - Severity: **High**

### JavaScript Evaluation Sinks

- [ ] **`eval()`** — `eval(userInput)` or `eval(req.query.x)`
  - CWE-95
  - Severity: **Critical**
  - Fix: Never eval user input. Use JSON.parse for data, or a safe expression evaluator library.
  - False positives: `eval(` in test/fixture files.

- [ ] **`new Function()`** — `new Function(userInput)()`
  - CWE-95
  - Severity: **Critical**
  - Fix: Same as `eval`.

- [ ] **`setTimeout(string, ...)`** / **`setInterval(string, ...)`** — first argument is a string (code), not a function
  - CWE-95
  - Severity: **High**
  - Fix: Always pass a function reference: `setTimeout(() => doThing(), 1000)`

- [ ] **`vm.runInNewContext` / `vm.runInThisContext`** — with user data
  - CWE-95
  - Severity: **Critical**

### URL Injection

- [ ] **`href={userInput}`** — `<a href={userInput}>` or `<a href={variable}>` where variable comes from props/state/URL params
  - CWE-601 / CWE-79
  - Severity: **High** — `javascript:` URLs execute code
  - Fix: Validate URL starts with `https://` or `/`. Use a safeguard:
    ```js
    const safeHref = /^https?:\/\//.test(url) ? url : '#';
    ```

- [ ] **`window.location = userInput`** — open redirect or XSS
  - CWE-601
  - Severity: **High**
  - Fix: Validate against an allowlist of trusted origins.

### `target="_blank"` without `rel="noopener"`

- [ ] `target="_blank"` without `rel="noopener noreferrer"`
  - CWE-1022
  - Severity: **Medium** — the opened page can access `window.opener` and redirect the parent
  - Fix: Always add `rel="noopener noreferrer"` to external links.

---

## Insecure Storage

- [ ] **`localStorage.setItem` with sensitive keys** — `localStorage.setItem('token', ...)`, `localStorage.setItem('jwt', ...)`, `localStorage.setItem('auth', ...)`
  - CWE-922
  - Severity: **Medium**
  - Why: localStorage is accessible to any JavaScript on the page (XSS risk).
  - Fix: Store auth tokens in `HttpOnly` cookies. If SPA, use in-memory state for the access token and a refresh token in an `HttpOnly` cookie.

- [ ] **`sessionStorage.setItem` with JWT/token** — similar to above
  - Severity: **Low** (cleared on tab close, but still XSS-accessible)

---

## Client-Side Access Control

- [ ] **Role/permission checks in JavaScript only** — `if (user.role === 'admin')` on the client with no server-side enforcement
  - CWE-602
  - Severity: **High**
  - Fix: Always enforce authorization on the server. Client-side checks are UX only.

- [ ] **Sensitive routes behind only a client-side auth guard** — check if protected routes verify the session server-side on load
  - Severity: **High** if API data is fetched without auth headers

---

## Content Security Policy

- [ ] **Missing CSP** — no `Content-Security-Policy` header in server config or `<meta>` tag
  - Severity: **Medium**
  - Fix: Add a restrictive CSP. Minimum: `default-src 'self'; script-src 'self'; object-src 'none';`

- [ ] **`unsafe-inline` in CSP** — `script-src 'unsafe-inline'`
  - Severity: **Medium** — negates XSS protection
  - Fix: Use nonces or hashes instead of `unsafe-inline`.

- [ ] **`unsafe-eval` in CSP** — `script-src 'unsafe-eval'`
  - Severity: **Low**

---

## postMessage Security

- [ ] **`window.addEventListener('message', handler)` without origin check** — handler processes `event.data` without verifying `event.origin`
  - CWE-346
  - Severity: **High**
  - Fix:
    ```js
    window.addEventListener('message', (event) => {
      if (event.origin !== 'https://trusted.example.com') return;
      // process event.data
    });
    ```

- [ ] **`postMessage(data, '*')`** — sending to any origin
  - Severity: **Medium** — sensitive data may leak to malicious iframes
  - Fix: Always specify the target origin: `postMessage(data, 'https://example.com')`

---

## Prototype Pollution

- [ ] **`Object.assign({}, userInput)`** or `_.merge({}, userInput)` or `deepMerge({}, userInput)` with unsanitized user data
  - CWE-1321
  - Severity: **High**
  - Fix: Use `Object.create(null)` as the target, or validate input keys don't include `__proto__`, `constructor`, `prototype`.

---

## Regex DoS (ReDoS)

- [ ] User-controlled input passed to a complex regex with nested quantifiers — e.g., `/(a+)+$/` or `/([a-zA-Z]+)*$/`
  - CWE-1333
  - Severity: **Medium**
  - Fix: Use simple regexes on user input. Consider a timeout wrapper or limit input length.

---

## Subresource Integrity

- [ ] **`<script src="https://...">` or `<link href="https://...">` without `integrity` attribute**
  - CWE-829
  - Severity: **Low**
  - Fix: Add `integrity="sha384-..."` and `crossorigin="anonymous"` to external script/link tags.
