# SLSA Compliance Framework

> [!NOTE]
> This page is part of the Windlass organization security policy. Start with [SECURITY.md](../../SECURITY.md).

This organization follows the [SLSA (Supply-chain Levels for Software Artifacts)](https://slsa.dev/) framework to ensure software supply chain security. SLSA provides a checklist of standards and controls to prevent tampering, improve integrity, and secure software packages.

## Current SLSA Level Status

<table>
  <thead>
    <tr>
      <th>Track</th>
      <th>Level</th>
      <th>Status</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3"><strong>Build Track</strong></td>
      <td><strong>SLSA Build L1</strong></td>
      <td>✅ Required</td>
      <td>Provenance exists documenting how artifacts are built</td>
    </tr>
    <tr>
      <td><strong>SLSA Build L2</strong></td>
      <td>✅ Required</td>
      <td>Hosted build platform with signed provenance</td>
    </tr>
    <tr>
      <td><strong>SLSA Build L3</strong></td>
      <td>🎯 Target</td>
      <td>Hardened builds with isolated, ephemeral environments</td>
    </tr>
    <tr>
      <td rowspan="4"><strong>Source Track</strong></td>
      <td><strong>SLSA Source L1</strong></td>
      <td>✅ Required</td>
      <td>Version controlled source with provenance</td>
    </tr>
    <tr>
      <td><strong>SLSA Source L2</strong></td>
      <td>✅ Required</td>
      <td>Immutable history with source provenance attestations</td>
    </tr>
    <tr>
      <td><strong>SLSA Source L3</strong></td>
      <td>🎯 Target</td>
      <td>Continuous technical controls on protected branches</td>
    </tr>
    <tr>
      <td><strong>SLSA Source L4</strong></td>
      <td>🚫 Blocked</td>
      <td>Two-party review (requires multiple trusted persons)</td>
    </tr>
  </tbody>
</table>

> [!NOTE]
> SLSA v1.2 defines Build Track levels 0-3 and Source Track levels 1-4. Build L0 represents no SLSA guarantees. Source L4 requires two-party review and is not achievable for a single-person organization. See [Source Track Achievability for 1-Person Organizations](#source-track-achievability-for-1-person-organizations) for details.

## SLSA Build Track Requirements

### Build L0 — No Guarantees

Build L0 represents the absence of SLSA Build Track guarantees. No requirements are imposed. This is the default state for software that does not follow the SLSA Build Track.

### Build L1 — Provenance Exists

- All builds must generate provenance describing:
  - Entity that performed the build
  - Build process and steps used
  - Top-level inputs (source repository, commit SHA, dependencies)
- Provenance must be in SLSA format or equivalent

### Build L2 — Hosted Build Platform

- All production builds must run on hosted CI/CD platforms (GitHub Actions)
- Provenance must be cryptographically signed by the build platform
- No local/developer workstation builds for releases

### Build L3 — Hardened Builds

- Builds must run in isolated, ephemeral environments
- Build steps cannot access platform signing keys
- Concurrent builds cannot influence each other
- Cache poisoning must be prevented between builds
- All external interactions must be captured in provenance

## SLSA Source Track Requirements

The SLSA Source Track ensures the integrity and trustworthiness of source code throughout the development lifecycle. It provides increasing guarantees about how source revisions are created, managed, and protected.

### Source L1 — Version Controlled

The source is stored and managed through a modern version control system.

Requirements:

- Use a version control system (VCS) such as Git
- Repositories must be uniquely identifiable with a stable locator (URI)
- Revisions must be immutable and uniquely identifiable (e.g., Git commit SHA)
- The SCS must provide tooling to display changes in a human-readable form (diffs)
- Source Verification Summary Attestations (Source VSAs) must be generated for revisions at Level 1 or above

Implementation for this organization:

- All repositories are hosted on GitHub with unique URIs (`github.com/windlasstech/{repo}`)
- All commits use Git SHA-1 digests for immutable identification
- GitHub provides diff tooling for all changes
- Repository rulesets enforce version control standards

### Source L2 — History and Provenance

Branch history is continuous, immutable, and retained. The source control system issues Source Provenance attestations for each new revision.

Requirements:

- Change history must be continuous and immutable
- The SCS must record all changes to named references (branches, tags), including when they occurred, who made them, and the new revision ID
- Branch updates must only point to revisions that descend from the current revision (no force push or history rewriting)
- Source Provenance attestations must be created contemporaneously with branch updates
- Identity management must be configured to authenticate actors and attribute actions
- Tags must be protected from being moved or deleted

Implementation for this organization:

- Branch protection rules require linear history (no merge commits) on `main`
- Force pushes to `main` are blocked via repository rulesets
- All commits must be cryptographically signed (GPG, SSH, or Sigstore gitsign)
- GitHub audit logs record all changes to branches and tags
- Tag protection rules prevent tag deletion and movement
- Repository rulesets enforce signed commits on protected branches

### Source L3 — Continuous Technical Controls

The source control system enforces the organization's technical controls for specific named references within the repository.

Requirements:

- The SCS must enforce customized technical controls for protected named references (branches, tags)
- The organization must document the meaning of all enforced technical controls
- Evidence of continuous enforcement must be provided via Source Provenance attestations or VSAs
- Technical controls must be continuously enforced from a specific start revision (continuity)
- If a control is disabled and re-enabled, continuity must be re-established

Implementation for this organization:

- Repository rulesets protect `main` and release branches with required status checks
- All PRs must pass CI checks (markdown lint, dependency review, scorecard) before merge
- Required reviewers are enforced on all PRs to protected branches
- Status checks include: OpenSSF Scorecard, Dependency Review, CodeQL, OSV Scanner
- Branch protection requires signed commits
- The organization documents all technical controls in this policy

### Source L4 — Two-Party Review

Changes to protected branches require review and agreement by two or more trusted persons prior to submission.

Requirements:

- Changes must be agreed to by two different trusted persons (uploader + reviewer, or two reviewers)
- Reviews must cover security-relevant properties of the code
- The final revision submitted must be the one reviewed (reset votes on changes)
- Approvals are context-specific (repo + branch)
- The SCS must present reviewers with a clear representation of the proposed change
- Trusted robots may be granted exceptions for automated changes (e.g., Dependabot)

Achievability for this organization:

- **Not achievable under a 1-person organization model.** Source L4 explicitly requires two distinct trusted persons for every change. A single individual cannot satisfy the "two-party" requirement by definition.
- See [Source Track Achievability for 1-Person Organizations](#source-track-achievability-for-1-person-organizations) for mitigation strategies and alternative controls.

## Source Track Achievability for 1-Person Organizations

### Maximum Achievable Level: Source L3

For a single-person organization, **Source L3 is the maximum realistically achievable level** under the SLSA v1.2 specification. Source L4 is fundamentally incompatible with a 1-person model because it requires two distinct trusted persons to review every change.

### Why Source L4 Is Not Achievable

The SLSA Source L4 requirement for two-party review exists specifically to mitigate insider threats and unilateral changes. The specification states:

> Changes in protected branches MUST be agreed to by two or more trusted persons prior to submission.

A "trusted person" is defined as a human authorized by the organization to propose and approve changes. A single individual cannot simultaneously be two different trusted persons. This is not a technical limitation but a structural requirement of the security model.

### Achieving Source L3: Practical Implementation

A 1-person organization can achieve Source L3 by implementing the following controls using GitHub features:

1. **Branch Protection Rulesets**
   - Protect `main` and all release branches
   - Require pull request reviews (even if self-reviewed, the mechanism is enforced)
   - Require status checks to pass before merging
   - Require signed commits
   - Block force pushes

2. **Required Status Checks**
   - OpenSSF Scorecard analysis
   - Dependency Review
   - CodeQL code scanning
   - Markdown lint and format checks
   - Any repository-specific test suites

3. **Commit Signing Enforcement**
   - Require all commits on protected branches to be signed
   - Use Sigstore gitsign for keyless signing (recommended)
   - Alternative: GPG or SSH signing

4. **Audit Logging**
   - Enable GitHub audit logs for all repository events
   - Monitor for unexpected access patterns or policy violations
   - Export logs for long-term retention

5. **Repository Rulesets (GitHub)**
   - Use rulesets instead of legacy branch protection for granular control
   - Apply rulesets organization-wide for consistency
   - Require ruleset bypassers to be documented

### Mitigating the Absence of Source L4

While two-party review is not possible with one person, the following compensating controls reduce the risk of unilateral malicious changes:

| Control                         | Purpose                       | Implementation                                                            |
| :------------------------------ | :---------------------------- | :------------------------------------------------------------------------ |
| **Self-review checklist**       | Force deliberate review       | Maintain a security checklist that must be completed before self-approval |
| **Automated security scanning** | Detect malicious patterns     | CodeQL, Semgrep, or similar tools scanning for suspicious code            |
| **Immutable audit trail**       | Enable post-hoc investigation | Signed commits + GitHub audit logs provide tamper-evident history         |
| **External monitoring**         | Detect unauthorized changes   | OpenSSF Scorecard monitors branch protection and signed commits           |
| **Artifact attestations**       | Verify build integrity        | All releases include signed SLSA Build provenance                         |

## Transition Path to Source L4

If the organization grows to multiple trusted contributors, the following steps enable Source L4:

1. Add at least one additional trusted person with write access
2. Configure branch protection to require 1 reviewer (GitHub minimum)
3. Document the review policy requiring security-relevant review
4. Maintain the existing Source L3 controls as baseline
5. Update this policy to reflect the new achievable level

## References

### SLSA Framework

- [SLSA Specification v1.2](https://slsa.dev/spec/v1.2/)
- [SLSA Get Started Guide](https://slsa.dev/how-to/get-started)
- [SLSA for Organizations](https://slsa.dev/how-to/how-to-orgs)
- [SLSA for Infrastructure Providers](https://slsa.dev/how-to/how-to-infra)
- [SLSA GitHub Generator](https://github.com/slsa-framework/slsa-github-generator)
