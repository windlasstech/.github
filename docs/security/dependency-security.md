# Dependency Security

> [!NOTE]  
> This page is part of the Windlass organization security policy. Start with
> [SECURITY.md](../../SECURITY.md).

All projects must commit dependency lockfiles to version control to ensure reproducible builds and
enable security auditing.

## Dependency Defense Overview

Dependency security uses layered controls so each tool covers a different point in the lifecycle:

| Control               | Timing                       | Scope                        | Primary outcome                              |
| :-------------------- | :--------------------------- | :--------------------------- | :------------------------------------------- |
| **Lockfiles**         | During development and build | Resolved dependency graph    | Reproducible builds and reviewable diffs     |
| **Dependabot**        | Continuous and scheduled     | Existing dependency versions | Update PRs for vulnerable or stale packages  |
| **Cooldowns**         | Before version update PRs    | Newly published versions     | Time for malicious releases to be detected   |
| **Dependency Review** | Pull request                 | Newly changed dependencies   | Blocks vulnerable or non-compliant additions |
| **OSV Scanner**       | Pull request and schedule    | Full dependency tree         | Finds known vulnerabilities across projects  |

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

### Dependabot Cooldown Configuration

Cooldown periods delay version update PRs to allow community vetting of new releases. This mitigates
supply chain attacks by providing time for malicious packages to be detected and yanked before
adoption.

**Organization baseline: a uniform 7-day minimum release age across all ecosystems.** This is a
floor, not a target — the longer windows in the table below remain recommended where listed. The
baseline is a deliberate conservative choice — OpenSSF guidance suggests ~3 days and several package
managers default to 1 day — and it satisfies the Semgrep cooldown rules, which all require at least
7 days. SemVer-major updates use 14 days as an explicit organizational choice.

> [!NOTE]  
> Security updates bypass cooldowns and are created immediately. Cooldowns apply only to version
> updates.

#### Ecosystem-Specific Recommendations

| Ecosystem                 | Patch     | Minor      | Major      | Rationale                                                                        |
| :------------------------ | :-------- | :--------- | :--------- | :------------------------------------------------------------------------------- |
| **GitHub Actions**        | N/A       | N/A        | N/A        | `default-days: 7`; SemVer cooldown not supported. SHA-pinning mitigates risk     |
| **NPM/Bun (Production)**  | 7 days    | 7-14 days  | 21-30 days | Aligns with the install-time release-age gate and the Semgrep 7-day baseline     |
| **NPM/Bun (Development)** | 7 days    | 7 days     | 14 days    | Same 7-day floor as production; dev dependencies get no shorter window           |
| **Rust/Cargo**            | 7 days    | 14-21 days | 30-60 days | Stricter SemVer compliance allows longer vetting; Cargo.lock provides safety net |
| **Python/pip**            | 7-14 days | 14-21 days | 21-30 days | PyPI has slower yank response; conservative approach recommended                 |
| **Go**                    | 7 days    | 7-14 days  | 14-21 days | Go modules are immutable; supply chain attacks less common but possible          |

> [!IMPORTANT]  
> SemVer-based cooldown (`semver-major-days`, `semver-minor-days`, `semver-patch-days`) is only
> supported by package ecosystems that use Semantic Versioning. Ecosystems like **GitHub Actions**,
> **Docker**, **Terraform**, and **Bazel** only support `default-days`.

#### SemVer Cooldown Support by Ecosystem

| Support                   | Ecosystems                                                                                                                                                      |
| :------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SemVer + default-days** | `bundler`, `bun`, `cargo`, `composer`, `dotnet_sdk`, `elm`, `gomod`, `gradle`, `hex`, `julia`, `maven`, `npm`, `nuget`, `opentofu`, `pip`, `pub`, `swift`, `uv` |
| **default-days only**     | `bazel`, `devcontainers`, `docker`, `docker_compose`, `github-actions`, `gitsubmodule`, `helm`, `terraform`                                                     |

#### Cooldown Rationale

**Why cooldowns matter:**

- Most malicious packages are detected within 24-72 hours of publication
- 7-day cooldown provides comfortable safety margin for community detection
- Organizations with cooldowns were not exposed during major incidents (e.g., Nx, xz utils backdoor)

**SemVer-specific delays:**

- **Patch (z in x.y.z)**: Bug fixes only; shortest cooldown acceptable
- **Minor (y in x.y.z)**: New features; moderate cooldown for API stability verification
- **Major (x in x.y.z)**: Breaking changes; longest cooldown for migration planning and community
  feedback

#### Rust-Specific Considerations

The Rust ecosystem has unique characteristics affecting cooldown strategy:

1. **Strict SemVer Compliance**: Cargo enforces SemVer strictly. Minor updates are generally safer
   than in npm, but the community tends toward conservatism.

2. **Cargo Yank Mechanism**: Yanked crates remain available for existing `Cargo.lock` but prevent
   new adoption. Cooldowns work synergistically: malicious versions can be yanked before PRs are
   created.

3. **Recent Attack Patterns** (March 2026): Five malicious crates (`chrono_anchor`, `dnp3times`,
   etc.) targeted CI/CD environments to exfiltrate `.env` files. Cooldowns provide critical
   detection window.

4. **Cargo.lock Pinning**: Unlike npm, Cargo.lock should always be committed. This creates natural
   review gates that complement cooldowns.

#### Example Configuration

```yaml
# dependabot.yml - Production-grade Rust project
version: 2
updates:
  - package-ecosystem: "cargo"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "tuesday"
    cooldown:
      default-days: 7
      semver-major-days: 30
      semver-minor-days: 14
      semver-patch-days: 7
    groups:
      patch-updates:
        patterns: ["*"]
        update-types: ["patch"]
    reviewers:
      - "windlass-tech/security-team"
```

#### Install-Time Release-Age Gates

Dependabot cooldowns only delay update PRs; several package managers can also block installation of
too-recent versions directly. Configure both so an update bot never proposes a version the installer
would refuse: the install-time gate matches the bot's baseline window (`default-days` and the
minor/patch cooldowns, 7 days). The gate must never exceed the bot's cooldown; a repository choosing
a longer window from the ranges above simply makes the bot more conservative than the gate, which is
safe. SemVer-major cooldowns longer than 7 days are a review and migration-planning delay, not a
release-age gate, and need no installer equivalent.

| Ecosystem  | File                         | Setting                       | Org value  | Units             |
| :--------- | :--------------------------- | :---------------------------- | :--------- | :---------------- |
| Bun        | `bunfig.toml`                | `[install].minimumReleaseAge` | `604800`   | seconds           |
| Bundler    | `Gemfile` (per source)       | `cooldown`                    | `7`        | days              |
| npm        | `.npmrc`                     | `min-release-age`             | `7`        | days              |
| pnpm       | `pnpm-workspace.yaml`        | `minimumReleaseAge`           | `10080`    | minutes           |
| Poetry     | `poetry.toml`                | `[solver].min-release-age`    | `7`        | days              |
| uv         | `pyproject.toml` `[tool.uv]` | `exclude-newer`               | `"7 days"` | relative duration |
| Yarn Berry | `.yarnrc.yml`                | `npmMinimalAgeGate`           | `"7d"`     | duration string   |

On the bot side, Renovate repositories set `minimumReleaseAge: "7 days"` as the equivalent of
Dependabot's cooldown; it delays update PRs but does not gate installation.

Unlike Dependabot, install-time gates also delay urgent security fixes. Every gate must ship with a
documented, narrow exception path for emergency patches — Bun package/exact-version exceptions, pnpm
`minimumReleaseAgeExclude`, uv `exclude-newer-package`, Poetry `min-release-age-exclude`, Yarn
`npmPreapprovedPackages`; where a manager offers no per-package exception (for example npm), the
emergency path is a reviewed config change that is reverted immediately after the hotfix lands
(Renovate needs no installer exception because it is a bot, not an install-time gate). Exceptions
must be approved by a maintainer, scoped to a specific package and version, and time-boxed. These
exceptions do not expire on their own: the approving maintainer must record a removal date (for
example in the PR or issue that introduced the exception) and delete the entry once the fixed
release has aged past the gate. Never disable a gate globally to land a hotfix.

pip, Cargo, Go modules, Maven/Gradle, NuGet, and Composer have no native release-age gate (Python
repositories using uv are covered by the gate above). For repositories in those ecosystems,
compensate with the Dependabot cooldown above plus lockfile diff review, Dependency Review and OSV
Scanner gates, and — where available — a proxy registry that delays mirroring new releases.

### Additional Supply Chain Defenses

Combine cooldowns with:

- **`cargo-audit`**: Scan for known vulnerabilities in `Cargo.lock`
- **`cargo-vet`**: Establish trust relationships for dependencies
- **Dependency review workflows**: Require human review for new dependencies
- **Lockfile verification**: Always review `Cargo.lock`/`package-lock.json` diffs in PRs

## Dependency Review

The [Dependency Review Action](https://github.com/actions/dependency-review-action) scans pull
requests for vulnerable or non-compliant dependencies before they enter the codebase. It performs
diff-based analysis comparing dependencies in the PR head against the base, identifying changes that
introduce known vulnerabilities or license violations.

> [!NOTE]  
> Dependency Review is available for all public repositories. For private repositories, GitHub
> Advanced Security is required.

### Required Workflow Template

All repositories must implement Dependency Review with this hardened configuration:

```yaml
name: "Dependency Review"

permissions:
  contents: read

on:
  pull_request:
    paths-ignore:
      - "**.md"
      - "docs/**"
  merge_group:
    paths-ignore:
      - "**.md"
      - "docs/**"

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
      - name: Harden the runner (Audit all outbound calls)
        uses: step-security/harden-runner@e14015d583714f6e62063499dc959a02595150a1 # v2.21.1
        with:
          egress-policy: audit

      - name: "Checkout Repository"
        uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2
        with:
          persist-credentials: false

      - name: "Dependency Review"
        uses: actions/dependency-review-action@a1d282b36b6f3519aa1f3fc636f609c47dddb294 # v5.0.0
        with:
          config-file: "windlasstech/.github/.github/dependency-review-config.yml@main"
```

### Configuration Options

| Option                  | Type    | Description                                                         | Recommended                              |
| :---------------------- | :------ | :------------------------------------------------------------------ | :--------------------------------------- |
| `fail-on-severity`      | string  | Minimum severity to block PR: `low`, `moderate`, `high`, `critical` | `moderate`                               |
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
- name: "Dependency Review"
  uses: actions/dependency-review-action@a1d282b36b6f3519aa1f3fc636f609c47dddb294 # v5.0.0
  with:
    config-file: "windlasstech/.github/.github/dependency-review-config.yml@main"
```

**Standard allow-licenses list** (mirrors the centralized configuration):

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
  # Google patent license for Go standard library dependencies
  - LicenseRef-scancode-google-patent-license-golang # Google patent license for Go (BSD-3-Clause + patent grant)
  # Language-specific permissive licenses
  - Python-2.0 # Python Software Foundation License - permissive
  - PHP-3.01 # PHP License v3.01 - permissive
  - Ruby # Ruby License - permissive
```

> [!NOTE]  
> License identifiers must be SPDX-compliant. For licenses detected as `OTHER`, use
> `LicenseRef-clearlydefined-OTHER` in allow/deny lists.

`LicenseRef-scancode-google-patent-license-golang` is allowed because Go toolchain dependencies such
as `golang.org/x/sys` and `golang.org/x/term` are licensed
`BSD-3-Clause AND LicenseRef-scancode-google-patent-license-golang`. The Google patent grant is a
permissive patent clause, not a restrictive license, so it must not block otherwise-compliant Go
dependencies.

#### Package-Scoped License Exemptions

When a package's detected license expression contains a false-positive `LicenseRef-*` but the
underlying licenses are all on the allow-list, exempt the **package** via
`allow-dependencies-licenses` instead of adding the `LicenseRef-*` to `allow-licenses`. Package
scoping keeps the policy tight: GitHub-internal identifiers such as `LicenseRef-bad-*` are unstable
and could appear on arbitrary packages, so widening the global allow-list for them would silently
weaken enforcement.

Current organization-wide exemptions in the centralized configuration:

| Package purl (all versions)                  | Reported expression                          | Rationale                                                                                                                           |
| :------------------------------------------- | :------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| `pkg:golang/github.com/go-openapi/runtime`   | `LicenseRef-bad-apache-2.0-mit`              | Apache-2.0, but bundles the MIT-licensed `naoina/denco` router; both Apache-2.0 and MIT are allow-listed                            |
| `pkg:golang/github.com/sassoftware/relic/v8` | `Apache-2.0 AND LicenseRef-scancode-dco-1.1` | Apache-2.0; `ContributorAgreement.txt` embeds the DCO 1.1 text, which ScanCode classifies as a CLA document, not a software license |

Exemption policy:

- Prefer package-scoped `allow-dependencies-licenses` entries over widening `allow-licenses` for
  unstable or non-standard `LicenseRef-*` detections.
- Document the ground truth for each exemption (upstream `LICENSE` files, ScanCode LicenseDB
  classification) in a comment next to the entry.
- Leave versions unpinned (`purl` without `@version`) so future Dependabot bumps do not re-trip the
  check. Trade-off: a genuinely non-permissive license introduced in a future version of an exempted
  package would not be flagged. Re-review exemptions when upstream packaging changes.
- Because consumer repositories reference the centralized configuration via `@main`, every change to
  it takes effect organization-wide on merge; review exemption changes accordingly.

Exemptions are declared in the centralized configuration,
[`.github/dependency-review-config.yml`](../../.github/dependency-review-config.yml), using purl
format with a rationale comment per entry:

```yaml
allow-dependencies-licenses:
  # <package>: <why the detected expression is a false positive; upstream ground truth>
  - pkg:golang/github.com/go-openapi/runtime
  # <package>: <why the detected expression is a false positive; upstream ground truth>
  - pkg:golang/github.com/sassoftware/relic/v8
```

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

This ensures all PRs must pass dependency review before merging, regardless of individual repository
configuration.

### Common Pitfalls

| Pitfall                                                     | Solution                                              |
| :---------------------------------------------------------- | :---------------------------------------------------- |
| Missing `pull-requests: write` with `comment-summary-in-pr` | Add permission to workflow job                        |
| Internal packages fail license check                        | Add to `allow-dependencies-licenses` with purl format |
| Both `allow-licenses` and `deny-licenses` set               | Choose one approach; they are mutually exclusive      |
| External config in private repo fails                       | Provide `external-repo-token` with read access        |
| Action fails before dependency submission completes         | Use `retry-on-snapshot-warnings: true`                |

## OSV Scanner

The [OSV Scanner](https://google.github.io/osv-scanner/) workflow detects known vulnerabilities in
project dependencies using the Open Source Vulnerabilities database. It complements Dependency
Review by scanning the entire dependency tree, not just changed dependencies.

### Recommended Defaults

- `upload-sarif: true`: upload results to the repository's code scanning dashboard
- `fail-on-vuln: true`: fail the workflow when vulnerabilities are found
- `scan-args` defaulting to `--recursive ./` for automatic lockfile detection

Repository-specific overrides are supported:

- Set `upload-sarif: false` when SARIF upload should not be used for that repository
- Override `scan-args` for explicit lockfiles, generated dependency artifacts, or excluded paths
- Use `fail-on-vuln: false` only for short initial calibration, then restore the standard default of
  `true`

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

The `.github` repository dogfoods both reusable workflows in its
[`osv-scanner-smoke.yml`](../../.github/workflows/osv-scanner-smoke.yml) workflow. Consult it as a
working reference consumer when wiring the PR diff scan and full scan into your own repository.

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

### Dependency Cooldown

- [OpenSSF Concise Guide for Developing More Secure Software](https://best.openssf.org/Concise-Guide-for-Developing-More-Secure-Software.html)
- [GitHub Dependabot cooldown configuration](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/optimizing-pr-creation-version-updates)
- [Bun minimumReleaseAge](https://bun.com/docs/pm/cli/install#minimum-release-age)
- [pnpm minimumReleaseAge](https://pnpm.io/settings#minimumreleaseage)
- [Yarn npmMinimalAgeGate](https://yarnpkg.com/configuration/yarnrc#npmMinimalAgeGate)
- [uv dependency cooldowns](https://docs.astral.sh/uv/concepts/resolution/#dependency-cooldowns)
- [Poetry solver.min-release-age](https://python-poetry.org/docs/configuration/#solvermin-release-age)
- [Bundler cooldown configuration](https://bundler.io/v4.0/man/bundle-config.1.html)
- [Semgrep package-manager cooldown rules](https://github.com/semgrep/semgrep-rules/tree/develop/package_managers)

### Additional Resources

- [Rust Lockfile Guidelines](https://blog.rust-lang.org/2023/08/29/committing-lockfiles/)
