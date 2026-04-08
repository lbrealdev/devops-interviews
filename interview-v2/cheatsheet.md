# DevOps Interview Questions — Set 2 (Cheat Sheet)

Quick reference for interviewers. Questions with concise expected answers.

---

## IaC

**1. What are Terraform modules and why are they important?**

A module encapsulates related resources into a reusable, versioned unit. Enforces DRY, consistency, and standards across teams. One source of truth — change once, every consumer gets the update. Pin provider versions and publish to a private registry with semantic versioning.

**2. What is Terraform state and why does it matter?**

State (`terraform.tfstate`) maps your config to real infrastructure. Without it, Terraform can't tell what's actually deployed versus what you wrote in code. It tracks dependencies, resource IDs, and current attributes. Key rule: never edit state manually and never store it locally in a team environment — use a remote backend (S3 + DynamoDB, Terraform Cloud) with state locking to prevent concurrent changes from corrupting it.

---

## Cloud

**3. What is serverless and when does it make sense?**

Deploy code without managing servers. Scales automatically to zero, pay-per-use, event-driven. Ideal for variable traffic, webhooks, glue tasks, and scheduled jobs. Trade-offs: cold start latency, stateless only, not suited for long-running processes. Example: S3 upload triggers Lambda to process the file — no persistent server needed.

**4. How would you reduce cloud costs?**

Right-size instances (most are over-provisioned). Use Reserved/Savings Plans for steady baseline, Spot for fault-tolerant batch jobs. Auto-scale to match real demand, go serverless for bursty traffic. Tag everything so you know who's spending what, set budgets and alerts. FinOps mindset: make engineering teams own the cost of what they deploy — show costs in their dashboards, not just the finance team's.

---

## Git / VCS

**5. Compare trunk-based development and GitFlow.**

Trunk-based: small commits to `main` (or short-lived branches merged in 1-2 days), feature flags hide incomplete work, strong CI required. Fast feedback, continuous delivery. GitFlow: long-lived branches (`develop`, `release/*`, `hotfix/*`), more ceremony, suited for versioned releases or regulated environments. Most modern cloud-native teams lean trunk-based.

**6. When would you use git merge versus git rebase?**

Merge preserves history as-is — creates a merge commit and keeps all commits separate. Safe for shared branches, makes the timeline accurate but noisier. Rebase rewrites commits on top of the target branch, producing a linear, clean history — use it on feature branches before merging to `main`, never on shared/public branches. Rule of thumb: rebase to keep your branch tidy, merge to integrate completed work. Rebase rewrites history, so it's only for local or branch-level commits.

---

## AI / LLM

**7. How would you use LLMs in your DevOps work?**

Log summarization and incident triage (paste an error, get a root cause hypothesis), IaC scaffolding (describe infra in words, refine the output), query building (PromQL, KQL, SQL from natural language), documentation generation, script authoring, AI-assisted code review. Always review the output — LLMs are confidently wrong sometimes.

**8. What's the difference between an LLM and an AI agent?**

LLM: stateless text generation — prompt in, response out. AI Agent: uses an LLM as reasoning engine but adds tool use, memory, and autonomy — plans steps, executes actions, observes results, iterates. LLMs explain. Agents act. Example: an LLM describes why a pod crashed; an agent would detect it, check logs, identify the issue, and restart it.

---

## Cloud Security

**9. How do Security Groups and NACLs work in AWS?**

Security Groups (SGs) are stateful — if you allow inbound port 80, outbound is automatically allowed back. They're attached to ENIs (network interfaces) on instances and act as an instance-level firewall. NACLs are stateless and operate at the subnet level — you must explicitly allow both directions. Rules are evaluated in order by rule number. Best practice: lock down SGs to least privilege (only allow what each service needs), use NACLs as an additional perimeter layer. SGs are the primary control, NACLs are for broad subnet-level blocks (e.g., block a malicious IP range across the entire subnet).

---

## Cloud Networking

**10. How does traffic flow in a VPC? Walk me through the basics.**

A VPC is an isolated network. Inside it you have subnets (public and private), route tables that define where traffic goes, an Internet Gateway (IGW) for public internet access, and NAT Gateways for private subnets to initiate outbound internet access without being reachable from outside. Security Groups and NACLs control who can talk to what. For internal service-to-service communication, you use private DNS or private link (VPC endpoint) — no IGW involved. Think of it as layers: NACLs (subnet level) → Security Groups (instance/ENI level) → routing (where to send traffic) → IGW/NAT (how to reach the internet).

---

## Summary Table

| # | Topic | Level |
|---|-------|-------|
| 1 | Terraform Modules | Core |
| 2 | Terraform State | Core |
| 3 | Serverless Computing | Core |
| 4 | Cloud Cost Optimization | Core |
| 5 | Git Branching Strategies | Core |
| 6 | Git Merge vs Rebase | Core |
| 7 | LLMs in DevOps | Desirable |
| 8 | LLM vs AI Agent | Desirable |
| 9 | Security Groups / NACLs | Core |
| 10 | VPC Networking | Core |
