# DevOps Interview Questions — Set 2 (Cheat Sheet)

Quick reference for interviewers. Questions with concise expected answers.

---

## IaC

**1. What are Terraform modules and why are they important?**

A module encapsulates related resources into a reusable, versioned unit. Promotes DRY, consistency across teams, and abstraction of complexity. Structure: `main.tf`, `variables.tf`, `outputs.tf`, `versions.tf`.

**2. What is Terraform state and what are `plan` vs `apply`?**

State (`terraform.tfstate`) maps your config to real resources. It's how Terraform knows what exists and what changed. `plan` shows a preview of changes without applying them. `apply` executes those changes against the actual infrastructure. Always `plan` before `apply`.

---

## Cloud

**3. What is serverless computing and when does it make sense?**

Deploy code without managing servers. Scales automatically, pay-per-use. Good for variable traffic, event-driven tasks, and glue/automation. Not ideal for long-running or latency-sensitive workloads. Example: an S3 upload triggers a Lambda that resizes images.

**4. How would you reduce cloud costs?**

Right-size instances, use reserved/savings plans for steady workloads, spot instances for fault-tolerant jobs, auto-scaling, serverless for bursty traffic, tag everything for cost allocation, set budgets/alerts, decommission unused resources.

---

## Git / VCS

**5. Compare trunk-based development and GitFlow.**

Trunk-based: commit directly to main (or short-lived branches). Requires strong CI, feature flags. GitFlow: long-lived branches (`develop`, `release/*`, `hotfix/*`). More complex, suited for versioned releases and regulated environments. Most modern teams lean trunk-based.

**6. What are interactive rebase, cherry-pick, and bisect?**

Rebase: rewrite/squash/reorder commits before merging. Cherry-pick: apply a specific commit from one branch to another (e.g., backport a fix). Bisect: binary search through history to find which commit introduced a bug.

---

## AI / LLM

**7. How would you use LLMs in your DevOps work?**

Incident triage (summarize logs, suggest root causes), IaC scaffolding, documentation generation, query building (PromQL, KQL), script authoring, AI-assisted code review. Key: always review output, never feed secrets to external APIs.

**8. What's the difference between an LLM and an AI agent?**

LLM: stateless text generation — prompt in, response out. Agent: uses an LLM as a reasoning engine but adds tool use, memory, and autonomy — can plan, execute actions, and iterate toward a goal. LLMs explain, agents act.

---

## Security

**9. How do you manage secrets in Kubernetes?**

K8s Secrets are base64, not encrypted — not enough alone. Options: Sealed Secrets (encrypt for Git storage), External Secrets Operator (sync from vaults like AWS SM or HashiCorp Vault), Vault itself (dynamic secrets, leasing, audit logging). Best practices: rotate automatically, restrict via RBAC, enable etcd encryption at rest, prefer workload identity over static credentials.

---

## Networking

**10. How does networking work in Kubernetes?**

CNI plugins (Calico, Cilium, AWS VPC CNI) give each pod an IP. Services expose pods: ClusterIP (internal), NodePort, LoadBalancer. Ingress controllers route external HTTP traffic by host/path. Network Policies restrict pod-to-pod traffic (default is allow-all). CoreDNS handles service discovery.
