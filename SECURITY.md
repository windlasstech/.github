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

## Supply Chain Integrity

### Workflow Hardening

All GitHub Actions workflows enforce two supply-chain protections:

- **SHA-pinned actions** — Every `uses:` reference is pinned to the full 40-character commit SHA (not a mutable tag like `@v4`). This prevents supply-chain attacks where a tag is silently repointed to a compromised commit. Dependabot keeps SHAs up to date automatically via weekly PRs.
- **Harden-runner** — Every job starts with [`step-security/harden-runner`](https://github.com/step-security/harden-runner) in audit mode (`egress-policy: audit`), logging all outbound network calls. This provides visibility into unexpected egress from CI runners.

When adding or modifying workflows, contributors must:

1. Pin all new action references to commit SHAs (add a `# vX.Y.Z` comment for readability).
2. Include the harden-runner step as the first step in every job.
3. Prefer job-level `permissions` over top-level `permissions` to enforce least privilege.

### Release Integrity

- **GPG-signed tags** — All release tags are GPG-signed annotated tags.
- **GPG-signed commits** — All commits on `main` must be GPG-signed (enforced by repository rulesets).
- **Linear history** — Squash-merge-only policy prevents history rewriting; force-push to `main` is blocked.
- **CI gate** — All required status checks must pass before any merge to `main`.

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
