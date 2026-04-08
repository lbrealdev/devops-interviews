# DevOps Interview Questions

Technical interview questions for a DevOps Engineer position.

---

## Core Skills

### 1. Kubernetes Workloads

**Question:** Explain the difference between a Deployment, StatefulSet, and DaemonSet. When would you use each?

**Answer:**

- **Deployment**: Manages stateless workloads. Creates ReplicaSets that ensure a specified number of pod replicas are running. Pods are interchangeable and can be rescheduled to any node. Use for web servers, APIs, stateless microservices.
- **StatefulSet**: Manages stateful workloads. Provides stable network identities, persistent storage, and ordered deployment/scaling. Use for databases, message queues, or any service requiring stable identity (e.g., PostgreSQL, Kafka, Redis).
- **DaemonSet**: Ensures a pod runs on every (or selected) node. Automatically adds pods to new nodes. Use for node-level agents like log collectors (Fluentd), monitoring agents (Prometheus Node Exporter), or CNI plugins.

---

### 2. Helm & ArgoCD (GitOps)

**Question:** How does a GitOps workflow using Helm and ArgoCD work end-to-end? What happens when you push a change to the repo?

**Answer:**

1. **Source of Truth**: Application manifests (Helm charts + values files) are stored in a Git repository.
2. **Change Trigger**: A developer pushes a change (e.g., image tag update in `values.yaml`) and opens a PR.
3. **CI Pipeline**: GitHub Actions runs linting, validation, and tests against the Helm chart.
4. **Merge to Main**: Upon merge, ArgoCD detects the change via polling or webhook.
5. **Sync**: ArgoCD compares the desired state (Git) with the live state (cluster). If there's drift, it applies the changes using `helm template` to render manifests and syncs them to the cluster.
6. **Health Checks**: ArgoCD monitors the health and sync status of the application, reporting any issues.

Key concepts: declarative configuration, single source of truth, automated reconciliation, audit trail via Git history.

---

### 3. Terraform (IaC)

**Question:** What is Terraform state and why is it important? How would you manage state safely in a team environment?

**Answer:**

**State** is a JSON file (`terraform.tfstate`) that maps your Terraform configuration to real-world resources. It tracks resource metadata, dependencies, and current attributes so Terraform knows what exists and what needs to change.

**Why it matters**: Without state, Terraform cannot determine the difference between desired and actual infrastructure.

**Safe team practices:**
- **Remote backend**: Store state remotely (e.g., S3 + DynamoDB for locking, Terraform Cloud, Azure Blob Storage).
- **State locking**: Prevent concurrent modifications that could corrupt state. DynamoDB or equivalent provides this.
- **Workspaces or separate state files**: Isolate environments (dev, staging, prod) to limit blast radius.
- **Access control**: Restrict who can read/write state, as it may contain sensitive data (passwords, keys).
- **Versioning**: Enable versioning on the backend storage for recovery.

---

### 4. GitHub Actions (CI/CD)

**Question:** Describe a GitHub Actions workflow you've built. How do you handle secrets, caching, and reusable workflows?

**Answer:**

A typical workflow might look like:

```yaml
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm test
      - run: npm run build
```

- **Secrets**: Stored in GitHub repository/org secrets (`${{ secrets.MY_SECRET }}`). Never hardcoded. Accessible at the job/step level. Use OIDC for cloud provider authentication when possible (no long-lived keys).
- **Caching**: Use `actions/cache` or built-in cache support (e.g., `cache: npm` in `setup-node`) to cache dependencies between runs and reduce build times.
- **Reusable Workflows**: Use `workflow_call` to create shared workflows stored in a central repository. Reference them with `uses: org/repo/.github/workflows/shared.yml@main` to enforce consistency across repositories.

---

### 5. EFK/ELK (Logging)

**Question:** Walk me through how a log message travels from an application container to being searchable in Kibana. What would you do if logs stop appearing?

**Answer:**

**Log flow:**

1. **Application** writes logs to stdout/stderr.
2. **Fluentd / Fluent Bit** (running as DaemonSet) collects logs from container runtime (via `/var/log/containers/*.log`).
3. **Parsing & Enrichment**: Fluentd applies filters to parse JSON, add metadata (namespace, pod name, labels).
4. **Elasticsearch**: Fluentd ships parsed logs to Elasticsearch, where they are indexed.
5. **Kibana**: Users query Elasticsearch via Kibana dashboards and discover interfaces.

**Troubleshooting if logs stop:**
1. Check Fluentd/Fluent Bit pods: `kubectl get pods -n logging` — are they running?
2. Check Fluentd logs: `kubectl logs <fluentd-pod>` for errors (connection refused, parsing errors).
3. Verify Elasticsearch health: check cluster status (green/yellow/red), disk usage, index count.
4. Check if indices are being rotated or if ILM policies are blocking writes.
5. Verify network connectivity between logging namespace and Elasticsearch.
6. Check if log volume has spiked (backpressure, buffer issues).

---

### 6. ALM Governance

**Question:** What does Application Lifecycle Management governance mean to you? How would you enforce quality gates or approval processes in a CI/CD pipeline?

**Answer:**

ALM governance is the set of policies, processes, and controls that ensure software is developed, delivered, and maintained consistently, securely, and in compliance with organizational standards.

**Quality gates in CI/CD:**
- **Code quality**: Enforce linting, static analysis (SonarQube), and minimum code coverage thresholds.
- **Security**: Run SAST/DAST scans, dependency vulnerability checks (Dependabot, Snyk). Block merges on critical CVEs.
- **Approval workflows**: Require manual approvals for production deployments (GitHub Environments with required reviewers).
- **Branch protection**: Enforce PR reviews, status checks passing, signed commits.
- **Change management**: Integrate with ticketing systems (Jira, ServiceNow) to link deployments to approved changes.
- **Compliance**: Maintain audit logs, enforce tagging standards, use policy-as-code (OPA, Kyverno).

---

### 7. Troubleshooting Scenario

**Question:** A Kubernetes pod is stuck in CrashLoopBackOff. Walk me through your step-by-step troubleshooting process.

**Answer:**

1. **Get pod info**: `kubectl get pods` — confirm the status is `CrashLoopBackOff` and check restart count.
2. **Describe the pod**: `kubectl describe pod <pod>` — check events, last state, exit codes.
   - Exit code `0`: Application exited normally (misconfigured command).
   - Exit code `1`: Application error.
   - Exit code `137`: OOMKilled (memory limit exceeded).
   - Exit code `139`: Segmentation fault.
3. **Check logs**: `kubectl logs <pod>` and `kubectl logs <pod> --previous` — read the application output from the current and previous crashed instance.
4. **Common causes**:
   - Missing or incorrect environment variables / config maps / secrets.
   - Failed health checks (liveness probe failing too early).
   - Application dependencies not available (database down, missing migrations).
   - Resource limits too low (OOMKilled).
   - Incorrect image or entrypoint.
5. **Exec into the pod** (if it stays up briefly): `kubectl exec -it <pod> -- sh` to inspect filesystem, connectivity, env vars.
6. **Check dependencies**: Verify services, configmaps, secrets, PVCs the pod depends on.

---

## Desirable Skills

### 8. Monitoring & Observability

**Question:** What is the difference between monitoring and observability? What role do metrics, logs, and traces play (the three pillars)?

**Answer:**

- **Monitoring**: Collecting and alerting on predefined known metrics (CPU, memory, error rates). Answers "is the system healthy?" You define what to watch ahead of time.
- **Observability**: The ability to understand the internal state of a system by examining its external outputs. Answers "why is the system behaving this way?" Supports exploratory debugging of unknown issues.

**Three pillars:**

| Pillar | What it answers | Tools |
|--------|----------------|-------|
| **Metrics** | How much / how fast / how often? (quantitative, time-series) | Prometheus, Grafana, Datadog |
| **Logs** | What happened? (discrete events) | EFK/ELK, Loki |
| **Traces** | What was the request path and where was time spent? (distributed request flow) | Jaeger, Tempo, Zipkin |

A mature observability strategy combines all three and correlates them (e.g., jump from a metric spike to related traces and logs).

---

### 9. Service Mesh & Multi-Cluster

**Question:** What problems does a service mesh like Istio or Linkerd solve? How would you approach GitOps across multiple Kubernetes clusters?

**Answer:**

**Service Mesh** (Istio, Linkerd):
- **mTLS**: Automatic mutual TLS between services — encrypts traffic without code changes.
- **Traffic management**: Canary deployments, traffic splitting, retries, timeouts, circuit breaking.
- **Observability**: Automatic metrics, traces, and access logs for all service-to-service traffic.
- **Security policies**: Fine-grained authorization policies (which service can call which).

**Multi-cluster GitOps:**
- Use an **ApplicationSet** controller in ArgoCD to template applications across clusters from a single definition.
- Define cluster-specific `values.yaml` overlays (dev, staging, prod) using Kustomize or Helm.
- Store cluster registrations and access credentials securely (ArgoCD cluster secrets).
- Use **progressive delivery** strategies: deploy to dev first, then staging, then prod with promotion gates.
- Consider a **hub-and-spoke** model or **pull-based** agents on each cluster for security (clusters pull config rather than central system pushing).

---

### 10. AI for Productivity

**Question:** How have you used (or would you use) AI tools to improve your DevOps workflows? Give a concrete example.

**Answer:**

Practical examples of AI in DevOps:

- **IaC generation**: Use AI (Copilot, ChatGPT) to scaffold Terraform modules, Helm templates, or GitHub Actions workflows, then review and refine.
- **Incident analysis**: Feed log excerpts or error messages to AI for faster root cause hypotheses during incidents.
- **Code review assistance**: AI-powered PR review to catch security issues, anti-patterns, or suggest optimizations.
- **Documentation**: Auto-generate runbooks, architecture diagrams descriptions, or onboarding docs from existing code and configs.
- **Script writing**: Generate shell scripts, migration scripts, or one-off data transformation tasks.
- **Query generation**: Build complex Kibana queries, Prometheus PromQL, or SQL queries from natural language descriptions.

**Key principle**: AI accelerates output, but human review is essential. Use it as a productivity multiplier, not a replacement for understanding.

---

## Summary Table

| # | Topic | Type | Level |
|---|-------|------|-------|
| 1 | Kubernetes workloads | Conceptual | Core |
| 2 | Helm + ArgoCD GitOps | Conceptual + Practical | Core |
| 3 | Terraform state | Conceptual | Core |
| 4 | GitHub Actions | Practical | Core |
| 5 | EFK/ELK logging | Practical + Troubleshooting | Core |
| 6 | ALM Governance | Conceptual | Core |
| 7 | Pod CrashLoopBackOff | Scenario / Debug | Core |
| 8 | Monitoring vs Observability | Conceptual | Desirable |
| 9 | Service Mesh + Multi-cluster | Conceptual | Desirable |
| 10 | AI in DevOps | Open-ended | Desirable |
