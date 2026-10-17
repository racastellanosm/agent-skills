# SecOps Scanner & Security Audit Agent Skill

[![agentskills.io](https://img.shields.io/badge/spec-agentskills.io-blue.svg)](https://agentskills.io)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](../../LICENSE)

An open-standard security engineering skill for AI coding assistants (Antigravity, Claude Code, Gemini CLI, Cursor, OpenAI Codex, OpenCode) that executes a **100% Dockerized ("Zero Host Dependencies")** application security audit across **SAST, SCA/Dependencies, Docker/IaC, Secret Detection, and OWASP ASVS Code Review**, producing a comprehensive Markdown report (`SECURITY-AUDIT-REPORT.md`).

---

## 🎯 Core Philosophy: 100% Dockerized Security Auditing

Traditional security audits require installing and maintaining multiple language-specific CLIs on the host machine (Python for Semgrep, Go binaries for OSV-Scanner/Gitleaks, Homebrew packages for Trivy).

This skill enforces a strict **Zero Host Dependencies** invariant:
* **The host machine requires ONLY Docker.**
* All 4 industry-standard security engines run inside ephemeral containers (`docker run --rm --pull=always -v "$(pwd):/src:ro"`).
* Every scan automatically verifies and pulls the latest upstream scanner image (`--pull=always`) before execution.
* Every audit automatically produces a structured **Markdown Security Audit Report** containing an executive posture score, a consolidated findings table, and actionable remediation code.

---

## 📦 Universal Installation

Install this skill into your workspace or globally on your system using `skills.sh`:

```bash
# Add to current workspace / project
npx skills add racastellanosm/agent-skills --skill secops-scanner

# Or install globally for all projects on your machine (Recommended)
npx skills add racastellanosm/agent-skills --skill secops-scanner -g
```

---

## 🛡️ The 5 Security Audit Pillars

| Pillar | Engine (Official Docker Image) | Scope & Coverage |
| :--- | :--- | :--- |
| **1. SAST** | `semgrep/semgrep:latest` | Static code analysis across TypeScript, JavaScript, React, Python, Go, PHP, Rust, and Java against OWASP Top 10 & CWE rulesets. |
| **2. SCA / Lockfiles** | `ghcr.io/google/osv-scanner:latest` | Dependency CVE scanning across `bun.lock`, `package-lock.json`, `pnpm-lock.yaml`, `go.mod`, `Cargo.lock`, `requirements.txt`, `composer.lock`. |
| **3. Container & IaC** | `aquasec/trivy:latest` | Audits `Dockerfile`, `docker-compose.yml`, Kubernetes/Terraform configs, and local container image layers (`Alpine`, `Distroless`, `Debian`). |
| **4. Secret Scanning** | `zricethezav/gitleaks:latest` | Detects hardcoded secrets, API tokens, and private keys across the working tree and Git commit history. |
| **5. OWASP Code Review** | Agent Architectural Analysis | Manual verification of authentication (JWT/JWE), password policies (NIST SP 800-63B), RBAC, PII masking, CORS/CSP, and injection boundaries. |

---

## 📊 Automated Markdown Report Output

Every execution generates a structured Markdown report (`.build/security/SECURITY-AUDIT-REPORT.md` or `SECURITY-AUDIT-REPORT.md`) featuring:
1. **Executive Security Posture & Scorecard** (`A+` to `F`).
2. **Scanner Execution Matrix** (status and finding counts per Docker engine).
3. **Consolidated Findings & Remediation Table** (ID, Engine, Severity, Location, CVE/CWE, Description, and Solution).
4. **Step-by-Step Remediation Guide** with drop-in code fixes and package upgrade commands.

---

## 📂 Skill Architecture & Reference Guides

```text
skills/secops-scanner/
├── SKILL.md                               # Open specification skill definition & Docker commands
├── README.md                              # Public documentation and usage guide
└── references/
    ├── audit-report-template.md           # Canonical Markdown template for SECURITY-AUDIT-REPORT.md
    ├── sast-and-sca-containers.md         # Semgrep SAST rulesets & Google OSV-Scanner lockfile guide
    ├── container-and-iac-trivy.md         # Trivy Dockerfile, Compose, and container image scanning guide
    └── secrets-and-code-review.md         # Gitleaks secret detection & OWASP ASVS / NIST review checklist
```
