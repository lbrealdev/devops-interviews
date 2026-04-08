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

**Question:** What does SDLC stand for and what does it mean? Give a concrete example of how DevOps interacts with it in practice.

**Answer:**

**SDLC** (Software Development Life Cycle) is the structured process teams follow to plan, build, test, deploy, and maintain software — from initial concept to retirement.

**Concrete example — deploying a new API endpoint:**

1. **Planning**: Product defines requirements in Jira. DevOps ensures the ticket includes infrastructure needs.
2. **Development**: DevOps provides a DevContainer so the local environment matches production.
3. **CI Pipeline**: On PR, GitHub Actions runs tests, linting, and a container build. DevOps maintains this pipeline.
4. **Deploy to Staging**: ArgoCD detects the new image tag in Git and syncs to staging automatically.
5. **QA / Testing**: DevOps ensures the environment is stable and seeded with test data.
6. **Production Deploy**: PR merges to `main`. ArgoCD syncs to prod with a canary rollout — 10% traffic first, then gradual promotion.
7. **Monitoring**: Prometheus alerts if error rates spike. DevOps owns the alerting rules and runbooks.

DevOps doesn't own a single stage — it **connects** them by automating handoffs so code flows from commit to production with minimal friction.

**Interview signal:** Gives a concrete end-to-end example, not just a list. Explains how DevOps collapses dev/ops silos into a continuous feedback loop.

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

## AI / LLM

### 4. LLM vs AI Agent in DevOps

**Question:** What's the difference between an LLM and an AI Agent? Which daily DevOps tasks can each support?

**Answer:**

**LLM** (Large Language Model) — A stateless model that generates text from a prompt. It explains, summarizes, and creates — but doesn't take action or maintain context across interactions.

**AI Agent** — Uses an LLM as its reasoning engine but adds **tool use**, **memory**, and **autonomy**. It can plan, execute actions (call APIs, run commands), observe results, and iterate until a goal is achieved.

| Aspect | LLM | AI Agent |
|--------|-----|----------|
| **What it does** | Text in, text out | Plans, acts, observes, iterates |
| **State** | Stateless per request | Maintains context across steps |
| **Autonomy** | None — you drive each prompt | Can make decisions and loop |

**Daily DevOps tasks an LLM can support:**
- Explain error logs — paste a stack trace, get a root cause hypothesis.
- Generate configs — scaffold Terraform, Helm charts, or GitHub Actions workflows.
- Write queries — build PromQL, KQL, or SQL from natural language.
- Draft documentation — runbooks, onboarding guides, ADRs from existing code.
- Code review — flag anti-patterns or missing error handling in PRs.

**Daily DevOps tasks an AI Agent can support:**
- Incident triage — detect an alert, check logs, inspect pod status, correlate with recent deploys, suggest a fix.
- Auto-remediation — restart a crashed service, clear a full disk, or roll back a failed deployment.
- Drift detection — compare live infrastructure to IaC state and open a ticket or PR to reconcile.
- Cost optimization — analyze cloud spend, identify underutilized resources, propose right-sizing changes.

**Key caveat:** LLMs can be confidently wrong — always review output. Agents need guardrails in production: human-in-the-loop approvals, audit logging, and bounded permissions.

**Interview signal:** Clearly distinguishes LLM (explains) from Agent (acts). Gives concrete DevOps examples for both. Mentions guardrails and the risk of blind trust in AI output.

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
| 2 | SDLC & DevOps Integration | Conceptual + Example | Core |
| 3 | Deployment Strategies | Practical + Scenario | Core |
| 4 | LLM vs AI Agent in DevOps | Conceptual + Practical | Desirable |
| 5 | Container Internals & Optimization | Conceptual + Practical | Core |
