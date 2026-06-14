# Workflow Hardening

> [!NOTE]
> This page is part of the Windlass organization security policy. Start with [SECURITY.md](../../SECURITY.md).

All GitHub Actions workflows enforce supply-chain protections:

## Action References

- **SHA-pinned actions** — Every `uses:` reference is pinned to the full 40-character commit SHA (not a mutable tag like `@v4`). This prevents supply-chain attacks where a tag is silently repointed to a compromised commit. Dependabot keeps SHAs up to date automatically via weekly PRs.

  Example:

  ```yaml
  # Correct - SHA pinned with version comment
  - uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2

  # Incorrect - mutable tag
  - uses: actions/checkout@v6
  ```

> [!NOTE] Exception for internal reusable workflows
> Reusable workflows published from `windlasstech/.github` should be referenced by branch name (e.g., `@main`) instead of a commit SHA. This repository does not maintain semantic version tags, and Dependabot cannot propose updates for SHA-pinned references to internal reusable workflows. Adding tags or introducing an additional automation tool solely for this purpose would create more operational overhead than value. This exception applies only to Windlass-owned reusable workflows in this repository.

## Runner Security

- **Harden-runner** — Every job starts with [`step-security/harden-runner`](https://github.com/step-security/harden-runner) in audit mode (`egress-policy: audit`), logging all outbound network calls. This provides visibility into unexpected egress from CI runners.

- **Ephemeral environments** — All builds use fresh, isolated runners with no persistence between builds

## Permission Management

When adding or modifying workflows, contributors must:

1. **Explicit top-level permissions** — Set `permissions: {}` or minimal read-only permissions at the workflow level. This establishes a secure default that prevents accidental privilege escalation.

2. **Job-level elevation** — Grant additional permissions at the job level only when required:
   - `id-token: write` — Required for OIDC token generation
   - `attestations: write` — Required for artifact attestations
   - `artifact-metadata: write` — Required to upload storage records to the organization's linked artifacts page
   - `packages: write` — Required for container registry pushes
   - `pull-requests: write` — Required for PR comments

   Example:

   ```yaml
   name: CI

   # Minimal permissions at top level
   permissions:
     contents: read

   jobs:
     build:
       runs-on: ubuntu-latest
       # Inherits top-level permissions (read-only)
       steps:
         - name: Harden the runner (Audit all outbound calls)
           uses: step-security/harden-runner@fa2e9d605c4eeb9fcad4c99c224cee0c6c7f3594 # v2.16.0
           with:
             egress-policy: audit

         - name: Checkout
           uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2

     release:
       runs-on: ubuntu-latest
       # Elevated permissions only for this job
       permissions:
         contents: read
         id-token: write # Required for OIDC
         attestations: write # Required for attestations
         artifact-metadata: write # Required for linked artifact storage records
         packages: write # Required for GHCR
       steps:
         - name: Harden the runner (Audit all outbound calls)
           uses: step-security/harden-runner@fa2e9d605c4eeb9fcad4c99c224cee0c6c7f3594 # v2.16.0
           with:
             egress-policy: audit

         - name: Checkout
           uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2
   ```

3. **Environment protection** — Production deployments must use protected environments with:
   - Required reviewers for approval
   - Deployment branch policies
   - Wait timers where appropriate

## OIDC and Cloud Authentication

Use OpenID Connect (OIDC) for authentication to cloud providers instead of long-lived credentials stored as GitHub secrets. OIDC credentials are short-lived, scoped to the workflow identity, and controlled by provider-side trust policies.

Supported providers:

- AWS — `aws-actions/configure-aws-credentials`
- Azure — `azure/login`
- Google Cloud — `google-github-actions/auth`

### AWS OIDC Example

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@e3dd6a429c905ace6919b0a7664b96b2b5dc3c81 # v4.0.2
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-deploy-role
          aws-region: us-east-1

      - name: Deploy
        run: aws s3 sync ./dist s3://my-bucket/
```

## Release Artifact Integrity

Release artifact provenance, SBOM attestations, linked artifacts uploads, and consumer verification commands are documented in [Artifact Attestations](./artifact-attestations.md).

Use this workflow hardening guide for GitHub Actions permissions, SHA-pinning, runner security, and OIDC configuration. Use the artifact attestation guide for release artifact signing and verification requirements.

## References

### GitHub Security

- [GitHub Reusable Workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows)
- [GitHub Actions Security Hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [OIDC in GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-your-deployments/configuring-openid-connect-in-cloud-providers)

### Step Security

- [Step Security - Admin Experience](https://docs.stepsecurity.io/admin-experience)
- [Step Security - Security Engineer Experience](https://docs.stepsecurity.io/security-engineer-experience)
- [Step Security - Harden-Runner](https://docs.stepsecurity.io/harden-runner)
- [Step Security - Secure Workflow](https://app.stepsecurity.io/secureworkflow)
