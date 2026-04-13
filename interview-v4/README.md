# DevOps Interview Questions — Set 4

Advanced security topics for DevOps Engineer positions.

---

## Security

### 1. OIDC in CI/CD

**Question:** How does GitHub OIDC work with cloud providers, and why would you use it over long-lived access keys?

**Answer:**

OIDC (OpenID Connect) allows GitHub Actions to authenticate to cloud providers without storing secrets.

**How it works:**
1. Your workflow requests a token from GitHub's OIDC provider
2. GitHub returns a signed JWT with claims about the workflow (repo, branch, actor)
3. Your pipeline sends this token to the cloud provider (e.g., AWS STS)
4. The cloud provider validates it and issues temporary credentials

**Why it's better than access keys:**
- Short-lived tokens instead of months/years-long keys
- No secrets stored in GitHub
- Automatic rotation per run
- Revocable at any time via GitHub settings
- CloudTrail logs the exact identity that performed the action

**Key concept:** You configure a trust policy in the cloud (e.g., IAM role) that only accepts tokens from your specific repo/branch. No access keys ever leave GitHub.

---

## Summary Table

| # | Topic | Type | Level |
|---|-------|------|-------|
| 1 | OIDC in CI/CD | Conceptual + Practical | Advanced |
