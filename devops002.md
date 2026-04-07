# DevOps Interview Questions — Set 2

Technical interview questions for a DevOps Engineer position.

---

## IaC (Infrastructure as Code)

### 1. Terraform Modules

**Question:** What are Terraform modules and why are they important? How would you structure a reusable module for your team?

**Answer:**

A **module** is a container for multiple resources that are used together. It encapsulates related infrastructure (e.g., a VPC, an ECS cluster, an RDS instance) into a reusable, versioned unit.

**Why they matter:**
- **DRY**: Define infrastructure patterns once, reuse across environments and teams.
- **Consistency**: Enforce standards (naming conventions, security baselines) across all consumers.
- **Abstraction**: Expose only the inputs that should be configurable, hiding internal complexity.

**Module structure:**

```
modules/
  ecs-service/
    main.tf          # Resource definitions
    variables.tf     # Input variables with descriptions and defaults
    outputs.tf       # Exported values for consumers
    versions.tf      # Provider and Terraform version constraints
    README.md        # Usage documentation
```

**Best practices:**
- Pin provider versions inside the module.
- Use `validation` blocks on variables to enforce constraints early.
- Publish to a private registry (Terraform Cloud, Git) with semantic versioning.
- Keep modules small and focused — one module per logical unit.

---

### 2. Terraform vs Pulumi vs CDK

**Question:** Compare Terraform, Pulumi, and AWS CDK. What are the trade-offs and when would you choose one over the other?

**Answer:**

| Aspect | Terraform | Pulumi | AWS CDK |
|--------|-----------|--------|---------|
| **Language** | HCL (domain-specific) | General-purpose (Python, TS, Go, etc.) | General-purpose (Python, TS, Java, etc.) |
| **State** | Self-managed or Terraform Cloud | Self-managed or Pulumi Cloud | Synthesized into CloudFormation (managed by AWS) |
| **Ecosystem** | Largest provider support (multi-cloud) | Good multi-cloud support | AWS-focused (constructs for other clouds exist) |
| **Maturity** | Most established, huge community | Growing, strong developer experience | Mature for AWS, limited outside it |
| **Learning curve** | Low for simple use cases | Familiar if you know the language | Familiar if you know AWS + a language |

**When to choose:**
- **Terraform**: Multi-cloud, established teams, need for broad provider support, strong opinion on declarative IaC.
- **Pulumi**: Teams comfortable with programming languages, need for complex logic (loops, conditionals, abstractions) that HCL struggles with.
- **AWS CDK**: AWS-only shops, teams that want to stay close to CloudFormation without writing raw templates, alignment with AWS-native tooling.

**Key trade-off**: HCL is intentionally limited, which forces simplicity and readability. General-purpose languages offer more power but also more room for overly complex code.

---

## Cloud

### 3. Serverless Computing

**Question:** What is serverless computing, when does it make sense to use it, and can you give a practical example?

**Answer:**

**Serverless** means you deploy code without managing servers. The cloud provider provisions, scales, and manages the infrastructure automatically. You pay only for the compute time consumed (per-invocation pricing).

**Key characteristics:**
- No server provisioning or patching.
- Scales automatically from zero to peak.
- Pay-per-use (no cost when idle).
- Stateless and event-driven by nature.

**When it makes sense:**
- **Variable/unpredictable traffic** — scales to zero, no idle cost.
- **Event-driven workloads** — file uploads, webhooks, scheduled tasks, API endpoints.
- **Glue/automation tasks** — orchestrate services without standing infrastructure.
- **Rapid prototyping** — deploy fast, iterate quickly.

**When it doesn't:**
- Long-running processes (use containers/VMs instead).
- Latency-sensitive workloads (cold starts).
- Heavy stateful processing.

**AWS Example — S3 Image Thumbnail Generator:**

1. User uploads an image to an S3 bucket.
2. S3 triggers an **AWS Lambda** function.
3. Lambda resizes the image and saves the thumbnail back to S3.
4. No servers to manage. Costs nothing when no images are uploaded.

```python
import boto3
from PIL import Image

s3 = boto3.client("s3")

def handler(event, context):
    bucket = event["Records"][0]["s3"]["bucket"]["name"]
    key = event["Records"][0]["s3"]["object"]["key"]

    # Download, resize, upload
    s3.download_file(bucket, key, "/tmp/image.jpg")
    img = Image.open("/tmp/image.jpg")
    img.thumbnail((200, 200))
    img.save("/tmp/thumb.jpg")
    s3.upload_file("/tmp/thumb.jpg", bucket, f"thumbs/{key}")
```

---

### 4. Cloud Cost Optimization

**Question:** How would you approach reducing cloud costs in an organization? What strategies and tools would you use?

**Answer:**

Cost optimization is an ongoing discipline, not a one-time task. Key strategies:

**Right-sizing:**
- Analyze actual CPU, memory, and storage utilization.
- Downsize over-provisioned instances (e.g., an EC2 running at 5% CPU).
- Use tools like AWS Compute Optimizer or GCP Recommender.

**Pricing models:**
- **Reserved Instances / Savings Plans**: Commit to 1-3 year terms for predictable baseline workloads (up to 70% savings).
- **Spot Instances**: Use for fault-tolerant, flexible workloads (batch jobs, CI runners, data processing) — up to 90% savings.
- **On-demand**: For unpredictable or short-lived workloads.

**Architecture decisions:**
- Serverless for bursty/event-driven workloads.
- Auto-scaling to match demand instead of running at peak capacity 24/7.
- Use managed services (RDS, EKS) to reduce operational overhead.

**Visibility & governance:**
- Tag everything (team, environment, project) for cost allocation.
- Set budgets and alerts (AWS Budgets, GCP Billing Alerts).
- Use dashboards (AWS Cost Explorer, FinOps tools like Kubecost for Kubernetes).
- Review and decommission unused resources (old snapshots, idle load balancers, unattached volumes).

**FinOps culture:** Involve engineering teams in cost ownership. Make cost a visible metric alongside performance and reliability.

---

## Git / Version Control

### 5. Branching Strategies

**Question:** Compare trunk-based development and GitFlow. When would you use each?

**Answer:**

**Trunk-Based Development:**
- Developers commit directly to `main` (or through short-lived feature branches merged within 1-2 days).
- Relies heavily on feature flags to hide incomplete work.
- Requires strong CI, fast tests, and automated quality gates.

**GitFlow:**
- Uses long-lived branches: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`.
- Features branch off `develop`, merge back via PR.
- Releases are cut from `develop` into `release/*`, then merged to `main`.

| Aspect | Trunk-Based | GitFlow |
|--------|-------------|---------|
| **Merge frequency** | Continuous (multiple times/day) | Periodic (per feature or release) |
| **Branch lifespan** | Hours to 1-2 days | Days to weeks |
| **Release cadence** | Continuous / on-demand | Scheduled releases |
| **Complexity** | Low | High |
| **Best for** | Fast-moving teams, CD pipelines | Regulated environments, versioned releases |

**When to choose:**
- **Trunk-based**: Teams practicing continuous delivery, fast feedback loops, mature CI/CD.
- **GitFlow**: Teams with strict release windows, compliance requirements, or products that ship versioned releases (e.g., mobile apps, on-prem software).

Most modern DevOps teams lean toward trunk-based development with feature flags.

---

### 6. Advanced Git

**Question:** Explain interactive rebase, cherry-pick, and bisect. When would you use each?

**Answer:**

**Interactive Rebase (`git rebase -i`):**
Rewrites commit history before merging. You can squash, reorder, edit, or drop commits.

```
pick abc1234 Add user authentication
squash def5678 Fix typo in auth module
pick ghi9012 Add login endpoint
```

Use when: Cleaning up a feature branch before merging to keep history clean and meaningful.

**Cherry-Pick (`git cherry-pick <commit>`):**
Applies a specific commit from one branch onto another without merging the entire branch.

Use when: You need to backport a bug fix from `main` to a release branch, or apply a specific change across environments without bringing in unrelated commits.

**Bisect (`git bisect`):**
Binary search through commit history to find which commit introduced a bug.

```bash
git bisect start
git bisect bad           # Current commit is broken
git bisect good abc1234  # This older commit was working
# Git checks out a midpoint — test it, then mark as good/bad
git bisect good / git bisect bad
# Repeat until Git identifies the culprit
```

Use when: A regression was introduced but you don't know which commit caused it. Much faster than manually testing every commit.

---

## AI / LLM

### 7. Leveraging LLMs in DevOps

**Question:** How would you leverage LLMs to improve your day-to-day DevOps work?

**Answer:**

LLMs are best used as productivity accelerators, not replacements for judgment. Practical applications:

- **Incident triage**: Feed error logs or alert descriptions to an LLM for faster root cause hypotheses. It can summarize verbose logs and suggest next steps.
- **IaC scaffolding**: Generate Terraform modules, Helm charts, or GitHub Actions workflows from natural language descriptions, then refine and review.
- **Documentation**: Auto-generate runbooks, architecture decision records, or onboarding guides from existing code and configurations.
- **Query generation**: Build complex PromQL, Kibana KQL, or SQL queries by describing what you need in plain language.
- **Script authoring**: Quickly generate shell scripts, migration scripts, or one-off data transformations.
- **Code review**: Use AI-assisted PR reviews to catch anti-patterns, missing error handling, or security issues.

**Key principles:**
- Always review AI output — it can be confidently wrong.
- Don't feed secrets or sensitive data into external LLM APIs.
- Use it for speed, not as a substitute for understanding the systems you operate.

---

### 8. LLM vs AI Agent

**Question:** What's the difference between an LLM and an AI agent? Where does each fit in a DevOps workflow?

**Answer:**

**LLM (Large Language Model):**
A model that generates text based on input. It's stateless — given a prompt, it produces a response. It doesn't take actions or maintain memory across interactions unless built into a surrounding system.

**AI Agent:**
A system that uses an LLM as its reasoning engine but adds **tool use**, **memory**, and **autonomy**. An agent can plan, execute actions (call APIs, run commands, query databases), observe results, and iterate until a goal is achieved.

| Aspect | LLM | AI Agent |
|--------|-----|----------|
| **Interaction** | Single prompt → response | Multi-step, goal-driven loop |
| **Actions** | Text output only | Can call tools, APIs, run commands |
| **State** | Stateless (per request) | Maintains context across steps |
| **Autonomy** | None | Can make decisions and iterate |
| **Example** | "Explain this error log" | "Investigate why the pod is crashing, check logs, check dependencies, and suggest a fix" |

**DevOps fit:**
- **LLMs**: Best for generation and explanation tasks — writing configs, summarizing logs, answering questions.
- **Agents**: Best for autonomous workflows — incident response pipelines, auto-remediation, infrastructure drift detection with self-healing.

**Caveat**: Agents require strong guardrails. Autonomous actions in production infrastructure need human-in-the-loop approval, audit logging, and bounded permissions.

---

## Security

### 9. Secrets Management

**Question:** How would you manage secrets in a Kubernetes environment? Compare different approaches.

**Answer:**

Kubernetes `Secrets` objects are base64-encoded by default — not encrypted. They're a starting point, but not sufficient for production security.

**Approaches:**

| Tool | How it works | Pros | Cons |
|------|-------------|------|------|
| **K8s Secrets (native)** | Store in etcd, mount as env/volumes | Simple, built-in | Base64 not encryption, stored in etcd unencrypted at rest |
| **Sealed Secrets** | Encrypt secrets client-side, store in Git, controller decrypts in-cluster | GitOps-friendly, no external dependency | Tied to cluster, can't share across clusters easily |
| **External Secrets Operator** | Syncs secrets from external vaults (AWS Secrets Manager, HashiCorp Vault, GCP SM) into K8s Secrets | Centralized management, rotation support | Requires external vault infrastructure |
| **HashiCorp Vault** | Central secrets engine with dynamic secrets, leasing, and fine-grained policies | Most powerful, dynamic secrets, audit logging | Operational complexity, learning curve |

**Best practices:**
- Never commit plain-text secrets to Git.
- Use short-lived, dynamically generated credentials where possible (Vault, cloud IAM roles).
- Rotate secrets automatically — don't rely on manual rotation.
- Restrict access via RBAC — limit which pods/namespaces can access which secrets.
- Enable encryption at rest for etcd.
- Use workload identity (IRSA, Workload Identity) instead of long-lived static credentials when possible.

---

## Networking

### 10. Kubernetes Networking

**Question:** Explain how networking works in Kubernetes. What are the key components and how do they interact?

**Answer:**

Kubernetes networking is built on three core requirements:
1. Pods can communicate with all other pods without NAT.
2. Nodes can communicate with all pods without NAT.
3. Pods see their own IP as others see it.

**Key components:**

**CNI (Container Network Interface):**
The plugin that provides pod networking. Each pod gets its own IP address from the CNI.
- Common CNIs: Calico, Cilium, Flannel, AWS VPC CNI.
- Handles pod-to-pod routing, sometimes across nodes.

**Services:**
Abstractions that expose pods. Types:
- **ClusterIP** (default): Internal-only virtual IP, load balances across pods.
- **NodePort**: Exposes the service on a port on every node.
- **LoadBalancer**: Provisions an external load balancer (cloud provider).
- **ExternalName**: Maps to a DNS name (CNAME).

**Ingress Controllers:**
Route external HTTP/HTTPS traffic to services based on host/path rules.
- Examples: NGINX Ingress, Traefik, AWS ALB Ingress Controller.
- Handles TLS termination, path-based routing, rate limiting.

**Network Policies:**
Firewall rules for pod-to-pod traffic. Default behavior is all pods can talk to all pods — network policies let you restrict this.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

**DNS:**
CoreDNS runs as a cluster service. Pods resolve service names via `<service>.<namespace>.svc.cluster.local`.

---

## Summary Table

| # | Topic | Type | Level |
|---|-------|------|-------|
| 1 | Terraform Modules | Conceptual + Practical | Core |
| 2 | Terraform vs Pulumi vs CDK | Conceptual | Core |
| 3 | Serverless Computing | Conceptual + Practical | Core |
| 4 | Cloud Cost Optimization | Practical + Strategic | Core |
| 5 | Git Branching Strategies | Conceptual | Core |
| 6 | Advanced Git | Practical | Core |
| 7 | LLMs in DevOps | Open-ended | Desirable |
| 8 | LLM vs AI Agent | Conceptual | Desirable |
| 9 | Secrets Management | Practical | Core |
| 10 | Kubernetes Networking | Conceptual + Practical | Core |
