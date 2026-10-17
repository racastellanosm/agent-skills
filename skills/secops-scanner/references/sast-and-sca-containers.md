# SAST (Semgrep) & SCA (Google OSV-Scanner) Container Guide

This reference details how to execute and customize **Static Application Security Testing (SAST)** with Semgrep and **Software Composition Analysis (SCA)** with Google OSV-Scanner using 100% ephemeral Docker containers.

---

## 1. Static Application Security Testing (SAST) with Semgrep

### Standard Auto-Detection Scan
Automatically detects all programming languages and frameworks in the repository and applies curated security rules:
```bash
docker run --rm --pull=always -v "$(pwd):/src:ro" semgrep/semgrep:latest \
  semgrep scan --config=auto --quiet /src
```

### Targeted Ruleset Scans (High Signal / Low Noise)
When auditing specific stacks or focusing strictly on high-confidence security rules (`p/security-audit`, `p/owasp-top-ten`, `p/secrets`), specify explicit `--config` packs:

* **TypeScript / JavaScript / React / Node / Bun:**
  ```bash
  docker run --rm --pull=always -v "$(pwd):/src:ro" semgrep/semgrep:latest \
    semgrep scan \
    --config p/owasp-top-ten \
    --config p/typescript \
    --config p/react \
    --config p/jwt \
    --exclude="node_modules" --exclude="dist" --exclude=".build" --exclude="coverage" \
    --quiet /src
  ```

* **Python (FastAPI / Django / Flask):**
  ```bash
  docker run --rm --pull=always -v "$(pwd):/src:ro" semgrep/semgrep:latest \
    semgrep scan --config p/owasp-top-ten --config p/python --quiet /src
  ```

* **Go (Golang):**
  ```bash
  docker run --rm --pull=always -v "$(pwd):/src:ro" semgrep/semgrep:latest \
    semgrep scan --config p/owasp-top-ten --config p/golang --quiet /src
  ```

* **PHP (Laravel / Symfony):**
  ```bash
  docker run --rm --pull=always -v "$(pwd):/src:ro" semgrep/semgrep:latest \
    semgrep scan --config p/owasp-top-ten --config p/php --quiet /src
  ```

### Exporting JSON Results to `.build/security/`
To persist machine-readable SAST output for report generation:
```bash
mkdir -p .build/security
docker run --rm --pull=always -v "$(pwd):/src" semgrep/semgrep:latest \
  semgrep scan --config=auto --json --output=/src/.build/security/semgrep-results.json /src
```

---

## 2. Software Composition Analysis (SCA) with Google OSV-Scanner

Google's **OSV-Scanner** queries the open-source [OSV.dev](https://osv.dev) database directly against project lockfiles—making it ideal for modern runtimes like **Bun (`bun.lock`)**, **Node (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`)**, **Go (`go.mod`)**, **Rust (`Cargo.lock`)**, **Python (`requirements.txt`, `poetry.lock`, `uv.lock`)**, and **PHP (`composer.lock`)**.

### Recursive Repository Lockfile Scan
```bash
docker run --rm --pull=always -v "$(pwd):/src:ro" ghcr.io/google/osv-scanner:latest \
  scan source -r /src
```

### Scanning a Specific Lockfile Explicitly
If a repository has multiple nested directories or vendor folders, target specific lockfiles directly:
```bash
# Bun lockfile (Text-based bun.lock)
docker run --rm --pull=always -v "$(pwd):/src:ro" ghcr.io/google/osv-scanner:latest \
  scan source -L /src/bun.lock

# Node / NPM lockfile
docker run --rm --pull=always -v "$(pwd):/src:ro" ghcr.io/google/osv-scanner:latest \
  scan source -L /src/package-lock.json
```

### Fallback / Complementary Dependency Scan with Aqua Trivy FS
If a lockfile format is custom or requires complementary advisory matching (GitHub Security Advisories, NVD, RedHat, Alpine, Debian):
```bash
docker run --rm --pull=always -v "$(pwd):/src:ro" aquasec/trivy:latest \
  fs --scanners vuln --severity CRITICAL,HIGH,MEDIUM /src
```

### Supply-Chain Lifecycle Script Check (Bun / Node)
When auditing a Bun or Node.js repository, always verify that arbitrary `preinstall`/`postinstall` lifecycle scripts are restricted:
1. Check for `ignore_scripts = true` in `bunfig.toml` or `ignore-scripts=true` in `.npmrc`.
2. Run untrusted lifecycle check inside Docker:
   ```bash
   docker run --rm --pull=always -v "$(pwd):/app:ro" -w /app oven/bun:1.4-alpine bun pm untrusted
   ```
