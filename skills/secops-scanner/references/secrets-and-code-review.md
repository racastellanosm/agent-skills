# Secret Detection (Gitleaks) & OWASP ASVS / NIST Code Review Checklist

Automated scanners catch known patterns and CVEs, but business logic vulnerabilities, authorization bypasses, and cryptographic flaws require structured code inspection by the AI agent.

---

## 1. Automated Secret Detection with Gitleaks

### Scan Full Git Commit History & Working Directory
Detects leaked API keys, JWT secrets, RSA/ECDSA private keys (`-----BEGIN ... PRIVATE KEY-----`), database connection strings, and cloud credentials:
```bash
docker run --rm --pull=always -v "$(pwd):/src:ro" zricethezav/gitleaks:latest \
  detect --source="/src" --no-banner
```

### Scan Uncommitted / Working Tree Changes Only (Pre-Commit Check)
```bash
docker run --rm --pull=always -v "$(pwd):/src:ro" zricethezav/gitleaks:latest \
  dir "/src" --no-banner
```

---

## 2. Manual Architectural & Code Security Audit Checklist (OWASP ASVS v4.0)

During every security audit, the agent MUST inspect the codebase across these 5 critical domains and include findings in `SECURITY-AUDIT-REPORT.md`:

### Domain A: Authentication, Passwords & Session Management
- [ ] **Password Hashing:** Passwords are hashed using memory-hard or adaptive algorithms (`Argon2id`, `bcrypt` with cost $\ge 12$, or `scrypt`). Never MD5/SHA-256.
- [ ] **Password Complexity (NIST SP 800-63B / OWASP):** Enforces minimum length ($\ge 10$ characters), maximum length ($\le 128$ to prevent DoS), character diversity, sequential/repeated character blocks, and context-specific identity checks (blocks email handle / user name tokens).
- [ ] **JWT / Token Verification:**
  - Enforces explicit algorithm allowlists (e.g., `algorithms: ["HS256"]` or `["RS256"]`) to prevent `alg: none` or key-confusion attacks.
  - Rejects weak or default fallback secrets in production (`NODE_ENV === "production"`).
  - Sets strict expiration (`exp`) claims.
- [ ] **Asymmetric / Hybrid Payload Encryption (if present):** Verifies anti-replay timestamp windows and requires persistent key pairs in production.
- [ ] **Brute-Force & Bot Protection:** Login, password reset, and token verification endpoints enforce IP/account rate-limiting and bot challenges (e.g., Cloudflare Turnstile).

### Domain B: Access Control (RBAC / ABAC) & Data Privacy (PII)
- [ ] **Default Deny Routing:** All private API endpoints enforce authentication and explicit role checks via middleware before reaching controllers.
- [ ] **IDOR / Broken Object Level Authorization (BOLA):** Resource lookups scope queries by the authenticated user's tenant/organization/role.
- [ ] **Sensitive Data & PII Masking:**
  - Personal Identifiable Information (national IDs, minors' records, phone numbers, home addresses) is masked or stripped for roles that only require aggregate visibility.
  - Application logs (`console.log`, logger instances) never output plaintext passwords, tokens, or PII.

### Domain C: Input Validation & Injection Prevention
- [ ] **Boundary Schema Validation:** All HTTP inputs (body, query, params) are validated at the controller boundary using strict schemas (Zod, TypeBox, Valibot, Pydantic, etc.).
- [ ] **SQL / NoSQL Injection:** All database queries use parameterized ORM builders or prepared statements. Watch for raw string interpolation inside `sql.raw()` or `.execute()`.
- [ ] **CSV / Formula Injection (CWE-1236):** Exported CSV/TSV/Excel cells prefix or sanitize values starting with `=`, `+`, `-`, `@`, `\t`, or `\r`.
- [ ] **XSS & HTML Rendering:** Frontend components avoid unescaped `dangerouslySetInnerHTML` or `v-html` with untrusted content.

### Domain D: HTTP Security Headers, CORS & Network Boundaries
- [ ] **CORS Policy:** `Access-Control-Allow-Origin` uses an explicit allowlist of trusted origins rather than wildcard `*` on authenticated routes.
- [ ] **Defensive HTTP Headers:**
  - `Content-Security-Policy` (restricts `default-src`, `frame-ancestors 'none'`, `object-src 'none'`).
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Strict-Transport-Security` (HSTS) in TLS environments.
- [ ] **Internal Probe Protection:** Health/readiness probes (`/healthz`, `/readyz`, `/metrics`) do not leak sensitive internal state to public external IPs.
