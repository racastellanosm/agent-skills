# Canonical Security Audit Report Template

Every security audit executed via the `secops-scanner` skill MUST generate or update a Markdown report file (by default at `.build/security/SECURITY-AUDIT-REPORT.md`, or `SECURITY-AUDIT-REPORT.md` at the project root if requested by the user) following this exact structure.

---

## Report Structure & Formatting Rules

1. **Language:** Write the report in the language requested by the user (e.g., Spanish or English), while keeping technical identifiers (`CVE-*`, `CWE-*`, file paths, package names, and CLI commands) exact.
2. **Clickable Links:** Format all project file references as clickable links (`[filename.ext](path/to/filename.ext#L10-L20)`).
3. **False-Positive Filtering:** Never dump raw, unverified scanner logs directly into the findings table. Always inspect the flagged source line first. If a scanner alert is a false positive (e.g., test fixture dummy credentials or safe parameterized query), document it briefly under a "Verified False Positives / Accepted Risks" note rather than inflating the vulnerability count.
4. **Severity Classification:**
   - 🔴 **CRITICAL**: Immediate remote code execution (RCE), hardcoded production secrets/private keys, authentication bypass, unauthenticated SQL injection, or critical CVEs (`CVSS >= 9.0`).
   - 🟠 **HIGH**: Privilege escalation (RBAC bypass), PII leakage, insecure cryptographic algorithms, missing authentication on sensitive routes, or high CVEs (`7.0 <= CVSS < 9.0`).
   - 🟡 **MEDIUM**: Weak password policies, permissive CORS, missing rate limiting, container running as `root`, or medium CVEs (`4.0 <= CVSS < 7.0`).
   - 🔵 **LOW / INFO**: Defense-in-depth hardening (e.g., CSP `unsafe-inline`, floating Docker image tags, informational advisories).

---

## Markdown Template (`SECURITY-AUDIT-REPORT.md`)

```markdown
# 🛡️ Security Audit Report — <Project Name>

**Date:** YYYY-MM-DD  
**Audited Path / Branch:** `<project-root>` (`<git-branch>`)  
**Overall Security Posture:** <🟢 STRONG (A/A+) | 🟡 MODERATE (B/C) | 🔴 CRITICAL RISK (D/F)>  

---

## 1. Executive Summary

<1-2 concise paragraphs summarizing the overall security posture, key architectural strengths discovered (e.g., Distroless containers, parameterized ORM queries, strong cryptography, supply-chain script blocking), and the high-level breakdown of findings.>

### Scanner Execution Summary (100% Dockerized)

| Security Pillar | Engine (Docker Image) | Status | Critical | High | Medium | Low / Info |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **SAST (Static Code)** | `semgrep/semgrep:latest` | ✅ / ⚠️ / ❌ | 0 | 0 | 0 | 0 |
| **SCA (Dependencies)** | `ghcr.io/google/osv-scanner:latest` | ✅ / ⚠️ / ❌ | 0 | 0 | 0 | 0 |
| **IaC & Docker Config** | `aquasec/trivy:latest (config)` | ✅ / ⚠️ / ❌ | 0 | 0 | 0 | 0 |
| **Container Images** | `aquasec/trivy:latest (image)` | ✅ / ⚠️ / ❌ / ⏭️ | 0 | 0 | 0 | 0 |
| **Secret Detection** | `zricethezav/gitleaks:latest` | ✅ / ⚠️ / ❌ | 0 | 0 | 0 | 0 |
| **OWASP / Manual Audit** | Architectural & Code Review | ✅ / ⚠️ / ❌ | 0 | 0 | 0 | 0 |

---

## 2. Architectural Strengths & Verified Controls

* **Supply Chain & Dependencies:** <Summary of lockfile integrity, `ignore-scripts`, OSV results>
* **Container & Runtime Hardening:** <Summary of multi-stage builds, non-root `USER`, Distroless/Alpine usage>
* **Authentication & Cryptography:** <Summary of password hashing, JWT/JWE validation, anti-replay, rate limiting>
* **Data Privacy & Injection Defenses:** <Summary of SQL parameterization, XSS/CSV injection protection, RBAC/PII masking>

---

## 3. Consolidated Findings & Remediation Matrix

| ID | Engine / Pillar | Severity | Location (`File:Line` / `Package` / `Image`) | CVE / CWE / Rule | Finding Description | Recommended Solution |
| :--- | :--- | :---: | :--- | :--- | :--- | :--- |
| `SEC-01` | Semgrep / Trivy / OSV / Manual | 🔴 / 🟠 / 🟡 / 🔵 | `[file.ts](path/to/file.ts#L42)` | `CWE-xxx` / `CVE-xxxx-xxxx` | Concise description of the vulnerability and its security impact. | Concrete, actionable fix (e.g., upgrade package to `vX.Y.Z`, add header, parameterize input). |

*(If zero vulnerabilities are found across all engines, state explicitly in this section that 0 actionable vulnerabilities were detected).*

---

## 4. Detailed Remediation Guide

### `SEC-01`: <Finding Title>
* **Severity:** <🔴 Critical / 🟠 High / 🟡 Medium / 🔵 Low>
* **Affected Component:** `[path/to/file](path/to/file#L10-L25)`
* **Risk Analysis:** <Why this matters and how it could be exploited>
* **Proposed Fix:**
  ```<language>
  // Before (Vulnerable)
  ...

  // After (Hardened)
  ...
  ```

---

## 5. Verification Commands

Run the following commands to re-verify findings after applying remediations:

```bash
# Re-run SAST scan
docker run --rm --pull=always -v "$(pwd):/src:ro" semgrep/semgrep:latest semgrep scan --config=auto --quiet /src

# Re-run Dependency CVE scan
docker run --rm --pull=always -v "$(pwd):/src:ro" ghcr.io/google/osv-scanner:latest scan source -r /src

# Re-run Dockerfile & Compose config scan
docker run --rm --pull=always -v "$(pwd):/src:ro" aquasec/trivy:latest config --severity HIGH,CRITICAL,MEDIUM /src
```
```
