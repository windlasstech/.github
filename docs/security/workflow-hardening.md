# Workflow Hardening

> [!NOTE]  
> This page is part of the Windlass organization security policy. Start with
> [SECURITY.md](../../SECURITY.md).

All GitHub Actions workflows enforce supply-chain protections:

## Action References

- **SHA-pinned actions**: Every `uses:` reference is pinned to the full 40-character commit SHA (not
  a mutable tag like `@v4`). This prevents supply-chain attacks where a tag is silently repointed to
  a compromised commit. Dependabot keeps SHAs up to date automatically via weekly PRs.

  Example:

  ```yaml
  # Correct - SHA pinned with version comment
  - uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2

  # Incorrect - mutable tag
  - uses: actions/checkout@v6
  ```

> [!NOTE]  
> **Exception for internal reusable workflows**
>
> Reusable workflows published from `windlasstech/.github` should be referenced by branch name
> (e.g., `@main`) instead of a commit SHA. This repository does not maintain semantic version tags,
> and Dependabot cannot propose updates for SHA-pinned references to internal reusable workflows.
> Adding tags or introducing an additional automation tool solely for this purpose would create more
> operational overhead than value. This exception applies only to Windlass-owned reusable workflows
> in this repository.

## Third-Party Action Selection

- **StepSecurity Maintained Actions preferred**: When a third-party action is needed, check whether
  a
  [StepSecurity Maintained Action](https://docs.stepsecurity.io/github-actions/actions/stepsecurity-maintained-actions)
  replacement exists and prefer it. Maintained Actions are hardened, continuously monitored forks of
  popular community actions. The
  [StepSecurity Action Advisor](https://app.stepsecurity.io/action-advisor?tab=maintained)
  identifies which actions in a workflow have maintained replacements, and StepSecurity can open
  [automated pull requests that swap third-party actions for their maintained equivalents](https://www.stepsecurity.io/blog/replace-third-party-actions-with-stepsecurity-maintained-actions-via-automated-pull-requests).
- SHA-pinning remains mandatory for all actions, including StepSecurity Maintained Actions.

## Workflow Linting

Lint all workflow files with
[`windlasstech/actionlint-hardened-action`](https://github.com/windlasstech/actionlint-hardened-action).
This action runs [actionlint](https://github.com/rhysd/actionlint) from a SHA-pinned Docker image
and checks for common workflow mistakes, invalid expressions, missing permissions, and insecure
patterns.

Add the action to CI after checkout:

```yaml
- name: Lint workflow files with actionlint
  uses: windlasstech/actionlint-hardened-action@043a16f2538fe7bee89d8e19bbd5292e925210e0 # v1.0.0
  with:
    paths: |
      .github/workflows/*.yml
      .github/workflows/*.yaml
```

For repository-wide workflow validation, include this step in the repository lint workflow so every
workflow change is checked automatically.

## Runner Security

- **Harden Runner (required, all repositories)**: Windlass uses StepSecurity Enterprise, so every
  job in every repository, public **and** private, must start with
  [`step-security/harden-runner`](https://github.com/step-security/harden-runner). The minimum
  permitted configuration is audit mode (`egress-policy: audit`), which logs all outbound network
  calls and provides visibility into unexpected egress from CI runners. Enterprise coverage removes
  the previous limitation where Harden Runner insights were unavailable on private repositories.

- **Recommended progression to block mode**: Once audit mode has established a baseline of a
  workflow's outbound destinations, switch to `egress-policy: block` with an `allowed-endpoints`
  allow-list so all egress traffic except the approved endpoints is blocked. Audit mode is the
  floor, not the target state:

  ```yaml
  - name: Harden the runner (Block all outbound calls except allowed endpoints)
    uses: step-security/harden-runner@dc2769abcc0fcfbadfc1df54081c1608579ef6b2 # v2.22.2
    with:
      egress-policy: block
      allowed-endpoints: >
        api.github.com:443  
        github.com:443
  ```

### Harden Runner in Reusable Workflows

Harden Runner must live inside the reusable workflow itself, as the first step of each of its jobs.
Two structural reasons rule out caller-side placement:

- A caller job that invokes a reusable workflow via `jobs.<id>.uses:` cannot define `steps:`, so the
  caller cannot wrap the call with its own Harden Runner step.
- Every job executes on a fresh ephemeral runner VM allocated in the caller's context, and Harden
  Runner monitors only the runner of the job it executes in. A step in any other caller job observes
  nothing about the called workflow's job.

This repository's reusable workflows (OSV Scanner PR/Full, Dependency Review, Scorecard) therefore
ship Harden Runner built in, with `egress-policy: audit` as the non-removable floor.

- **Caller-configurable policy**: The reusable workflows expose an `egress-policy` input (string,
  default `audit`) and an `allowed-endpoints` input (string, default empty, one endpoint per line
  such as `api.github.com:443`, effective only with `egress-policy: block`) so callers can opt into
  block mode. No input can disable Harden Runner or drop below audit.

> [!WARNING]  
> Block mode fails the job when a required endpoint is missing from the allow-list. Follow the
> progression recommended above: establish a baseline in audit mode first, then switch to
> `egress-policy: block` with a complete `allowed-endpoints` list.

- **Policy Store**: On StepSecurity Enterprise, when a reusable workflow is called, the calling
  repository's policy applies, not the policy of the repository hosting the reusable workflow.
  Organization-wide block policies can therefore be managed centrally in the Policy Store without
  per-workflow input wiring.

- **Callers outside the organization**: These reusable workflows are public and may be called from
  other organizations. Harden Runner executes on the caller's runner in the caller's context, so
  external callers get audit-mode coverage without any Windlass-side configuration. Note that the
  Community tier covers public repositories only.

- **Ephemeral environments**: All builds use fresh, isolated runners with no persistence between
  builds

## Permission Management

When adding or modifying workflows, contributors must:

1. **Explicit top-level permissions**: Set `permissions: {}` or minimal read-only permissions at the
   workflow level. This establishes a secure default that prevents accidental privilege escalation.

2. **Job-level elevation**: Grant additional permissions at the job level only when required:
   - `id-token: write`: Required for OIDC token generation
   - `attestations: write`: Required for artifact attestations
   - `artifact-metadata: write`: Required to upload storage records to the organization's linked
     artifacts page
   - `packages: write`: Required for container registry pushes
   - `pull-requests: write`: Required for PR comments

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
           uses: step-security/harden-runner@dc2769abcc0fcfbadfc1df54081c1608579ef6b2 # v2.22.2
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
           uses: step-security/harden-runner@dc2769abcc0fcfbadfc1df54081c1608579ef6b2 # v2.22.2
           with:
             egress-policy: audit

         - name: Checkout
           uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2
   ```

3. **Environment protection**: Production deployments must use protected environments with:
   - Required reviewers for approval
   - Deployment branch policies
   - Wait timers where appropriate

## OIDC and Cloud Authentication

Use OpenID Connect (OIDC) for authentication to cloud providers instead of long-lived credentials
stored as GitHub secrets. OIDC credentials are short-lived, scoped to the workflow identity, and
controlled by provider-side trust policies.

Supported providers:

- AWS: `aws-actions/configure-aws-credentials`
- Azure: `azure/login`
- Google Cloud: `google-github-actions/auth`

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

Release artifact provenance, SBOM attestations, linked artifacts uploads, and consumer verification
commands are documented in [Artifact Attestations](./artifact-attestations.md).

Use this workflow hardening guide for GitHub Actions permissions, SHA-pinning, runner security, and
OIDC configuration. Use the artifact attestation guide for release artifact signing and verification
requirements.

## References

### GitHub Security

- [GitHub Reusable Workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows)
- [GitHub Actions Security Hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [OIDC in GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-your-deployments/configuring-openid-connect-in-cloud-providers)
- [Reusable Workflows: Supported keywords for jobs that call a reusable workflow](https://docs.github.com/en/actions/reference/workflows-and-actions/reusable-workflows#supported-keywords-for-jobs-that-call-a-reusable-workflow)
- [GitHub-hosted runners](https://docs.github.com/en/actions/concepts/runners/github-hosted-runners)
- [How reusable workflows use runners](https://docs.github.com/en/actions/reference/workflows-and-actions/reusable-workflows#how-reusable-workflows-use-runners)

### Step Security

- [Step Security - Admin Experience](https://docs.stepsecurity.io/admin-experience)
- [Step Security - Security Engineer Experience](https://docs.stepsecurity.io/security-engineer-experience)
- [Step Security - Harden-Runner](https://docs.stepsecurity.io/harden-runner)
- [Step Security - Secure Workflow](https://app.stepsecurity.io/secureworkflow)
- [StepSecurity Maintained Actions](https://docs.stepsecurity.io/github-actions/actions/stepsecurity-maintained-actions)
- [Replacing third-party actions with StepSecurity Maintained Actions via automated pull requests](https://www.stepsecurity.io/blog/replace-third-party-actions-with-stepsecurity-maintained-actions-via-automated-pull-requests)
- [StepSecurity Action Advisor](https://app.stepsecurity.io/action-advisor?tab=maintained)
- [Harden-Runner Policy Store](https://docs.stepsecurity.io/github-actions/harden-runner/policy-store)
- [Fix a blocked endpoint in your workflow](https://docs.stepsecurity.io/start-here/guides/how-to-fix-a-blocked-endpoint-in-your-workflow)
- [Harden Runner auto-detects cache endpoints (reusable workflows)](https://www.stepsecurity.io/blog/harden-runner-github-action-now-auto-detects-cache-endpoints)
