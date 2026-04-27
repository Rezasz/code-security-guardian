# Mobile Security Rules

Applies to: iOS (Swift, Objective-C), Android (Kotlin, Java), React Native, Flutter/Dart

---

## Hardcoded Secrets in App Bundle

- [ ] **API keys hardcoded in source** — `let apiKey = "sk-abc123"`, `private val API_KEY = "AIzaSy..."`, `const API_KEY = "..."` in mobile source files
  - CWE-798
  - Severity: **Critical** — app bundles are trivially decompiled/disassembled; any secret in source is public
  - Fix: Move secrets to a backend proxy. Never embed secrets that need to be truly secret in mobile apps. Use backend-for-frontend pattern.

- [ ] **Secrets in `Info.plist` or `AndroidManifest.xml`** — `<key>APIKey</key><string>sk-live-...</string>`
  - Severity: **Critical**
  - Fix: Same as above.

- [ ] **Secrets in `.env` files bundled with the app** (React Native) — `.env` values end up in the JS bundle
  - Severity: **High**
  - Fix: Use runtime config fetched from a secure endpoint, not build-time `.env`.

---

## Insecure Storage

### iOS

- [ ] **Sensitive data in `UserDefaults`** — `UserDefaults.standard.set(token, forKey: "authToken")`
  - CWE-922
  - Severity: **High** — UserDefaults is unencrypted and visible in device backups
  - Fix: Store tokens and credentials in the Keychain:
    ```swift
    let query: [String: Any] = [
      kSecClass as String: kSecClassGenericPassword,
      kSecAttrAccount as String: "authToken",
      kSecValueData as String: tokenData
    ]
    SecItemAdd(query as CFDictionary, nil)
    ```

- [ ] **Sensitive data written to files without encryption** — `FileManager.default.createFile(atPath: ..., contents: sensitiveData)`
  - Severity: **Medium**
  - Fix: Use `.completeFileProtection` attribute.

### Android

- [ ] **Sensitive data in `SharedPreferences`** — `prefs.edit().putString("token", authToken).apply()`
  - CWE-922
  - Severity: **High** — SharedPreferences is unencrypted on non-rooted devices but trivially readable on rooted devices and in backups
  - Fix: Use Android Keystore + EncryptedSharedPreferences:
    ```kotlin
    val sharedPreferences = EncryptedSharedPreferences.create(
      context, "secret_shared_prefs",
      MasterKey.Builder(context).setKeyScheme(MasterKey.KeyScheme.AES256_GCM).build(),
      EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
      EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
    ```

- [ ] **SQLite database storing sensitive data without encryption**
  - Severity: **Medium**
  - Fix: Use SQLCipher for encrypted SQLite databases.

---

## Network Security

### iOS

- [ ] **`NSAllowsArbitraryLoads: true` in `Info.plist`**
  - CWE-295
  - Severity: **High** — disables App Transport Security (ATS), allowing cleartext HTTP to any host
  - Fix: Remove `NSAllowsArbitraryLoads`. Specify exceptions only for required domains with `NSExceptionDomains`.

- [ ] **`NSExceptionAllowsInsecureHTTPLoads: true`** for a domain
  - Severity: **Medium** — allows HTTP for that specific domain
  - Fix: Ensure the domain serves HTTPS and remove the exception.

- [ ] **Missing certificate pinning for sensitive endpoints**
  - CWE-295
  - Severity: **Medium**
  - Fix: Implement certificate or public key pinning using `URLSession` with a custom `URLSessionDelegate`:
    ```swift
    func urlSession(_ session: URLSession, didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
      // compare serverTrust certificate against pinned cert
    }
    ```

### Android

- [ ] **`android:usesCleartextTraffic="true"` in `AndroidManifest.xml`**
  - CWE-319
  - Severity: **High**
  - Fix: Remove this attribute. Use HTTPS for all network communication.

- [ ] **Network security config allowing cleartext** — `network_security_config.xml` with `<domain-config cleartextTrafficPermitted="true">`
  - Severity: **Medium**

- [ ] **`TrustManager` that trusts all certificates** — custom `X509TrustManager` with empty `checkServerTrusted`
  - CWE-295
  - Severity: **Critical** — disables all TLS validation (MITM trivial)
  - Fix: Use the default `TrustManager`. If custom validation is needed, implement it correctly.

- [ ] **`hostnameVerifier` that returns `true` for all hosts** — `_.ALLOW_ALL_HOSTNAME_VERIFIER` or anonymous `HostnameVerifier` returning `true`
  - Severity: **Critical**

---

## WebView Security

- [ ] **WebView with `setJavaScriptEnabled(true)` and `loadUrl(userInput)` (Android)**
  - CWE-79 / CWE-749
  - Severity: **High**
  - Fix: Whitelist allowed URLs. Never load arbitrary user-provided URLs in a JavaScript-enabled WebView.

- [ ] **`addJavascriptInterface` on WebView** — exposes Java methods to JavaScript
  - CWE-749
  - Severity: **High** — if the WebView loads untrusted content, XSS can call device APIs
  - Fix: Only use with trusted, local content. Add `@JavascriptInterface` annotation only to methods that are safe to expose.

- [ ] **iOS `WKWebView` loading user-provided URLs with JavaScript enabled** (default for WKWebView)
  - Severity: **Medium**
  - Fix: Validate and sanitize URLs. Consider disabling JS for content that doesn't need it.

---

## Android-Specific

- [ ] **Exported Activities/Services without permission checks** — `android:exported="true"` without `android:permission` in `AndroidManifest.xml`
  - CWE-926
  - Severity: **High** — other apps can launch the activity or call the service without restriction
  - Fix: Add `android:permission="com.example.permission.MY_PERMISSION"` or set `android:exported="false"` if not needed by other apps.

- [ ] **Implicit Intents for sensitive operations** — using implicit intents to start activities that handle sensitive data
  - CWE-927
  - Severity: **Medium**
  - Fix: Use explicit intents with the target component class name.

- [ ] **`android:debuggable="true"` in release manifest**
  - Severity: **High** — allows debugging of the app on any device, ADB access to app data
  - Fix: Remove this attribute from release builds. Let the build system control it.

- [ ] **`android:allowBackup="true"` for sensitive apps** (default is true)
  - Severity: **Low**
  - Fix: Set `android:allowBackup="false"` in `AndroidManifest.xml` for apps handling sensitive data.

### Deep Link / URL Scheme Handling

- [ ] **Deep link parameters used without validation** — `url.queryParameters['redirect']` used directly for navigation or `loadUrl`
  - CWE-601
  - Severity: **High**
  - Fix: Validate deep link parameters against an allowlist before processing.

- [ ] **URL scheme hijacking** — custom URL scheme registered but without uniqueness (another app could register the same scheme)
  - Severity: **Medium**
  - Fix: Use App Links (Android) or Universal Links (iOS) which are domain-verified instead of custom schemes.

---

## iOS-Specific

- [ ] **Biometric authentication bypass** — using `LAContext.evaluatePolicy` but not verifying the result leads to a flow that continues on failure
  - CWE-287
  - Severity: **High**
  - Fix: Always check the `error` parameter. Use the Keychain `kSecAccessControlBiometryCurrentSet` flag to tie the key to biometric state, rather than rolling your own check.

- [ ] **Sensitive data in pasteboard** — `UIPasteboard.general.string = sensitiveValue`
  - Severity: **Low**
  - Fix: Avoid putting passwords or tokens in the system pasteboard.

- [ ] **Sensitive data in screenshots** — app doesn't obscure sensitive fields on `applicationWillResignActive`
  - Severity: **Low**
  - Fix: Add a blur/overlay overlay in `sceneWillResignActive` or `applicationWillResignActive`.

---

## React Native

- [ ] **Secrets in JS bundle** — any `const SECRET = "..."` will be in the bundle
  - Severity: **Critical**
  - Fix: Use a backend-for-frontend. All secrets must live server-side.

- [ ] **`AsyncStorage` for sensitive data** — React Native's AsyncStorage is unencrypted
  - CWE-922
  - Severity: **High**
  - Fix: Use `react-native-keychain` or `react-native-encrypted-storage`.

- [ ] **`WebView` with `javaScriptEnabled` loading user URLs** — same risks as native WebView
  - Severity: **High**

---

## Flutter / Dart

- [ ] **Secrets in Dart source** — `final apiKey = 'sk-...'`
  - Severity: **Critical** — Dart code is compiled but not obfuscated by default

- [ ] **`flutter_secure_storage` not used for sensitive data** — using `SharedPreferences` for tokens
  - Severity: **High**
  - Fix: Use `flutter_secure_storage` package which uses Keychain (iOS) / Keystore (Android).

- [ ] **`http` package without TLS verification** — `HttpClient()..badCertificateCallback = (_, __, ___) => true`
  - Severity: **Critical**
