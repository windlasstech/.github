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

## Artifact Attestations

All released artifacts must include cryptographically signed attestations establishing build provenance.

### Required Permissions

Workflows generating attestations require these permissions:

```yaml
permissions:
  id-token: write # Required for OIDC token to request signing certificate
  attestations: write # Required to persist the attestation
  contents: read # Required to read source code
```

### Generating Attestations

Use the `actions/attest` action to generate SLSA-compliant provenance:

```yaml
- name: Generate artifact attestation
  uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
  with:
    subject-path: '${{ github.workspace }}/my-artifact'
```

For container images:

```yaml
- name: Generate container attestation
  uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
  with:
    subject-name: '${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}'
    subject-digest: '${{ steps.build.outputs.digest }}'
    push-to-registry: true
```

### Attestation Modes

| Mode                     | Input                          | Description                                  |
| ------------------------ | ------------------------------ | -------------------------------------------- |
| **Provenance** (default) | `subject-path` only            | Auto-generates SLSA build provenance         |
| **SBOM**                 | `sbom-path` provided           | Creates attestation from SPDX/CycloneDX SBOM |
| **Custom**               | `predicate-type` + `predicate` | User-defined predicate                       |

## SLSA GitHub Generator

For language-specific builds, use the [slsa-framework/slsa-github-generator](https://github.com/slsa-framework/slsa-github-generator) to generate SLSA Build Level 3 provenance:

### Available Builders

| Ecosystem   | Builder                                                                              | Status |
| ----------- | ------------------------------------------------------------------------------------ | ------ |
| Go          | `slsa-framework/slsa-github-generator/.github/workflows/builder_go_slsa3.yml`        | Stable |
| Node.js/npm | `slsa-framework/slsa-github-generator/.github/workflows/builder_nodejs_slsa3.yml`    | Beta   |
| Maven       | `slsa-framework/slsa-github-generator/.github/workflows/builder_maven_slsa3.yml`     | Beta   |
| Gradle      | `slsa-framework/slsa-github-generator/.github/workflows/builder_gradle_slsa3.yml`    | Beta   |
| Generic     | `slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml` | Stable |

### Reference Requirements

> [!IMPORTANT]
> SLSA generators must be referenced by tag (e.g., `@v2.1.0`) for `slsa-verifier` to validate the trusted builder. This is an intentional exception to the SHA-pinning requirement.

```yaml
jobs:
  build:
    uses: slsa-framework/slsa-github-generator/.github/workflows/builder_go_slsa3.yml@v2.1.0
    with:
      go-version: '1.26'
      # ... other inputs
```

## References

### GitHub Security

- [GitHub Artifact Attestations](https://docs.github.com/en/actions/security-guides/using-artifact-attestations-to-establish-provenance-for-builds)
- [GitHub Artifact Attestations - Increase Security](https://docs.github.com/en/actions/security-guides/using-artifact-attestations/increase-security-for-your-builds)
- [GitHub Reusable Workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows)
- [actions/attest](https://github.com/actions/attest)
- [GitHub Actions Security Hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [OIDC in GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-your-deployments/configuring-openid-connect-in-cloud-providers)

### Step Security

- [Step Security - Admin Experience](https://docs.stepsecurity.io/admin-experience)
- [Step Security - Security Engineer Experience](https://docs.stepsecurity.io/security-engineer-experience)
- [Step Security - Harden-Runner](https://docs.stepsecurity.io/harden-runner)
- [Step Security - Secure Workflow](https://app.stepsecurity.io/secureworkflow)
