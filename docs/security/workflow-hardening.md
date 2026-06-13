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

## Artifact Attestations

All released artifacts that consumers run, install, deploy, or download must include cryptographically signed artifact attestations. Attestations establish provenance and integrity by linking the artifact to the repository, workflow, commit SHA, triggering event, build environment, and OIDC identity that produced it.

Do not attest every test-only build or individual source/documentation files. Attest release binaries, packages, container images, and manifests that identify released contents by digest.

### Required Permissions

Workflows generating attestations require these permissions:

```yaml
permissions:
  id-token: write # Required for OIDC token to request signing certificate
  attestations: write # Required to persist the attestation
  contents: read # Required to read source code
```

Container image release jobs also require `packages: write` when pushing to GHCR or another registry. Jobs that should appear on the organization's linked artifacts page must also include `artifact-metadata: write`.

Artifact attestations are available for public repositories on current GitHub plans. Private or internal repositories require GitHub Enterprise Cloud. GitHub Enterprise Server does not support artifact attestations.

### Build Provenance Attestations

The organization default is to produce SLSA Build L3+ provenance wherever feasible. Choose the provenance path in this order:

1. **SLSA GitHub Generator builder** — Use when the project ecosystem and release flow match a supported builder. This is the preferred path for L3+ provenance because the builder is a trusted reusable workflow that builds the artifact and generates provenance with stronger isolation.
2. **Reusable workflow with `actions/attest`** — Use when a dedicated SLSA builder is not a good fit, but the build can be moved into a trusted reusable workflow. This follows GitHub's artifact-attestation path for SLSA Build L3 by isolating the build instructions in a reusable workflow and verifying the expected signer workflow.
3. **Direct `actions/attest` in the release workflow** — Use only as the baseline provenance path when Build L3+ is not yet feasible. It still creates signed GitHub artifact attestations and satisfies the release provenance requirement, but it should not be described as the repository's L3+ implementation by itself.

Direct `actions/attest` baseline example:

```yaml
- name: Generate artifact attestation
  uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
  with:
    subject-path: "${{ github.workspace }}/my-artifact"
```

For container images, attest the image by digest and push the attestation to the registry:

```yaml
- name: Generate container attestation
  uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
  with:
    subject-name: "${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}"
    subject-digest: "${{ steps.build.outputs.digest }}"
    push-to-registry: true
```

The `subject-name` must be the fully qualified image name without a tag, and `subject-digest` must be the immutable `sha256:...` digest from the image build/push step. `push-to-registry: true` is required when the attestation should upload registry storage metadata to the linked artifacts page, and `actions/attest` supports that option only with `subject-name` plus `subject-digest`.

For reusable workflow based L3 provenance, generate the artifact and the `actions/attest` provenance inside the reusable workflow. Both the caller and reusable workflow must grant `contents: read`, `id-token: write`, and `attestations: write`; container builds also need `packages: write`. Verification policy must use `--signer-workflow`, and `--signer-repo` when the reusable workflow lives in another repository.

### SBOM Attestations

Released binaries and container images must also have SBOM attestations when the build can produce them. Prefer generating both SPDX and CycloneDX SBOMs because downstream consumers and scanners do not all accept the same format. Generate each SBOM during the release workflow, then run one `actions/attest` step per SBOM file with `sbom-path`.

`actions/attest` accepts a single JSON-formatted SPDX or CycloneDX SBOM file per attestation. Each SBOM file must be 16 MB or smaller and cannot be combined with custom `predicate-type`, `predicate`, or `predicate-path` inputs in the same step.

This policy requires released artifacts to include provenance and SBOM attestations. `anchore/sbom-action` is only the example SBOM generator used in this document. Projects may use another SBOM generation tool if it produces valid SPDX or CycloneDX JSON for `actions/attest` or an equivalent attestation workflow.

When a project uses a different SBOM generator, configure workflow permissions and tool-specific settings according to that tool's requirements. The permission examples below apply to `anchore/sbom-action`; other tools may need different read, write, registry, or release permissions.

For public releases, generate local SBOM files for attestation and publish the same SBOM files as release assets whenever possible. Attestation provides cryptographic binding to the released artifact. Release assets make the SBOMs easy to download for auditors, license review, offline storage, and third-party scanners. Release asset uploads do not create linked artifacts storage records by themselves.

#### SBOM Permissions by Purpose

Use the narrowest permissions that match the release job's behavior:

| Purpose                                    | Required permissions                                                          | Notes                                                                                                  |
| :----------------------------------------- | :---------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------- |
| Local SBOM generation for attestation only | `contents: read`, plus `id-token: write` and `attestations: write` for attest | Set `upload-artifact: false` and `upload-release-assets: false` on `anchore/sbom-action`.              |
| Workflow artifact upload                   | `contents: write`                                                             | Required by `anchore/sbom-action` when it uploads generated SBOMs as workflow artifacts.               |
| Release asset upload                       | `actions: read`, `contents: write`                                            | Needed so Anchore can read workflow artifacts and write release assets. Grant only on release jobs.    |
| Release asset upload plus attestation      | `actions: read`, `contents: write`, `id-token: write`, `attestations: write`  | Add `packages: write` for container registry pushes and `artifact-metadata: write` for linked records. |

`anchore/sbom-action` defaults to `upload-artifact: true` and `upload-release-assets: true`. If the SBOM is only an intermediate file for `actions/attest`, disable both upload behaviors explicitly. If the workflow is creating a GitHub Release, leave release publishing enabled and grant `actions: read` plus `contents: write` at the job level.

Binary example:

```yaml
- name: Generate SPDX SBOM
  uses: anchore/sbom-action@f8bdd1d8ac5e901a77a92f111440fdb1b593736b # v0.20.6
  with:
    path: ./dist/my-artifact
    format: spdx-json
    output-file: sbom.spdx.json
    upload-artifact: false
    upload-release-assets: false

- name: Generate CycloneDX SBOM
  uses: anchore/sbom-action@f8bdd1d8ac5e901a77a92f111440fdb1b593736b # v0.20.6
  with:
    path: ./dist/my-artifact
    format: cyclonedx-json
    output-file: sbom.cyclonedx.json
    upload-artifact: false
    upload-release-assets: false

- name: Generate SPDX SBOM attestation
  uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
  with:
    subject-path: ./dist/my-artifact
    sbom-path: sbom.spdx.json

- name: Generate CycloneDX SBOM attestation
  uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
  with:
    subject-path: ./dist/my-artifact
    sbom-path: sbom.cyclonedx.json
```

Release asset publishing example:

```yaml
permissions:
  actions: read # Required to read workflow artifacts for release publishing
  contents: write # Required to write SBOM files to the GitHub Release
  id-token: write # Required for attestation signing
  attestations: write # Required to persist SBOM attestations
  artifact-metadata: write # Required for linked artifact storage records

steps:
  - name: Generate and publish SPDX SBOM
    uses: anchore/sbom-action@f8bdd1d8ac5e901a77a92f111440fdb1b593736b # v0.20.6
    with:
      path: ./dist/my-artifact
      format: spdx-json
      output-file: sbom.spdx.json
      artifact-name: sbom.spdx.json
      upload-artifact: true
      upload-release-assets: true

  - name: Generate and publish CycloneDX SBOM
    uses: anchore/sbom-action@f8bdd1d8ac5e901a77a92f111440fdb1b593736b # v0.20.6
    with:
      path: ./dist/my-artifact
      format: cyclonedx-json
      output-file: sbom.cyclonedx.json
      artifact-name: sbom.cyclonedx.json
      upload-artifact: true
      upload-release-assets: true

  - name: Generate SPDX SBOM attestation
    uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
    with:
      subject-path: ./dist/my-artifact
      sbom-path: sbom.spdx.json

  - name: Generate CycloneDX SBOM attestation
    uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
    with:
      subject-path: ./dist/my-artifact
      sbom-path: sbom.cyclonedx.json
```

Container image example:

```yaml
- name: Generate SPDX SBOM attestation
  uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
  with:
    subject-name: "${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}"
    subject-digest: "${{ steps.build.outputs.digest }}"
    sbom-path: sbom.spdx.json
    push-to-registry: true

- name: Generate CycloneDX SBOM attestation
  uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
  with:
    subject-name: "${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}"
    subject-digest: "${{ steps.build.outputs.digest }}"
    sbom-path: sbom.cyclonedx.json
    push-to-registry: true
```

SBOM attestations do not replace vulnerability scanning. They provide signed dependency metadata that consumers and auditors can verify alongside build provenance.

### Linked Artifacts Page Uploads

For container images and other registry-published release artifacts, workflows should upload storage metadata to the organization's linked artifacts page so security and compliance reviewers can trace artifacts to their source repository, build run, storage location, attestations, and deployment context.

The `actions/attest` action automatically creates a linked artifacts storage record when both conditions are true:

- `push-to-registry: true` is set on the attestation step
- The job has `artifact-metadata: write`

For `actions/attest`, `push-to-registry: true` requires `subject-name` and `subject-digest`. Use it for container images and other registry artifacts that have a fully qualified registry name and immutable digest. Do not add `push-to-registry: true` to file-path attestations that use `subject-path`; those attestations are still valid, but they do not create linked artifacts storage records through `actions/attest`.

Storage record creation is enabled by default when `push-to-registry: true` is set. Set `create-storage-record: false` only when the artifact must not appear on the linked artifacts page. Storage records can be created only for artifacts built from organization-owned repositories.

For non-registry release assets that still need linked artifacts metadata, use the artifact metadata REST API or an approved integration instead of forcing `push-to-registry` into a file-path attestation.

Recommended container release pattern:

```yaml
permissions:
  contents: read
  id-token: write
  attestations: write
  packages: write
  artifact-metadata: write

steps:
  - name: Build and push image
    id: build
    uses: docker/build-push-action@263435318d21b8e681c14492fe198d362a7d2c83 # v6.18.0
    with:
      context: .
      push: true
      tags: |
        ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
        ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

  - name: Generate artifact attestation
    uses: actions/attest@61d634515b50b54366a3498d04742794e07fc381 # v4.1.0
    with:
      subject-name: "${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}"
      subject-digest: "${{ steps.build.outputs.digest }}"
      push-to-registry: true
```

If an artifact is not attested, or if deployment/runtime records must be uploaded, use the artifact metadata REST API or an approved integration such as JFrog Artifactory, Dynatrace, or Microsoft Defender for Cloud. Linked artifacts records store metadata only; they do not store artifact files.

### Attestation Modes

| Mode                     | Input                          | Description                                  |
| :----------------------- | :----------------------------- | :------------------------------------------- |
| **Provenance** (default) | `subject-path` only            | Auto-generates SLSA build provenance         |
| **SBOM**                 | `sbom-path` provided           | Creates attestation from SPDX/CycloneDX SBOM |
| **Custom**               | `predicate-type` + `predicate` | User-defined predicate                       |

### Verification Expectations

Consumers verify build provenance with `gh attestation verify`. SBOM attestations must be verified with the SBOM predicate type, such as `https://spdx.dev/Document/v2.3` for SPDX SBOMs.

```bash
# Verify SPDX SBOM attestation
gh attestation verify ./my-artifact \
  -R windlasstech/my-repo \
  --predicate-type https://spdx.dev/Document/v2.3

# Verify CycloneDX SBOM attestation
gh attestation verify ./my-artifact \
  -R windlasstech/my-repo \
  --predicate-type https://cyclonedx.org/bom
```

For reusable workflow builders, verification policy should pin the expected signing workflow with `--signer-workflow`, and use `--signer-repo` when the reusable workflow lives in a separate repository.

## SLSA GitHub Generator

For language-specific builds, prefer the [slsa-framework/slsa-github-generator](https://github.com/slsa-framework/slsa-github-generator) when a supported builder fits the project. These builders are the organization's default path for SLSA Build L3+ provenance because they build the artifact and generate provenance inside a trusted reusable workflow.

### Available Builders

| Ecosystem   | Builder                                                                              | Status |
| :---------- | :----------------------------------------------------------------------------------- | :----- |
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
      go-version: "1.26"
      # ... other inputs
```

## References

### GitHub Security

- [GitHub Artifact Attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations)
- [Using artifact attestations](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/use-artifact-attestations)
- [Using artifact attestations and reusable workflows to achieve SLSA v1 Build Level 3](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/increase-security-rating)
- [GitHub Linked Artifacts](https://docs.github.com/en/code-security/concepts/supply-chain-security/linked-artifacts)
- [Uploading storage and deployment data to the linked artifacts page](https://docs.github.com/en/code-security/how-tos/secure-your-supply-chain/establish-provenance-and-integrity/upload-linked-artifacts)
- [GitHub Reusable Workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows)
- [actions/attest](https://github.com/actions/attest)
- [GitHub Actions Security Hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [OIDC in GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-your-deployments/configuring-openid-connect-in-cloud-providers)

### Step Security

- [Step Security - Admin Experience](https://docs.stepsecurity.io/admin-experience)
- [Step Security - Security Engineer Experience](https://docs.stepsecurity.io/security-engineer-experience)
- [Step Security - Harden-Runner](https://docs.stepsecurity.io/harden-runner)
- [Step Security - Secure Workflow](https://app.stepsecurity.io/secureworkflow)
