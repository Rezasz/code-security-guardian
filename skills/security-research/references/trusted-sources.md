# Trusted Security Research Sources

Use these when executing the security-research skill. All are free/public unless marked otherwise.

---

## Official Advisory Databases

| Source | URL | What it covers | Check frequency |
|--------|-----|----------------|-----------------|
| GitHub Security Advisories | https://github.com/advisories | All ecosystems: npm, pip, Maven, Go, Rust, Ruby, Composer, NuGet | Weekly |
| NVD (NIST) | https://nvd.nist.gov/vuln/search | All CVEs, all vendors | Weekly |
| CISA Known Exploited Vulnerabilities | https://www.cisa.gov/known-exploited-vulnerabilities-catalog | Actively exploited CVEs | Weekly |
| Mitre CVE List | https://cve.mitre.org/ | CVE IDs and descriptions | As needed |

---

## Ecosystem-Specific Advisories

| Ecosystem | URL | Notes |
|-----------|-----|-------|
| npm | https://github.com/advisories?query=ecosystem%3Anpm | Node.js packages |
| PyPI | https://github.com/advisories?query=ecosystem%3Apip | Python packages |
| Go | https://pkg.go.dev/vuln/ | Go module vulnerabilities |
| Rust / cargo | https://rustsec.org/advisories/ | RustSec Advisory Database |
| Ruby gems | https://rubysec.com/advisories/ | RubySec |
| Maven / Java | https://github.com/advisories?query=ecosystem%3Amaven | Java packages |
| NuGet / .NET | https://github.com/advisories?query=ecosystem%3Anuget | .NET packages |
| Packagist / PHP | https://github.com/advisories?query=ecosystem%3Acomposer | PHP packages |
| Swift / CocoaPods | https://github.com/advisories?query=ecosystem%3Aswift | iOS packages |

---

## Security Research Blogs

These publish original vulnerability research, novel attack techniques, and framework-specific deep dives. Check for writeups, not just CVEs.

| Blog / Team | URL | Known for |
|-------------|-----|-----------|
| PortSwigger Web Security Blog | https://portswigger.net/research | Web app vulns, new attack classes |
| Google Project Zero | https://googleprojectzero.blogspot.com | Zero-days, browser/OS vulns |
| Trail of Bits Blog | https://blog.trailofbits.com | Smart contracts, crypto, systems |
| NCC Group Research | https://research.nccgroup.com | Wide coverage: web, mobile, cloud |
| Doyensec Blog | https://blog.doyensec.com | Web app, Electron, node security |
| Assetnote Research | https://www.assetnote.io/resources/research | Web app attack surface |
| SnykSec Blog | https://snyk.io/blog/security/ | Dependency vulns, supply chain |
| Checkmarx Blog | https://checkmarx.com/blog/ | SAST, supply chain attacks |
| OWASP | https://owasp.org/www-project-top-ten/ | Top 10, cheat sheets |
| HackerOne Hacktivity | https://hackerone.com/hacktivity | Real-world disclosed bugs |
| Intigriti Blog | https://blog.intigriti.com | XSS, web research |

---

## Framework / Library Security Pages

| Technology | Security advisory URL |
|------------|----------------------|
| Next.js | https://github.com/vercel/next.js/security/advisories |
| React | https://github.com/facebook/react/security/advisories |
| Express.js | https://expressjs.com/en/advanced/security-updates.html |
| Django | https://docs.djangoproject.com/en/stable/releases/security/ |
| Flask | https://flask.palletsprojects.com/en/stable/security/ |
| Rails | https://rubyonrails.org/security |
| Laravel | https://github.com/laravel/framework/security/advisories |
| Spring Boot | https://spring.io/security |
| Kubernetes | https://kubernetes.io/docs/reference/issues-security/official-cve-feed/ |

---

## Skip These (Require Auth / Paid)

- Tenable / Nessus feed — requires subscription
- Qualys VMDR — requires subscription
- Rapid7 Nexpose — requires subscription
- Any source requiring API key or login

---

## Research Quality Notes

- **Prefer primary sources**: vendor advisories and original disclosure posts over news articles and secondary reporting.
- **Check publication date**: only include items newer than the `since` cutoff.
- **Be skeptical of "0-day" claims** on Twitter/X without a linked PoC or CVE — mark confidence as `low`.
- **Check CVSS score** when available: prioritize Critical (9.0+) and High (7.0–8.9) for rule extraction.
- **Look for PoC code**: if a PoC exists, the pattern is more concrete and a tighter regex rule is possible.
