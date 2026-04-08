# DevOps Interview Questions — Set 4

Advanced security topics for DevOps Engineer positions.

---

## Security

### 1. OIDC in CI/CD

**Question:** How does GitHub OIDC work with cloud providers, and why would you use it over long-lived access keys?

**Answer:**

**What is OIDC?**
OIDC (OpenID Connect) is an identity layer on top of OAuth 2.0. It allows a client (GitHub Actions) to verify a user's identity via an authorization server (GitHub).

**How it works in GitHub Actions:**

1. Your workflow requests a token from GitHub's OIDC provider
2. GitHub provides a signed JWT token containing claims about the workflow (repo, branch, actor, etc.)
3. Your CI/CD pipeline sends this token to the cloud provider (e.g., AWS STS)
4. The cloud provider validates the token and issues temporary credentials
5. The workflow uses these temporary credentials to access cloud resources

**Why it's better than access keys:**

| Access Keys | OIDC Tokens |
|------------|-------------|
| Long-lived (months/years) | Short-lived (minutes/hours) |
| Stored as secrets | No secrets needed |
| Rotation is manual | Automatic per-run |
| Compromise = full access | Revocable via GitHub settings |
| Audit trail is limited | CloudTrail logs the identity |

**AWS Example — GitHub Actions workflow:**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write   # Required to request OIDC token
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
          aws-region: us-east-1

      - name: Deploy to S3
        run: aws s3 sync ./dist s3://my-bucket
```

**IAM Trust Policy (what you configure in AWS):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:sub": "repo:your-org/your-repo:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

**Key concepts:**
- `id-token: write` permission tells GitHub to mint a token for this workflow
- The trust policy restricts which repo/branch can assume the role
- No access keys are ever stored — only the OIDC token exchange happens
- Works with AWS, Azure, GCP, and other OIDC-compatible providers

**Interview signal:** Understands the token exchange flow, knows the security benefits of short-lived credentials, and can read/write a basic IAM trust policy. Bonus: mentions the `sub` claim filtering for repo/branch control.

---

## Summary Table

| # | Topic | Type | Level |
|---|-------|------|-------|
| 1 | OIDC in CI/CD | Practical + Conceptual | Advanced |
