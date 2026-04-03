---
name: security-baseline
description: >
  Perform a frontend security audit of any web application via Playwright
  MCP. Checks OWASP Top 10 items that are testable from the browser: XSS
  vectors, sensitive data exposure, insecure cookies, open redirects,
  CSRF tokens, information leakage in HTML/JS/console. No server access
  needed. Use when user says "security test", "security audit", "OWASP
  check", "is this secure", "penetration test the UI", "check for XSS",
  "security review", "vulnerability scan", "check for data leaks", or
  "frontend security". Also trigger on "can this be hacked" or "security
  baseline".
---

# Frontend Security Baseline (OWASP-Informed)

You perform a security-focused audit of a web application from the browser
using Playwright MCP. You test what an attacker with browser access could
exploit — no server or code access needed.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

**Disclaimer**: This is a baseline frontend check, not a full penetration
test. Recommend professional security assessment for production apps.

---

## Setup

1. **URL** (default: `http://localhost:3000`)
2. **Scope**: entire app or specific area?
3. **Auth**: test account available? (needed for auth-related checks)
4. **Environment**: dev / staging / production? (affects severity of findings)

---

## Checks

### SEC-1: Cross-Site Scripting (XSS) Vectors

For every text input and URL parameter:

**Reflected XSS** — input appears in the page:
- Enter `<img src=x onerror=alert(1)>` in text fields
- Enter `"><svg onload=alert(1)>` in text fields
- Add `?q=<script>alert(1)</script>` to URL params
- Check: is the input rendered as HTML or safely escaped?

**Stored XSS** — input is saved and displayed later:
- Submit `<b>bold test</b>` via forms that save data
- Check: does it render as bold (vulnerable) or show raw tags (safe)?
- Check user-generated content areas: comments, profiles, names

**DOM XSS** — client-side code handles input unsafely:
- Check URL fragments: `#<img src=x onerror=alert(1)>`
- Check if `location.hash` or `document.referrer` is inserted into DOM

**Verdict per field**: SAFE (escaped) / VULNERABLE (executes) / PARTIAL (some filtered)

### SEC-2: Sensitive Data Exposure

**In HTML source**:
- View page source — search for API keys, tokens, passwords, emails
- Check hidden form fields for sensitive values
- Check HTML comments for developer notes, internal URLs, credentials

**In JavaScript**:
- Check inline `<script>` blocks for hardcoded secrets
- Look for `.env` variables exposed in client bundles
- Check for source maps (`.map` files) — they expose original source

**In network / console**:
- Check console output for logged tokens, user data, debug info
- Look for sensitive data in URL query parameters (tokens, emails)
- Check if API responses include more data than the UI displays

**In storage**:
- Check localStorage / sessionStorage for auth tokens, PII
- Check cookies for sensitive data stored without encryption

### SEC-3: Authentication & Session

**Cookie security** (read via accessibility tree or page.evaluate):
- Auth cookies have `HttpOnly` flag? (prevents JS access)
- Auth cookies have `Secure` flag? (HTTPS only)
- Auth cookies have `SameSite` attribute? (CSRF protection)
- Session cookie has reasonable expiration?

**Session handling**:
- After logout: can you hit Back button and see authenticated content?
- After logout: is the session token invalidated? (try reusing it)
- Login form: is autocomplete disabled on password field?
- Login page: does it transmit over HTTPS?

**Password handling**:
- Password field uses `type="password"`?
- No password visible in URL, logs, or error messages?
- Password requirements communicated before submission?

### SEC-4: CSRF Protection

For every form that modifies data (POST/PUT/DELETE):
- Check for CSRF token in hidden field or custom header
- Check if the form works without a referer header
- Check if the token changes per session/request

### SEC-5: Insecure Direct Object References

- Change ID parameters in URLs: `/user/123` → `/user/124`
- Can you access another user's data by changing the ID?
- API endpoints: `/api/users/123` → `/api/users/124`
- Check if authorization is enforced on the server side

### SEC-6: Security Headers (check via page response)

Using Playwright MCP or page.evaluate to inspect response headers:
- `X-Content-Type-Options: nosniff` present?
- `X-Frame-Options: DENY` or `SAMEORIGIN` present?
- `Content-Security-Policy` header present?
- `Strict-Transport-Security` (HSTS) on HTTPS sites?
- `X-XSS-Protection` present? (legacy but signals awareness)

### SEC-7: Information Leakage

- Custom error pages or default server error pages? (stack traces, versions)
- Server version exposed in headers? (`X-Powered-By`, `Server`)
- Directory listing enabled? (try `/static/`, `/uploads/`, `/assets/`)
- Robots.txt reveals sensitive paths?
- `.env`, `config.json`, `.git/` accessible from browser?
- API returns excessive data? (password hashes, internal IDs, emails)

### SEC-8: Open Redirects

- Check URL parameters that control redirects: `?redirect=`, `?next=`, `?url=`
- Try: `?redirect=https://evil.com` — does it redirect externally?
- Check post-login redirect parameter for manipulation

### SEC-9: Client-Side Validation Only

- Submit forms with browser DevTools bypassing validation
- Check: does the server also validate? (or only the frontend)
- Via Playwright: submit empty required fields directly via API
- File uploads: does the server check file type, or only the frontend?

---

## Severity for Security Issues

| Finding | Severity |
|---------|----------|
| Working XSS (stored) | S0 Blocker |
| Working XSS (reflected) | S1 Critical |
| Sensitive data in HTML/JS | S1 Critical |
| Missing auth cookie flags | S2 Major |
| No CSRF protection on state-changing forms | S2 Major |
| Missing security headers | S2 Major |
| Information leakage (stack traces, versions) | S2 Major |
| Open redirect | S2 Major |
| Client-only validation | S3 Minor |
| Missing Content-Security-Policy | S3 Minor |

---

## Report

Save to `qa/reports/{project}/security/run-[date].md`:

```
## Frontend Security Baseline — [URL]
### Date: [date] | Environment: [dev/staging/prod]

### Summary
| Category                  | Status | Findings |
|---------------------------|--------|----------|
| XSS vectors               | ⚠ WARN | 2        |
| Sensitive data exposure    | ✗ FAIL | 3        |
| Auth & session             | ✓ PASS | 0        |
| CSRF protection            | ✓ PASS | 0        |
| Object reference           | ⚠ WARN | 1        |
| Security headers           | ✗ FAIL | 4        |
| Information leakage        | ⚠ WARN | 2        |
| Open redirects             | ✓ PASS | 0        |
| Client-only validation     | ⚠ WARN | 3        |

### Risk Level: [Low / Medium / High / Critical]

### Findings (sorted by severity)

#### SEC-VULN-001: Stored XSS in comment field [S0]
- **Category**: XSS
- **Location**: /posts/:id — comment input
- **Payload**: `<img src=x onerror=alert(1)>`
- **Result**: Script executes when any user views the comment
- **Impact**: Attacker can steal session cookies of all visitors
- **Fix**: Sanitize HTML output, use Content-Security-Policy
- **Evidence**: qa/screenshots/SEC-001.png

(continue for each finding...)

### Checklist Summary
| Check | Result |
|-------|--------|
| XSS: all inputs escaped | ⚠ 2 fields vulnerable |
| No secrets in HTML source | ✗ API key found |
| Auth cookies: HttpOnly + Secure + SameSite | ✓ |
| CSRF tokens on all forms | ✓ |
| Security headers present | ✗ Missing 4 headers |
| No stack traces in errors | ✓ |
| No open redirects | ✓ |

### Recommendations (priority order)
1. [Fix XSS — immediate]
2. [Remove API key from source — immediate]
3. [Add security headers — this sprint]
```

### After the Report

- "Want me to **create Jira tickets** for the security findings?" → `jira-create-bug`
  (all findings mapped as bugs with severity from the security severity table)

---

## Rules
- **Never exploit vulnerabilities** — only prove they exist
- **Don't test production** unless explicitly authorized
- **Stop at proof-of-concept** — `alert(1)` is enough, don't escalate
- Report findings even if you're not sure of exploitability
- Every finding needs a concrete fix recommendation
- This is NOT a replacement for professional penetration testing
- **Always close the browser** (`browser_close`) after the session is complete
