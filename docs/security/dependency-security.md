# Dependency Security

> [!NOTE]
> This page is part of the Windlass organization security policy. Start with [SECURITY.md](../../SECURITY.md).

All projects must commit dependency lockfiles to version control to ensure reproducible builds and enable security auditing.

## Lockfile Requirements

| Ecosystem    | Lockfile                                     | Commit Required |
| ------------ | -------------------------------------------- | --------------- |
| Rust/Cargo   | `Cargo.lock`                                 | ✅ Yes          |
| Node.js/npm  | `package-lock.json`                          | ✅ Yes          |
| Node.js/Yarn | `yarn.lock`                                  | ✅ Yes          |
| Node.js/pnpm | `pnpm-lock.yaml`                             | ✅ Yes          |
| Bun          | `bun.lock`                                   | ✅ Yes          |
| Python/pip   | `requirements.txt` (pinned) or `poetry.lock` | ✅ Yes          |
| Go           | `go.sum`                                     | ✅ Yes          |
| Java/Maven   | `pom.xml` (with versions)                    | ✅ Yes          |
| Java/Gradle  | `gradle.lockfile`                            | ✅ Yes          |
| Ruby         | `Gemfile.lock`                               | ✅ Yes          |

## Dependency Update Automation

- Use Dependabot or Renovate to automatically update dependencies
- Security updates should be applied immediately
- Major version updates require review and testing

## Dependabot Cooldown Configuration

Cooldown periods delay version update PRs to allow community vetting of new releases. This mitigates supply chain attacks by providing time for malicious packages to be detected and yanked before adoption.

> [!NOTE]
> Security updates bypass cooldowns and are created immediately. Cooldowns apply only to version updates.

### Ecosystem-Specific Recommendations

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

### SemVer Cooldown Support by Ecosystem

| Support                   | Ecosystems                                                                                                                                                      |
| :------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SemVer + default-days** | `bundler`, `bun`, `cargo`, `composer`, `dotnet_sdk`, `elm`, `gomod`, `gradle`, `hex`, `julia`, `maven`, `npm`, `nuget`, `opentofu`, `pip`, `pub`, `swift`, `uv` |
| **default-days only**     | `bazel`, `devcontainers`, `docker`, `docker_compose`, `github-actions`, `gitsubmodule`, `helm`, `terraform`                                                     |

### Cooldown Rationale

**Why cooldowns matter:**

- Most malicious packages are detected within 24-72 hours of publication
- 7-day cooldown provides comfortable safety margin for community detection
- Organizations with cooldowns were not exposed during major incidents (e.g., Nx, xz utils backdoor)

**SemVer-specific delays:**

- **Patch (z in x.y.z)**: Bug fixes only; shortest cooldown acceptable
- **Minor (y in x.y.z)**: New features; moderate cooldown for API stability verification
- **Major (x in x.y.z)**: Breaking changes; longest cooldown for migration planning and community feedback

### Rust-Specific Considerations

The Rust ecosystem has unique characteristics affecting cooldown strategy:

1. **Strict SemVer Compliance**: Cargo enforces SemVer strictly. Minor updates are generally safer than in npm, but the community tends toward conservatism.

2. **Cargo Yank Mechanism**: Yanked crates remain available for existing `Cargo.lock` but prevent new adoption. Cooldowns work synergistically—malicious versions can be yanked before PRs are created.

3. **Recent Attack Patterns** (March 2026): Five malicious crates (`chrono_anchor`, `dnp3times`, etc.) targeted CI/CD environments to exfiltrate `.env` files. Cooldowns provide critical detection window.

4. **Cargo.lock Pinning**: Unlike npm, Cargo.lock should always be committed. This creates natural review gates that complement cooldowns.

### Example Configuration

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

### Additional Supply Chain Defenses

Combine cooldowns with:

- **`cargo-audit`**: Scan for known vulnerabilities in `Cargo.lock`
- **`cargo-vet`**: Establish trust relationships for dependencies
- **Dependency review workflows**: Require human review for new dependencies
- **Lockfile verification**: Always review `Cargo.lock`/`package-lock.json` diffs in PRs

## Dependency Review

The [Dependency Review Action](https://github.com/actions/dependency-review-action) scans pull requests for vulnerable or non-compliant dependencies before they enter the codebase. It performs diff-based analysis comparing dependencies in the PR head against the base, identifying changes that introduce known vulnerabilities or license violations.

> [!NOTE]
> Dependency Review is available for all public repositories. For private repositories, GitHub Advanced Security is required.

### Key Differences from Dependabot

| Feature         | Dependabot                         | Dependency Review                     |
| :-------------- | :--------------------------------- | :------------------------------------ |
| **Timing**      | Alerts on existing vulnerabilities | Blocks new vulnerabilities at PR time |
| **Scope**       | Scans entire dependency tree       | Scans only changed dependencies       |
| **Action**      | Creates update PRs                 | Fails PR checks to prevent merge      |
| **Integration** | Standalone alerts                  | GitHub Actions workflow               |

### Required Workflow Template

All repositories must implement Dependency Review with this hardened configuration:

```yaml
name: 'Dependency Review'

permissions:
  contents: read

on:
  pull_request:
    paths-ignore:
      - '**.md'
      - 'docs/**'
  merge_group:
    paths-ignore:
      - '**.md'
      - 'docs/**'

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
          config-file: 'windlasstech/.github/.github/dependency-review-config.yml@main'
```

### Configuration Options

| Option                  | Type    | Description                                                         | Recommended                              |
| :---------------------- | :------ | :------------------------------------------------------------------ | :--------------------------------------- |
| `fail-on-severity`      | string  | Minimum severity to block PR: `low`, `moderate`, `high`, `critical` | `critical`                               |
| `fail-on-scopes`        | string  | Scopes to check: `runtime`, `development`, `unknown`                | `development, runtime`                   |
| `allow-licenses`        | string  | Comma-separated allowed SPDX license IDs                            | See license policy below                 |
| `deny-licenses`         | string  | Comma-separated forbidden SPDX license IDs                          | Mutually exclusive with `allow-licenses` |
| `allow-ghsas`           | string  | GitHub Advisory IDs to temporarily allow                            | Document with justification              |
| `comment-summary-in-pr` | string  | PR comment mode: `always`, `on-failure`, `never`                    | `on-failure`                             |
| `warn-only`             | boolean | Log warnings without failing                                        | Use for phased rollout only              |

### Organization-Wide License Policy

Use an allow-list approach for license compliance. Reference a centralized configuration file:

```yaml
# In repository workflow
- name: 'Dependency Review'
  uses: actions/dependency-review-action@2031cfc080254a8a887f58cffee85186f0e49e48 # v4.9.0
  with:
    config-file: 'windlasstech/.github/.github/dependency-review-config.yml@main'
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

### Permissions Requirements

| Use Case                 | Required Permissions                     |
| :----------------------- | :--------------------------------------- |
| Basic vulnerability scan | `contents: read`                         |
| PR comments enabled      | `contents: read`, `pull-requests: write` |
| External config file     | `contents: read` + `external-repo-token` |

### Severity Levels

| Level        | Description                          | Recommended Action |
| :----------- | :----------------------------------- | :----------------- |
| **Critical** | Active exploitation or severe impact | Always fail        |
| **High**     | Significant security impact          | Fail in production |
| **Moderate** | Limited impact                       | Consider failing   |
| **Low**      | Minimal security risk                | Warn only          |

### Repository Ruleset Integration

Enforce Dependency Review across the organization using repository rulesets:

1. Navigate to **Organization Settings > Rules > Rulesets**
2. Create ruleset targeting all repositories
3. Add requirement: **Require status checks to pass**
4. Search for and select the `dependency-review` workflow

This ensures all PRs must pass dependency review before merging, regardless of individual repository configuration.

### Common Pitfalls

| Pitfall                                                     | Solution                                              |
| :---------------------------------------------------------- | :---------------------------------------------------- |
| Missing `pull-requests: write` with `comment-summary-in-pr` | Add permission to workflow job                        |
| Internal packages fail license check                        | Add to `allow-dependencies-licenses` with purl format |
| Both `allow-licenses` and `deny-licenses` set               | Choose one approach; they are mutually exclusive      |
| External config in private repo fails                       | Provide `external-repo-token` with read access        |
| Action fails before dependency submission completes         | Use `retry-on-snapshot-warnings: true`                |

## OSV Scanner

The [OSV Scanner](https://google.github.io/osv-scanner/) workflow detects known vulnerabilities in project dependencies using the Open Source Vulnerabilities database. It complements Dependency Review by scanning the entire dependency tree, not just changed dependencies.

### Key Differences from Dependency Review

| Feature      | Dependency Review           | OSV Scanner                               |
| :----------- | :-------------------------- | :---------------------------------------- |
| **Timing**   | Blocks new vulns at PR time | Scans all dependencies on PR and schedule |
| **Scope**    | Only changed dependencies   | Full dependency tree                      |
| **Database** | GitHub Advisory Database    | Open Source Vulnerabilities (OSV)         |
| **Output**   | PR annotations and comments | SARIF for GitHub Security tab             |

### Recommended Defaults

- `upload-sarif: true` — upload results to the repository's code scanning dashboard
- `fail-on-vuln: true` — fail the workflow when vulnerabilities are found
- `scan-args` defaulting to `--recursive ./` for automatic lockfile detection

Repository-specific overrides are supported:

- Set `upload-sarif: false` when SARIF upload should not be used for that repository
- Override `scan-args` for explicit lockfiles, generated dependency artifacts, or excluded paths
- Use `fail-on-vuln: false` only for short initial calibration, then restore the standard default of `true`

### Consumer Caller Example

```yaml
name: OSV Scanner PR
on:
  pull_request:
  merge_group:
permissions:
  contents: read
jobs:
  osv:
    permissions:
      actions: read
      contents: read
      security-events: write
    uses: windlasstech/.github/.github/workflows/osv-scanner-pr-reusable.yml@<pin-sha>
```

## References

### Dependency Review

- [Dependency Review Action Repository](https://github.com/actions/dependency-review-action)
- [GitHub Dependency Review Documentation](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review)
- [SPDX License List](https://spdx.org/licenses/)
- [GitHub Advisory Database](https://github.com/advisories)

### OSV Scanner

- [OSV Scanner Documentation](https://google.github.io/osv-scanner/)
- [OSV Scanner Usage](https://google.github.io/osv-scanner/usage/)
- [OSV Scanner Source Scanning](https://google.github.io/osv-scanner/usage/scan-source)
- [OSV Scanner License Scanning](https://google.github.io/osv-scanner/usage/license-scanning/)
- [OSV Scanner Configuration](https://google.github.io/osv-scanner/configuration/)
- [OSV Scanner Output Formats](https://google.github.io/osv-scanner/output/)
- [OSV Scanner GitHub Action](https://google.github.io/osv-scanner/github-action/)
- [OSV Scanner Action Repository](https://github.com/google/osv-scanner-action)

### Additional Resources

- [Rust Lockfile Guidelines](https://blog.rust-lang.org/2023/08/29/committing-lockfiles/)
