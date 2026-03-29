# Security Policy

Thank you for helping keep this project and its users safe. We take security seriously. This document covers how to report vulnerabilities, what is in scope, and how the project's supply chain integrity works.

## Scope

This policy applies to repositories under this organization.

- Public repositories: client, protocol, and conformance assets
- Private repositories: backend, infrastructure, and internal operations

Do not disclose sensitive details publicly until we complete coordinated remediation.

## Out of Scope

The following are normally out of scope:

- Cosmetic/content issues without security impact
- Vulnerabilities in upstream repositories that we do not distribute yet (report these to the upstream project directly)
- DoS volume testing without prior approval
- Reports based only on missing best-practice headers without exploitability

## Reporting a Vulnerability

Please report vulnerabilities privately:

- Preferred: GitHub Security Advisories — Use the `Security > Advisories` tab to submit a private report.
- Alternate: Send a report to [oss-security@windlasstech.com](mailto:oss-security@windlasstech.com).

### What to Include

1. Affected repository and commit/tag/version
2. Description of the vulnerability and its potential impact
3. Reproduction steps or proof of concept
4. Suggested mitigation (if available)

### Response Timeline

| Step                              | Timeline                                      |
| --------------------------------- | --------------------------------------------- |
| Acknowledgment of report          | 3 business days                               |
| Triage and severity assessment    | 7 business days                               |
| Fix for critical or high severity | 14 days                                       |
| Fix for medium or low severity    | 30 days                                       |
| Public disclosure                 | After fix is released, or 90 days from report |

These timelines are best-effort commitments and are not guaranteed SLAs. Complex issues may take longer, but we will keep you informed of progress — at least every 7 business days until closure.

## Disclosure Policy

We follow coordinated disclosure:

1. The reporter submits a private vulnerability report.
2. We acknowledge, triage, and work on a fix privately.
3. Once a fix is released, we publish a GitHub Security Advisory with full details.
4. The reporter is credited in the advisory unless they request anonymity.
5. If no fix is released within 90 days, the reporter may disclose publicly.

We will not take legal action against researchers who report vulnerabilities in good faith and follow this policy.

## Safe Harbor

We support good-faith security research.

Please do not:

- Access non-public data beyond what is required to prove impact
- Degrade service availability
- Social engineer, spam, or perform destructive testing

## Supply Chain Integrity

### SLSA Compliance Framework

This organization follows the [SLSA (Supply-chain Levels for Software Artifacts)](https://slsa.dev/) framework to ensure software supply chain security. SLSA provides a checklist of standards and controls to prevent tampering, improve integrity, and secure software packages.

#### Current SLSA Level Status

| Level             | Status      | Description                                           |
| ----------------- | ----------- | ----------------------------------------------------- |
| **SLSA Build L1** | ✅ Required | Provenance exists documenting how artifacts are built |
| **SLSA Build L2** | ✅ Required | Hosted build platform with signed provenance          |
| **SLSA Build L3** | 🎯 Target   | Hardened builds with isolated, ephemeral environments |
| **SLSA Build L4** | 📋 Planned  | Hermetic and reproducible builds (future goal)        |

> [!NOTE]
> SLSA v1.2 defines Build Track levels 0-3. Level 4 and additional tracks (Source, Common) are planned for future specification versions.

#### SLSA Build Level Requirements

##### Build L1 — Provenance Exists

- All builds must generate provenance describing:
  - Entity that performed the build
  - Build process and steps used
  - Top-level inputs (source repository, commit SHA, dependencies)
- Provenance must be in SLSA format or equivalent

##### Build L2 — Hosted Build Platform

- All production builds must run on hosted CI/CD platforms (GitHub Actions)
- Provenance must be cryptographically signed by the build platform
- No local/developer workstation builds for releases

##### Build L3 — Hardened Builds

- Builds must run in isolated, ephemeral environments
- Build steps cannot access platform signing keys
- Concurrent builds cannot influence each other
- Cache poisoning must be prevented between builds
- All external interactions must be captured in provenance

### Workflow Hardening

All GitHub Actions workflows enforce supply-chain protections:

#### Action References

- **SHA-pinned actions** — Every `uses:` reference is pinned to the full 40-character commit SHA (not a mutable tag like `@v4`). This prevents supply-chain attacks where a tag is silently repointed to a compromised commit. Dependabot keeps SHAs up to date automatically via weekly PRs.

  Example:

  ```yaml
  # Correct - SHA pinned with version comment
  - uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2

  # Incorrect - mutable tag
  - uses: actions/checkout@v6
  ```

#### Runner Security

- **Harden-runner** — Every job starts with [`step-security/harden-runner`](https://github.com/step-security/harden-runner) in audit mode (`egress-policy: audit`), logging all outbound network calls. This provides visibility into unexpected egress from CI runners.

- **Ephemeral environments** — All builds use fresh, isolated runners with no persistence between builds

#### Permission Management

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

### Artifact Attestations

All released artifacts must include cryptographically signed attestations establishing build provenance.

#### Required Permissions

Workflows generating attestations require these permissions:

```yaml
permissions:
  id-token: write # Required for OIDC token to request signing certificate
  attestations: write # Required to persist the attestation
  contents: read # Required to read source code
```

#### Generating Attestations

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

#### Attestation Modes

| Mode                     | Input                          | Description                                  |
| ------------------------ | ------------------------------ | -------------------------------------------- |
| **Provenance** (default) | `subject-path` only            | Auto-generates SLSA build provenance         |
| **SBOM**                 | `sbom-path` provided           | Creates attestation from SPDX/CycloneDX SBOM |
| **Custom**               | `predicate-type` + `predicate` | User-defined predicate                       |

### SLSA GitHub Generator

For language-specific builds, use the [slsa-framework/slsa-github-generator](https://github.com/slsa-framework/slsa-github-generator) to generate SLSA Build Level 3 provenance:

#### Available Builders

| Ecosystem   | Builder                                                                              | Status |
| ----------- | ------------------------------------------------------------------------------------ | ------ |
| Go          | `slsa-framework/slsa-github-generator/.github/workflows/builder_go_slsa3.yml`        | Stable |
| Node.js/npm | `slsa-framework/slsa-github-generator/.github/workflows/builder_nodejs_slsa3.yml`    | Beta   |
| Maven       | `slsa-framework/slsa-github-generator/.github/workflows/builder_maven_slsa3.yml`     | Beta   |
| Gradle      | `slsa-framework/slsa-github-generator/.github/workflows/builder_gradle_slsa3.yml`    | Beta   |
| Generic     | `slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml` | Stable |

#### Reference Requirements

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

### Dependency Locking

All projects must commit dependency lockfiles to version control to ensure reproducible builds and enable security auditing.

#### Lockfile Requirements

| Ecosystem    | Lockfile                                     | Commit Required |
| ------------ | -------------------------------------------- | --------------- |
| Rust/Cargo   | `Cargo.lock`                                 | ✅ Yes          |
| Node.js/npm  | `package-lock.json`                          | ✅ Yes          |
| Node.js/Yarn | `yarn.lock`                                  | ✅ Yes          |
| Node.js/pnpm | `pnpm-lock.yaml`                             | ✅ Yes          |
| Python/pip   | `requirements.txt` (pinned) or `poetry.lock` | ✅ Yes          |
| Go           | `go.sum`                                     | ✅ Yes          |
| Java/Maven   | `pom.xml` (with versions)                    | ✅ Yes          |
| Java/Gradle  | `gradle.lockfile`                            | ✅ Yes          |
| Ruby         | `Gemfile.lock`                               | ✅ Yes          |

#### Dependency Update Automation

- Use Dependabot or Renovate to automatically update dependencies
- Security updates should be applied immediately
- Major version updates require review and testing

#### Dependabot Cooldown Configuration

Cooldown periods delay version update PRs to allow community vetting of new releases. This mitigates supply chain attacks by providing time for malicious packages to be detected and yanked before adoption.

> [!NOTE]
> Security updates bypass cooldowns and are created immediately. Cooldowns apply only to version updates.

##### Ecosystem-Specific Recommendations

| Ecosystem                 | Patch     | Minor      | Major      | Rationale                                                                             |
| :------------------------ | :-------- | :--------- | :--------- | :------------------------------------------------------------------------------------ |
| **GitHub Actions**        | N/A       | N/A        | N/A        | Use `default-days` only; SemVer cooldown not supported. SHA-pinning mitigates risk.   |
| **NPM/Bun (Production)**  | 3-7 days  | 7-14 days  | 21-30 days | Balance security with npm `min-release-age`; align cooldown with install restrictions |
| **NPM/Bun (Development)** | 1-3 days  | 3-7 days   | 14-21 days | Lower risk tolerance for dev dependencies                                             |
| **Rust/Cargo**            | 3-7 days  | 14-21 days | 30-60 days | Stricter SemVer compliance allows longer vetting; Cargo.lock provides safety net      |
| **Python/pip**            | 7-14 days | 14-21 days | 21-30 days | PyPI has slower yank response; conservative approach recommended                      |
| **Go**                    | 3-7 days  | 7-14 days  | 14-21 days | Go modules are immutable; supply chain attacks less common but possible               |

> [!IMPORTANT]
> SemVer-based cooldown (`semver-major-days`, `semver-minor-days`, `semver-patch-days`) is only supported by package ecosystems that use Semantic Versioning. Ecosystems like **GitHub Actions**, **Docker**, **Terraform**, and **Bazel** only support `default-days`.

##### SemVer Cooldown Support by Ecosystem

| Support                   | Ecosystems                                                                                                                                                      |
| :------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SemVer + default-days** | `bundler`, `bun`, `cargo`, `composer`, `dotnet_sdk`, `elm`, `gomod`, `gradle`, `hex`, `julia`, `maven`, `npm`, `nuget`, `opentofu`, `pip`, `pub`, `swift`, `uv` |
| **default-days only**     | `bazel`, `devcontainers`, `docker`, `docker_compose`, `github-actions`, `gitsubmodule`, `helm`, `terraform`                                                     |

##### Cooldown Rationale

**Why cooldowns matter:**

- Most malicious packages are detected within 24-72 hours of publication
- 7-day cooldown provides comfortable safety margin for community detection
- Organizations with cooldowns were not exposed during major incidents (e.g., Nx, xz utils backdoor)

**SemVer-specific delays:**

- **Patch (z in x.y.z)**: Bug fixes only; shortest cooldown acceptable
- **Minor (y in x.y.z)**: New features; moderate cooldown for API stability verification
- **Major (x in x.y.z)**: Breaking changes; longest cooldown for migration planning and community feedback

##### Rust-Specific Considerations

The Rust ecosystem has unique characteristics affecting cooldown strategy:

1. **Strict SemVer Compliance**: Cargo enforces SemVer strictly. Minor updates are generally safer than in npm, but the community tends toward conservatism.

2. **Cargo Yank Mechanism**: Yanked crates remain available for existing `Cargo.lock` but prevent new adoption. Cooldowns work synergistically—malicious versions can be yanked before PRs are created.

3. **Recent Attack Patterns** (March 2026): Five malicious crates (`chrono_anchor`, `dnp3times`, etc.) targeted CI/CD environments to exfiltrate `.env` files. Cooldowns provide critical detection window.

4. **Cargo.lock Pinning**: Unlike npm, Cargo.lock should always be committed. This creates natural review gates that complement cooldowns.

##### Example Configuration

```yaml
# dependabot.yml - Production-grade Rust project
version: 2
updates:
  - package-ecosystem: 'cargo'
    directory: '/'
    schedule:
      interval: 'weekly'
      day: 'tuesday'
    cooldown:
      default-days: 7
      semver-major-days: 30
      semver-minor-days: 14
      semver-patch-days: 5
    groups:
      patch-updates:
        patterns: ['*']
        update-types: ['patch']
    reviewers:
      - 'windlass-tech/security-team'
```

##### Additional Supply Chain Defenses

Combine cooldowns with:

- **`cargo-audit`**: Scan for known vulnerabilities in `Cargo.lock`
- **`cargo-vet`**: Establish trust relationships for dependencies
- **Dependency review workflows**: Require human review for new dependencies
- **Lockfile verification**: Always review `Cargo.lock`/`package-lock.json` diffs in PRs

### Dependency Review

The [Dependency Review Action](https://github.com/actions/dependency-review-action) scans pull requests for vulnerable or non-compliant dependencies before they enter the codebase. It performs diff-based analysis comparing dependencies in the PR head against the base, identifying changes that introduce known vulnerabilities or license violations.

> [!NOTE]
> Dependency Review is available for all public repositories. For private repositories, GitHub Advanced Security is required.

#### Key Differences from Dependabot

| Feature         | Dependabot                         | Dependency Review                     |
| :-------------- | :--------------------------------- | :------------------------------------ |
| **Timing**      | Alerts on existing vulnerabilities | Blocks new vulnerabilities at PR time |
| **Scope**       | Scans entire dependency tree       | Scans only changed dependencies       |
| **Action**      | Creates update PRs                 | Fails PR checks to prevent merge      |
| **Integration** | Standalone alerts                  | GitHub Actions workflow               |

#### Required Workflow Template

All repositories must implement Dependency Review with this hardened configuration:

```yaml
name: 'Dependency Review'
on:
  pull_request:
    paths-ignore:
      - '**.md'
      - 'docs/**'
  merge_group:
    paths-ignore:
      - '**.md'
      - 'docs/**'

permissions:
  contents: read

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
      - name: Harden the runner (Audit all outbound calls)
        uses: step-security/harden-runner@fa2e9d605c4eeb9fcad4c99c224cee0c6c7f3594 # v2.16.0
        with:
          egress-policy: audit

      - name: 'Checkout Repository'
        uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2
        with:
          persist-credentials: false

      - name: 'Dependency Review'
        uses: actions/dependency-review-action@2031cfc080254a8a887f58cffee85186f0e49e48 # v4.9.0
        with:
          fail-on-severity: critical
          fail-on-scopes: development, runtime
          comment-summary-in-pr: on-failure
```

#### Configuration Options

| Option                  | Type    | Description                                                         | Recommended                              |
| :---------------------- | :------ | :------------------------------------------------------------------ | :--------------------------------------- |
| `fail-on-severity`      | string  | Minimum severity to block PR: `low`, `moderate`, `high`, `critical` | `critical`                               |
| `fail-on-scopes`        | string  | Scopes to check: `runtime`, `development`, `unknown`                | `development, runtime`                   |
| `allow-licenses`        | string  | Comma-separated allowed SPDX license IDs                            | See license policy below                 |
| `deny-licenses`         | string  | Comma-separated forbidden SPDX license IDs                          | Mutually exclusive with `allow-licenses` |
| `allow-ghsas`           | string  | GitHub Advisory IDs to temporarily allow                            | Document with justification              |
| `comment-summary-in-pr` | string  | PR comment mode: `always`, `on-failure`, `never`                    | `on-failure`                             |
| `warn-only`             | boolean | Log warnings without failing                                        | Use for phased rollout only              |

#### Organization-Wide License Policy

Use an allow-list approach for license compliance. Reference a centralized configuration file:

```yaml
# In repository workflow
- name: 'Dependency Review'
  uses: actions/dependency-review-action@2031cfc080254a8a887f58cffee85186f0e49e48 # v4.9.0
  with:
    config-file: 'windlass/.github/dependency-review-config.yml@main'
    external-repo-token: ${{ secrets.DEPENDENCY_REVIEW_TOKEN }}
```

**Standard allow-licenses list:**

```yaml
allow-licenses:
  # Permissive licenses with attribution requirements
  - MIT # MIT License - permissive, requires attribution
  - Apache-2.0 # Apache License 2.0 - permissive, patent protection
  - BSD-2-Clause # BSD 2-Clause "Simplified" - permissive
  - BSD-3-Clause # BSD 3-Clause "New" - permissive, no endorsement clause
  - ISC # ISC License - permissive, functionally equivalent to MIT
  - 0BSD # Zero-Clause BSD - permissive, no attribution required
  # Public domain dedications (no attribution required)
  - CC0-1.0 # Creative Commons Zero - public domain dedication
  - Unlicense # The Unlicense - public domain dedication (not "unlicensed")
  # Language-specific permissive licenses
  - Python-2.0 # Python Software Foundation License - permissive
```

> [!NOTE]
> License identifiers must be SPDX-compliant. For licenses detected as `OTHER`, use `LicenseRef-clearlydefined-OTHER` in allow/deny lists.

#### Permissions Requirements

| Use Case                 | Required Permissions                     |
| :----------------------- | :--------------------------------------- |
| Basic vulnerability scan | `contents: read`                         |
| PR comments enabled      | `contents: read`, `pull-requests: write` |
| External config file     | `contents: read` + `external-repo-token` |

#### Severity Levels

| Level        | Description                          | Recommended Action |
| :----------- | :----------------------------------- | :----------------- |
| **Critical** | Active exploitation or severe impact | Always fail        |
| **High**     | Significant security impact          | Fail in production |
| **Moderate** | Limited impact                       | Consider failing   |
| **Low**      | Minimal security risk                | Warn only          |

#### Repository Ruleset Integration

Enforce Dependency Review across the organization using repository rulesets:

1. Navigate to **Organization Settings > Rules > Rulesets**
2. Create ruleset targeting all repositories
3. Add requirement: **Require status checks to pass**
4. Search for and select the `dependency-review` workflow

This ensures all PRs must pass dependency review before merging, regardless of individual repository configuration.

#### Common Pitfalls

| Pitfall                                                     | Solution                                              |
| :---------------------------------------------------------- | :---------------------------------------------------- |
| Missing `pull-requests: write` with `comment-summary-in-pr` | Add permission to workflow job                        |
| Internal packages fail license check                        | Add to `allow-dependencies-licenses` with purl format |
| Both `allow-licenses` and `deny-licenses` set               | Choose one approach; they are mutually exclusive      |
| External config in private repo fails                       | Provide `external-repo-token` with read access        |
| Action fails before dependency submission completes         | Use `retry-on-snapshot-warnings: true`                |

#### References

- [Dependency Review Action Repository](https://github.com/actions/dependency-review-action)
- [GitHub Dependency Review Documentation](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review)
- [SPDX License List](https://spdx.org/licenses/)
- [GitHub Advisory Database](https://github.com/advisories)

### Source Integrity

#### Commit Signing

All commits to protected branches must be cryptographically signed:

- **GPG signing** — Traditional approach using GPG keys
- **SSH signing** — Alternative to GPG using SSH keys
- **Sigstore gitsign** — Keyless signing linking to OIDC identity (recommended)

#### Branch Protection

All repositories must enforce:

- Require signed commits
- Require pull request reviews (minimum 1 reviewer)
- Require status checks to pass before merging
- Require linear history (no merge commits)
- Include administrators in restrictions
- Restrict force pushes

### OIDC and Cloud Authentication

Use OpenID Connect (OIDC) for authentication to cloud providers instead of long-lived credentials stored as secrets.

#### Supported Providers

- AWS — `aws-actions/configure-aws-credentials`
- Azure — `azure/login`
- Google Cloud — `google-github-actions/auth`

#### AWS OIDC Example

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@e3dd6a429c905ace6919b0a7664b96b2b5dc3c81 # v4.0.2
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-deploy-role
          aws-region: us-east-1

      - name: Deploy
        run: aws s3 sync ./dist s3://my-bucket/
```

### Release Integrity

- **GPG-signed tags** — All release tags are GPG-signed annotated tags
- **GPG-signed commits** — All commits on `main` must be GPG-signed (enforced by repository rulesets)
- **Linear history** — Squash-merge-only policy prevents history rewriting; force-push to `main` is blocked
- **CI gate** — All required status checks must pass before any merge to `main`
- **Artifact attestations** — All released artifacts must include signed attestations

## Verifying Artifacts

Consumers can verify the authenticity and provenance of artifacts from this organization using the GitHub CLI:

### Install GitHub CLI

```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Or download from https://cli.github.com/
```

### Verify Binary Attestations

```bash
# Verify a downloaded binary
gh attestation verify ./my-artifact -R windlass-tech/my-repo

# Verify with specific workflow
gh attestation verify ./my-artifact \
  -R windlass-tech/my-repo \
  --signer-workflow windlass-tech/my-repo/.github/workflows/release.yml
```

### Verify Container Image Attestations

```bash
# Verify a container image
gh attestation verify oci://ghcr.io/windlass-tech/my-image:latest \
  -R windlass-tech/my-repo

# Verify by digest (recommended)
gh attestation verify oci://ghcr.io/windlass-tech/my-image@sha256:abc123... \
  -R windlass-tech/my-repo
```

### Verify SLSA Provenance

For artifacts with SLSA provenance generated by slsa-github-generator:

```bash
# Install slsa-verifier
go install github.com/slsa-framework/slsa-verifier/cli/slsa-verifier@v2.0.0

# Verify provenance
slsa-verifier verify-artifact my-artifact \
  --provenance-path my-artifact.intoto.jsonl \
  --source-uri github.com/windlass-tech/my-repo \
  --source-tag v1.0.0
```

### What Verification Checks

Verification confirms:

1. **Authenticity** — The artifact was built by the claimed repository
2. **Integrity** — The artifact has not been tampered with since build
3. **Provenance** — The artifact's build process is documented
4. **Source** — The exact source code (commit SHA) used to build
5. **Build environment** — The workflow that produced the artifact

## Security Monitoring

### Audit Logging

- All workflow runs are logged via GitHub's audit log
- Access to secrets is logged
- Deployment activity is tracked

### Vulnerability Scanning

- Dependabot alerts are enabled on all repositories
- Code scanning with CodeQL runs on all PRs
- Secret scanning prevents credential leakage

### Security Scorecard

This organization uses [OpenSSF Scorecard](https://securityscorecards.dev/) to continuously monitor security posture:

- Binary artifacts
- Branch protection
- Code review
- Dependency update tool
- Fuzzing
- License
- Maintained
- Pinned dependencies
- SAST
- Security policy
- Signed releases
- Token permissions
- Vulnerabilities

## References

### SLSA Framework

- [SLSA Specification v1.2](https://slsa.dev/spec/v1.2/)
- [SLSA Get Started Guide](https://slsa.dev/how-to/get-started)
- [SLSA for Organizations](https://slsa.dev/how-to/how-to-orgs)
- [SLSA for Infrastructure Providers](https://slsa.dev/how-to/how-to-infra)
- [SLSA GitHub Generator](https://github.com/slsa-framework/slsa-github-generator)

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

### Additional Resources

- [Sigstore](https://www.sigstore.dev/)
- [in-toto attestation format](https://github.com/in-toto/attestation)
- [Rust Lockfile Guidelines](https://blog.rust-lang.org/2023/08/29/committing-lockfiles/)
