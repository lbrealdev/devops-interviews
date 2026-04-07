# DevOps Interview Questions — Set 3

Technical interview questions for a DevOps Engineer position.

---

## Security

### 1. Supply Chain Attacks

**Question:** What is a software supply chain attack? How would you defend against one?

**Answer:**

A supply chain attack targets dependencies, build systems, or distribution channels rather than the application itself. Attackers compromise trusted components (libraries, CI pipelines, container images) to inject malicious code that propagates downstream.

**Real-world examples:** SolarWinds (2020), CodeCov (2021), XZ Utils (2024).

**Defense:**
- **Pin dependency versions** and use lock files. Audit with Dependabot, Snyk, or Trivy.
- **Sign artifacts** with Sigstore/Cosign. Verify base images before use.
- **Harden CI/CD**: least-privilege tokens, ephemeral runners, GitHub OIDC instead of long-lived keys.
- **Generate SBOMs** (Syft, CycloneDX) for every artifact.
- **Scan container images** for CVEs before deployment.
- **Enforce policies** with OPA/Kyverno to block unsafe deployments.

**SLSA framework** grades supply chain integrity from levels 1–4. Aim for SLSA 3+ for production.

**Interview signal:** Mentions specific attack vectors, references SLSA or Sigstore, and advocates defense-in-depth over a single tool.

---

## SDLC

### 2. SDLC & DevOps Integration

**Question:** Walk me through the SDLC and where DevOps adds the most value.

**Answer:**

DevOps integrates into every SDLC stage to accelerate delivery while maintaining quality:

| Stage | DevOps Value |
|-------|-------------|
| **Planning** | Align engineering with business priorities, track dependencies |
| **Design** | Threat modeling, Architecture Decision Records (ADRs) |
| **Development** | Pre-commit hooks, linting, DevContainers for consistent environments |
| **Testing** | Automated quality gates in CI, shift-left security (SAST/DAST) |
| **Build** | Reproducible builds, artifact registries, SBOM generation |
| **Release** | GitOps (ArgoCD), environment promotion with approval gates |
| **Deploy** | Blue-green, canary, feature flags for zero-downtime releases |
| **Operate** | Monitoring, alerting, runbooks, auto-scaling to maintain SLOs |
| **Feedback** | Observability, error budgets, post-mortems closing the loop to planning |

**Key principles:** shift-left, automate everything, short feedback loops, blameless post-mortems.

**Interview signal:** Explains how DevOps collapses dev/ops silos into a continuous loop. References DORA metrics (deployment frequency, lead time, MTTR, change failure rate).

---

## Advanced CI/CD

### 3. Deployment Strategies

**Question:** Compare blue-green, canary, and rolling deployments. How do you implement automated rollback?

**Answer:**

| Strategy | How it works | Pros | Cons | Best for |
|----------|-------------|------|------|----------|
| **Rolling** | Replaces old instances incrementally | No extra cost, zero downtime, simple | Old + new run simultaneously | Stateless services, backward-compatible changes |
| **Blue-Green** | Two identical envs, switch traffic all at once | Instant cutover and rollback | 2x infrastructure cost | Major version changes, compliance sign-off |
| **Canary** | Small % of traffic to new version, gradually increase | Minimal blast radius, data-driven | Complex setup, needs robust monitoring | High-traffic services, risky changes |

**Automated rollback:**
- **Kubernetes**: Rolls back automatically if `progressDeadlineSeconds` is exceeded.
- **Metric-based**: Argo Rollouts/Flagger monitors error rate and latency, rolls back on threshold breach.
- **Pipeline-based**: Smoke tests post-deploy trigger a revert on failure.

**Interview signal:** Articulates cost vs. complexity vs. risk trade-offs. Notes that database migrations must be backward-compatible regardless of strategy.

---

## Linux & OS Fundamentals

### 4. High Load with Low CPU

**Question:** A server has high load average but low CPU usage. How do you diagnose it?

**Answer:**

High load with low CPU means processes are blocked on something other than CPU — typically **I/O wait**, **swap**, or **locks**.

Load average includes processes in **running state** and **uninterruptible sleep (D state)**. High D-state = I/O bottleneck.

**Diagnostic steps:**
1. `uptime` — Compare load average to CPU count (`nproc`).
2. `top` — Check `%wa` (I/O wait) column.
3. `iostat -x 1` — High `%util` or `await` = disk bottleneck.
4. `vmstat 1` — Check `si/so` (swap activity) and `wa` columns.
5. `iotop` — Find which processes are doing the most I/O.
6. `free -h` — Check if the system is memory-starved and swapping.

**Common causes:** slow disk/storage, swap thrashing from memory leaks, database lock contention, or stale NFS mounts causing processes to hang.

**Interview signal:** Knows load includes blocked processes (not just CPU). Mentions I/O wait, D-state, and swap. Familiarity with `iotop`, `iostat`, `vmstat` indicates real experience.

---

## Docker & Containers

### 5. Container Internals & Image Optimization

**Question:** How does Docker work under the hood? How do you optimize container images?

**Answer:**

Containers are **isolated processes sharing the host kernel** — not VMs. Three core Linux technologies make them work:

- **Namespaces** — Isolation (pid, net, mnt, uts, ipc, user). Each container gets its own view of system resources.
- **cgroups** — Resource limits (CPU, memory, I/O). Prevents a single container from consuming all host resources.
- **Union filesystems (Overlay2)** — Layered images. Each layer is read-only; a thin writable layer is added at runtime. Shared base layers save disk space.

**Image optimization:**
- **Multi-stage builds** — Build tools stay out of the final image.
- **Minimal base images** — Use `alpine`, `distroless`, or `scratch` for smaller attack surface.
- **Layer ordering** — Put stable dependencies first, app code last for better cache hits.
- **`.dockerignore`** — Exclude `node_modules`, `.git`, `.env` from build context.
- **Non-root user** — Limits damage if the container is compromised.
- **Pin image digests** — `FROM node:20@sha256:abc...` for reproducible builds.

**Interview signal:** Understands containers ≠ VMs. Explains namespaces, cgroups, and overlay filesystems clearly. Multi-stage builds and non-root users are table stakes.

---

## Summary Table

| # | Topic | Type | Level |
|---|-------|------|-------|
| 1 | Supply Chain Attacks | Conceptual + Practical | Core |
| 2 | SDLC & DevOps Integration | Conceptual | Core |
| 3 | Deployment Strategies | Practical + Scenario | Core |
| 4 | Linux High Load Debug | Troubleshooting | Core |
| 5 | Container Internals & Optimization | Conceptual + Practical | Core |
