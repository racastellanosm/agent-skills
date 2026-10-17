---
name: secops-scanner
description: Enforces a 100% Dockerized (Zero-Host-Dependencies) application security audit across SAST (Semgrep), SCA/Dependency CVEs (Google OSV-Scanner), Container & IaC Security (Aqua Trivy), Secret Detection (Gitleaks), and OWASP ASVS / NIST code review. Triggers whenever the user requests a security audit, SAST scan, dependency vulnerability check, Docker image/config scan, or pre-release security assessment, always producing a structured Markdown report with findings and actionable remediations.
version: 1.0.0
author: Raul Castellanos
license: MIT
compatibility:
  - gemini-cli
  - antigravity
  - claude-code
  - codex
  - cursor
  - opencode
categories:
  - security
  - secops
  - devops
  - quality-assurance
allowed-tools:
  - view_file
  - grep_search
  - find_by_name
  - list_dir
  - run_command
  - write_to_file
  - replace_file_content
---

# SecOps Scanner & Security Audit Skill (100% Dockerized)

This skill guides AI agents and engineers to execute comprehensive, multi-layered security audits across any software repository following the **Zero Host Dependencies** philosophy:

> 🛡️ **Zero Host Dependencies Invariant:**  
> The host machine requires **only Docker**.  
> All security scanners (SAST, SCA, IaC, Container Image, and Secret Detection) MUST execute inside isolated, ephemeral containers mounting the project in read-only mode (`:ro`). Zero local language runtimes, Python/Node CLIs, or host binaries are required.

---

## 🧭 Core Audit Pipeline (5 Security Pillars)

```
[ 1. SAST ] ➔ [ 2. SCA / OSV ] ➔ [ 3. IaC & Images ] ➔ [ 4. Secrets ] ➔ [ 5. Code Review & Report ]
 (Semgrep)     (OSV-Scanner)        (Aqua Trivy)         (Gitleaks)      (SECURITY-AUDIT-REPORT.md)
```

When the user requests a security audit (or specific scan layer), execute the corresponding ephemeral Docker containers from the project root (prefixing with `rtk` if available on the host):

### 1. SAST — Static Application Security Testing (Semgrep)
Auto-detects project languages (TypeScript, JavaScript, React, Python, Go, PHP, Rust, Java) and scans source code against OWASP Top 10 and security rulesets:
```bash
docker run --rm --pull=always -v "$(pwd):/src:ro" semgrep/semgrep:latest semgrep scan --config=auto --quiet /src
```

### 2. SCA — Dependency & Lockfile Vulnerability Scanning (Google OSV-Scanner)
Scans all lockfiles (`bun.lock`, `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `go.mod`, `Cargo.lock`, `requirements.txt`, `poetry.lock`, `composer.lock`) against the Google Open Source Vulnerabilities (OSV.dev) database:
```bash
docker run --rm --pull=always -v "$(pwd):/src:ro" ghcr.io/google/osv-scanner:latest scan source -r /src
```

### 3. IaC & Container Security (Aqua Trivy)
* **3A. Dockerfile, Compose & IaC Misconfigurations:**
  ```bash
  docker run --rm --pull=always -v "$(pwd):/src:ro" aquasec/trivy:latest config --severity HIGH,CRITICAL,MEDIUM /src
  ```
* **3B. Built Container Image CVE Scan (when inspecting local images):**
  ```bash
  docker run --rm --pull=always -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --severity HIGH,CRITICAL "<image-name:tag>"
  ```

### 4. Secret & Credential Leak Detection (Gitleaks)
Scans git history and working tree for hardcoded API keys, private keys (`.pem`), tokens, and credentials:
```bash
docker run --rm --pull=always -v "$(pwd):/src:ro" zricethezav/gitleaks:latest detect --source="/src" --no-banner
```

---

## 📄 Mandatory Output: Markdown Security Audit Report

> [!IMPORTANT]
> **MANDATORY DELIVERABLE:**  
> Every audit execution MUST generate or update a structured Markdown report file (default path: `.build/security/SECURITY-AUDIT-REPORT.md`, or `SECURITY-AUDIT-REPORT.md` if requested by the user) following **[Audit Report Template](references/audit-report-template.md)**.

Before writing the final report, the agent MUST:
1. **Triage & Filter False Positives:** Inspect the actual source files flagged by scanners to confirm exploitability and context.
2. **Perform Manual Architectural & Code Review:** Cross-check authentication, cryptography, RBAC/authorization, PII protection, and injection boundaries against **[Secrets & OWASP Code Review Checklist](references/secrets-and-code-review.md)**.
3. **Populate Findings & Remediations Table:** Present every confirmed finding in a consolidated Markdown table with severity, location, CVE/CWE identifier, risk explanation, and exact remediation instructions.

---

## 📚 Deep-Dive References (On-Demand Retrieval)

Load these specific reference guides when executing scans or writing the audit report:

* **[Audit Report Template](references/audit-report-template.md)**: Canonical Markdown report structure, severity taxonomy, executive summary scorecard, and findings/remediation table format.
* **[SAST & SCA Container Guide](references/sast-and-sca-containers.md)**: Advanced Semgrep rulesets (`p/owasp-top-ten`, `p/jwt`, `p/typescript`, `p/react`), custom ignores, and OSV-Scanner lockfile flags.
* **[Container & IaC Trivy Guide](references/container-and-iac-trivy.md)**: Auditing multi-stage Dockerfiles, Distroless/Alpine images, `docker-compose.yml` hardening, and SBOM generation.
* **[Secrets & OWASP Code Review Checklist](references/secrets-and-code-review.md)**: Gitleaks configuration and manual architecture audit checklist (OWASP ASVS v4.0, NIST SP 800-63B, JWT/JWE, CORS/CSP, SQLi/XSS/CWE-1236).
