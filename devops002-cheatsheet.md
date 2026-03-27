# DevOps Interview Questions — Set 2 (Cheat Sheet)

Quick reference for interviewers. Questions with concise expected answers.

---

## IaC

**1. What are Terraform modules and why are they important?**

A module encapsulates related resources into a reusable, versioned unit. Enforces DRY, consistency, and standards across teams. One source of truth — change once, every consumer gets the update. Pin provider versions inside the module and publish to a private registry with semantic versioning.

**2. What is Terraform state and what are `plan` vs `apply`?**

State (`terraform.tfstate`) maps your config to real infrastructure — it's how Terraform knows what exists and what changed. Without state, it can't tell drift from intent. `plan` previews changes before touching anything. `apply` executes them. Always `plan` before `apply`.

---

## Cloud

**3. What is serverless and when does it make sense?**

Deploy code without managing servers. Scales automatically to zero, pay-per-use, event-driven. Ideal for variable traffic, webhooks, glue tasks, and scheduled jobs. Trade-offs: cold start latency, stateless only, not suited for long-running processes. Example: S3 upload triggers Lambda to process the file — no persistent server needed.

**4. How would you reduce cloud costs?**

Right-size instances (most over-provisioned), use Reserved/Savings Plans for steady baseline, Spot for fault-tolerant batch jobs. Auto-scale to match real demand, serverless for bursty traffic. Tag everything so you know who's spending what. Set budgets and alerts. FinOps mindset: make engineering teams own the cost of what they deploy — show costs in their dashboards.

---

## Git / VCS

**5. Compare trunk-based development and GitFlow.**

Trunk-based: small commits straight to `main` (or merged in 1-2 days), feature flags hide incomplete work, needs strong CI. Fast feedback loops, continuous delivery. GitFlow: long-lived branches (`develop`, `release/*`, `hotfix/*`), more ceremony, suited for versioned releases or regulated environments. Modern teams building cloud-native software lean trunk-based.

**6. What are interactive rebase, cherry-pick, and git bisect?**

Rebase: rewrite/squash/reorder commits before merging — keep history clean. Cherry-pick: apply one specific commit from any branch onto another — useful for backporting a fix without merging everything. Bisect: binary search through commit history to isolate which commit introduced a bug. Much faster than testing every commit manually.

---

## AI / LLM

**7. How would you use LLMs in your DevOps work?**

Log summarization and incident triage (paste error → get root cause hypothesis), IaC scaffolding (describe what you want in words, refine the output), query building (PromQL, KQL, SQL from natural language), documentation generation, script authoring, AI-assisted code review. Always review the output — LLMs are confidently wrong sometimes.

**8. What's the difference between an LLM and an AI agent?**

LLM: stateless text generation — prompt in, response out. AI Agent: uses an LLM as a reasoning engine but adds tool use, memory, and autonomy — plans steps, executes actions, observes results, iterates. LLMs explain. Agents act. Example: an LLM describes why a pod crashed; an agent would detect the crash, check logs, identify the issue, and restart it.

---

## Security

**9. How do you manage secrets in Kubernetes?**

K8s Secrets are base64-encoded, not encrypted — not production-safe alone. Best approach: use Vault (dynamic secrets, lease rotation, audit logs) or External Secrets Operator (syncs from AWS SM, GCP SM, Vault into K8s Secrets). For GitOps-friendly: Sealed Secrets encrypts secrets so they can live in Git. Never commit plain secrets to Git. Prefer workload identity (IRSA, Workload Identity Federation) over static long-lived credentials.

---

## Networking

**10. How does networking work in Kubernetes?**

CNI plugins (Calico, Cilium, AWS VPC CNI) assign each pod a unique IP and handle pod-to-pod routing across nodes. Services (ClusterIP, NodePort, LoadBalancer) expose pods as stable endpoints — load balance traffic across pod replicas. Ingress controllers route external HTTP/HTTPS traffic by host or path (NGINX Ingress, Traefik). Network Policies act as a firewall — default allows all pod-to-pod traffic, you restrict what's needed. CoreDNS handles service discovery: `curl my-svc.namespace.svc.cluster.local` resolves to the service IP.

---

## Summary Table

| # | Topic | Level |
|---|-------|-------|
| 1 | Terraform Modules | Core |
| 2 | Terraform State / Plan / Apply | Core |
| 3 | Serverless Computing | Core |
| 4 | Cloud Cost Optimization | Core |
| 5 | Git Branching Strategies | Core |
| 6 | Advanced Git | Core |
| 7 | LLMs in DevOps | Desirable |
| 8 | LLM vs AI Agent | Desirable |
| 9 | Secrets Management | Core |
| 10 | Kubernetes Networking | Core |
