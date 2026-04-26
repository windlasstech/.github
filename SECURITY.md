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

This organization follows the [SLSA (Supply-chain Levels for Software Artifacts)](https://slsa.dev/) framework to ensure software supply chain security. Detailed implementation guidance is available in the companion documents linked below.

### At a Glance

- Build L1/L2 required; Build L3 target
- Source L1/L2 required; Source L3 target
- Source L4 not achievable in a 1-person organization
- SHA-pinned workflows, hardened runners, least-privilege permissions
- Lockfiles committed, dependency review, OSV scanning
- Signed commits, protected branches, OIDC, release integrity

### Detailed Requirements

| Topic                     | Companion document                                                                         |
| :------------------------ | :----------------------------------------------------------------------------------------- |
| SLSA compliance framework | [docs/security/slsa-compliance-framework.md](./docs/security/slsa-compliance-framework.md) |
| Workflow hardening        | [docs/security/workflow-hardening.md](./docs/security/workflow-hardening.md)               |
| Dependency security       | [docs/security/dependency-security.md](./docs/security/dependency-security.md)             |

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
- OSV Scanner runs on PRs and scheduled full scans
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

Detailed reference lists for each topic are available in the companion documents:

- [SLSA Framework references](./docs/security/slsa-compliance-framework.md#references)
- [GitHub Security and Step Security references](./docs/security/workflow-hardening.md#references)
- [OSV Scanner and Additional Resources](./docs/security/dependency-security.md#references)
