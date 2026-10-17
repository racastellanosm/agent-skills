# Container Image & IaC Security Guide (Aqua Trivy)

This reference covers auditing `Dockerfile` definitions, `docker-compose.yml` topologies, Infrastructure-as-Code (IaC) manifests, and built container images using **Aqua Trivy** inside ephemeral Docker containers.

---

## 1. Infrastructure-as-Code (IaC) & Dockerfile Configuration Audit

Scans all `Dockerfile*`, `docker-compose*.yml`, Kubernetes manifests, and Terraform files for security misconfigurations (e.g., running as `root`, exposing sensitive ports, missing healthchecks, excessive Linux capabilities, unpinned tags):

```bash
docker run --rm --pull=always -v "$(pwd):/src:ro" aquasec/trivy:latest \
  config --severity CRITICAL,HIGH,MEDIUM /src
```

### Key Dockerfile Hardening Checklist
When reviewing Trivy config output alongside the project's `Dockerfile`s, verify these architectural invariants:
1. **Multi-Stage Isolation:** Build toolchains, compilers, and devDependencies remain in `builder` stages and are never copied into the final production stage.
2. **Distroless or Minimal Production Base:** Production stages use `distroless`, `scratch`, or minimal hardened `-alpine` images without package managers or interactive shells (`/bin/sh`).
3. **Explicit Non-Root Execution:** Every production stage declares a non-root user (`USER 1000:1000` or `USER nonroot:nonroot`).
4. **Deterministic Image Tags:** Base images avoid floating `:latest` tags and pin explicit versions or SHA-256 digests.
5. **Zero Secrets in Build Layers:** No private keys, `.env` files, or credentials are passed via `ENV` or `ARG` into image layers (`.dockerignore` excludes `.env*`, `.git`, `.build`).

---

## 2. Built Container Image Vulnerability Scanning

To inspect the actual OS packages (`apk`, `dpkg`, `glibc`, `openssl`, `libcrypto`) and bundled application dependencies inside a locally built Docker image, mount the Docker socket into the ephemeral Trivy container:

### List Project Images First
```bash
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.Size}}"
```

### Scan Specific Local Image for OS & Library CVEs
```bash
docker run --rm --pull=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image \
  --scanners vuln \
  --severity CRITICAL,HIGH,MEDIUM \
  "<image-name:tag>"
```

### Scan Base Runtime Images Directly (e.g. Bun, Node, Postgres, Caddy)
You can also audit upstream base images used in `Dockerfile` and `docker-compose.yml`:
```bash
docker run --rm --pull=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image \
  --severity CRITICAL,HIGH \
  oven/bun:1.4-distroless
```

---

## 3. Caching Trivy Vulnerability DB (Faster Repeated Scans)

When scanning multiple images in a single audit session, mount a persistent Docker volume (`trivy-cache`) so Trivy downloads the vulnerability database only once:

```bash
docker run --rm --pull=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v trivy-cache:/root/.cache/ \
  aquasec/trivy:latest image \
  --severity CRITICAL,HIGH \
  "<image-name:tag>"
```
