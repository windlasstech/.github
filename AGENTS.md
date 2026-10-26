# AGENTS.md: Windlass .github Repository

**Generated:** 12026-10-27  
**Commit:** 7479655  
**Branch:** main

---

## OVERVIEW

GitHub Organization Community Health Files repository, plus the org profile in `profile/`. Files
auto-apply across all Windlass org repos without their own versions. Content is Markdown and YAML
only (no application code, no symbol map). Tooling: Bun 1.4.4 (pinned), markdownlint-cli2,
`bun fmt`, Lefthook.

## STRUCTURE

```text
.
├── CODE_OF_CONDUCT.md              # Contributor Covenant 3.0 (en, default)
├── CONTRIBUTING.md                 # Contribution guidelines (HE date convention defined here)
├── SECURITY.md                     # Security policy overview; links docs/security/ companions
├── README.md                       # Repo docs + reusable workflow consumer guide
├── profile/README.md               # Org profile shown on the GitHub org page
├── docs/
│   ├── CODE_OF_CONDUCT.<tag>.md    # 8 translations: bn de es fr hu ja ko zh-cn
│   └── security/                   # Security companions (linked from SECURITY.md)
│       ├── artifact-attestations.md       # Provenance, SBOM, verification requirements
│       ├── dependency-security.md         # Dependency/lockfile policy, update automation
│       ├── slsa-compliance-framework.md   # SLSA source/build controls
│       └── workflow-hardening.md          # Actions/permissions/runner hardening
├── .github/
│   ├── CODEOWNERS                  # Global owner: @yunseo-kim
│   ├── PULL_REQUEST_TEMPLATE.md    # PR template
│   ├── dependabot.yml              # Weekly updates, grouping, cooldowns
│   ├── dependency-review-config.yml # Central dependency review policy
│   └── workflows/                  # Caller + reusable pairs
│       ├── lint.yml                        # Blocking markdownlint + actionlint
│       ├── autofix.yml                     # bun fmt via autofix.ci
│       ├── dependency-review.yml           # Caller -> dependency-review-reusable.yml
│       ├── dependency-review-reusable.yml  # Reusable dep vulnerability/license check
│       ├── scorecard.yml                   # Caller -> scorecard-reusable.yml
│       ├── scorecard-reusable.yml          # Reusable OpenSSF Scorecard analysis
│       ├── osv-scanner-pr-reusable.yml     # Reusable OSV PR diff scan
│       ├── osv-scanner-full-reusable.yml   # Reusable OSV full scan
│       └── osv-scanner-smoke.yml           # Reference consumer / smoke test
├── bunfig.toml                     # bun fmt config + install security scanner + release age
├── .markdownlint-cli2.jsonc        # Markdown lint rule overrides
├── .editorconfig                   # Editor defaults (mirrors fmt)
├── lefthook.yml                    # DCO sign-off + staged lint/format hooks
└── package.json                    # Bun 1.4.4 pin + lint/format scripts
```

## WHERE TO LOOK

| Need                             | Location                                            |
| :------------------------------- | :-------------------------------------------------- |
| Edit org-wide Code of Conduct    | `CODE_OF_CONDUCT.md`                                |
| Add translation                  | `docs/CODE_OF_CONDUCT.{lang}.md` + README table row |
| Edit org profile page            | `profile/README.md`                                 |
| Edit PR template                 | `.github/PULL_REQUEST_TEMPLATE.md`                  |
| Change code ownership            | `.github/CODEOWNERS`                                |
| Modify lint CI                   | `.github/workflows/lint.yml`                        |
| Modify formatting automation     | `.github/workflows/autofix.yml`                     |
| Change lint rules                | `.markdownlint-cli2.jsonc`                          |
| Change format rules              | `bunfig.toml` (`[fmt]`)                             |
| Change install security scan     | `bunfig.toml` (`[install.security]`)                |
| Modify pre-commit hooks          | `lefthook.yml`                                      |
| Modify dependency automation     | `.github/dependabot.yml`                            |
| Change dependency review caller  | `.github/workflows/dependency-review.yml`           |
| Change dependency review logic   | `.github/workflows/dependency-review-reusable.yml`  |
| Change dependency review policy  | `.github/dependency-review-config.yml`              |
| Change scorecard caller          | `.github/workflows/scorecard.yml`                   |
| Change scorecard logic           | `.github/workflows/scorecard-reusable.yml`          |
| Edit artifact attestation policy | `docs/security/artifact-attestations.md`            |
| Edit dependency security         | `docs/security/dependency-security.md`              |
| Edit SLSA compliance             | `docs/security/slsa-compliance-framework.md`        |
| Edit CI hardening                | `docs/security/workflow-hardening.md`               |
| Modify OSV PR scan               | `.github/workflows/osv-scanner-pr-reusable.yml`     |
| Modify OSV full scan             | `.github/workflows/osv-scanner-full-reusable.yml`   |
| Modify OSV smoke test            | `.github/workflows/osv-scanner-smoke.yml`           |

## CONVENTIONS

### Markdown Style (Deviations from Standard)

- **Tables:** Left-aligned headers with `:` prefix (`:---`)
- **Lists:** Dashes only (`-`), not asterisks (MD004)
- **Duplicate headings:** Checked only among siblings (MD024 `siblings_only`)
- **Thematic breaks:** `---` style (MD035)
- **Disabled rules:** MD003, MD007, MD009-MD013, MD018-MD023, MD027-MD033, MD041, MD048-MD050,
  MD060. Net effect: no line-length limit, trailing spaces allowed, ATX and setext headers both
  tolerated, emphasis-as-heading allowed. Full list: `.markdownlint-cli2.jsonc`

### Table Style (CRITICAL)

Compact tables with minimal spacing. Never align with excessive whitespace.

```markdown
| Header | Header |
| :----- | :----- |
| cell   | cell   |
```

### Linting/Formatting Stack

- **markdownlint-cli2**: `.markdownlint-cli2.jsonc`
- **bun fmt** (Bun built-in formatter): `bunfig.toml` `[fmt]`: printWidth 100, proseWrap always
  (hard-wraps prose), no semicolons, trailing commas, sorted imports
- **.editorconfig** mirrors fmt: UTF-8, LF, 2-space indent, final newline; Markdown keeps trailing
  whitespace for hard breaks
- **Lefthook**: pre-commit runs markdownlint `--fix` then `bun fmt` on staged files and re-stages
  them; unfixable errors block the commit

### Installs (`bunfig.toml`)

- Minimum package release age: 7 days
- Socket security scanner: `@socketsecurity/bun-security-scanner`

### Dates in human-facing documents

Use Holocene Era / Human Era year format for prose, changelog headings, ADR dates, and other
human-reader dates (e.g., `12026-06-23`). Machine-readable timestamps and protocol/schema fields,
such as JSON `generated_at`, must preserve the applicable technical standard format, normally ISO
8601 with a standard four-digit Gregorian year (e.g., `2026-06-23T12:00:00Z`).

### Guidelines for Using the Em Dash

In English prose the em dash is grammatically valid: prefer an alternative construction (comma,
parentheses, colon, or a sentence split) when the em dash offers no clear readability advantage, but
keep the em dash where it is genuinely clearer, such as around an interjection that already contains
parentheses or a comma-heavy list.

### Commits

- **DCO sign-off required**: Every commit must include a `Signed-off-by:` line. Use `git commit -s`
  (or `git commit --signoff`) for all commits. Lefthook enforces this in the commit-msg hook.
- **Commit message body format**: When a commit needs a multi-line body, prefer
  `git commit -s -F - <<'EOF'` (heredoc) over repeating `-m`. Repeating `-m` inserts blank lines
  between each argument and hurts readability; use heredoc unless the blank lines are intentional
  paragraph breaks.

### GitHub Actions / Workflows

- **Caller/reusable split**: Event-triggered callers (`dependency-review.yml`, `scorecard.yml`)
  invoke `*-reusable.yml` definitions via `workflow_call`. Edit logic in the reusable file, triggers
  in the caller.
- **SHA pinning**: All third-party actions are pinned to full-length SHAs. Local reusable `uses:`
  paths (e.g. `./.github/workflows/...`) are resolved at the caller's commit, so they are
  structurally immutable and exempt.
- **Consumer exception**: Consumers reference this repo's reusable workflows by `@main` as a
  documented risk acceptance (immediate fleet-wide propagation, not SHA-equivalent security);
  high-privilege consumers may pin a SHA. See README.md and
  `docs/security/workflow-hardening.md#internal-reusable-workflows`.
- **Harden Runner**: Every job runs harden-runner. `egress-policy: audit` is the floor; callers may
  pass `block` with an explicit `allowed-endpoints` list (this repo's callers use block mode).
- **Least privilege**: Workflow-level `permissions` are empty/read-only; jobs add only the scopes
  they need (`security-events: write` for SARIF, `id-token: write` for Scorecard publishing, etc.).

### Dependency Automation

- **Dependabot**: Weekly, Monday 09:00 Asia/Seoul; Bun + GitHub Actions ecosystems; grouped
  minor/patch updates for dev tools and action families; cooldowns 7 days (14 days Bun major);
  commit prefixes `chore`/`ci`.
- **Dependency review**: Fails at moderate severity across dev+runtime scopes; license allow-list;
  GHSA exceptions require a reason, expiry (max 90 days), and a security-team approver.
- **Security updates bypass cooldowns** (`docs/security/dependency-security.md`).

## ANTI-PATTERNS

- **Don't add `node_modules/` to git**: Already ignored
- **Don't use `*` bullets**: Use `-` only (MD004: dash style enforced)
- **Don't fight the formatter**: `bun fmt` hard-wraps prose ("proseWrap": "always", printWidth 100)
- **Don't use setext headers**: ATX preferred (lint-tolerated: MD003, MD018-020 disabled)
- **Don't commit without signing**: Org requires GPG/SSH/Sigstore signed commits
- **Don't delete `bun.lock`**: Lockfiles must stay committed; dependency review and OSV scans depend
  on them
- **Don't SHA-pin local reusable calls**: `uses: ./.github/workflows/...` is a path, not a ref;
  pinning applies to third-party actions, plus optional SHA pins for high-privilege consumers of
  this repo's cross-repo reusable workflows
- **Don't add a translation without the README row**: `docs/CODE_OF_CONDUCT.<tag>.md` files are
  listed in the README translations table; keep them in sync

## UNIQUE STYLES

- Org-wide file precedence: Repo file > `.github` repo default > missing
- Reusable OSV Scanner workflows provided for consumer repos
- SHA-pin required on all GitHub Actions (supply chain); internal reusable workflows are the
  documented exception (consumers reference `@main` as a risk acceptance)

## COMMANDS

```bash
# Setup
bun install                    # Installs deps + lefthook hooks

# Lint/Format
bun run lint:md               # Check markdown
bun run lint:md:fix           # Auto-fix issues
bun run format                # Format with bun fmt
bun run format:check          # Check formatting

# Pre-commit (manual)
bunx lefthook run pre-commit
```

## NOTES

- Lint CI runs on PRs + pushes to `main` touching Markdown or workflow files; autofix.ci applies
  formatting fixes
- Callers run harden-runner in `block` egress mode with explicit `allowed-endpoints`; reusable
  workflows default to `audit` and accept caller overrides
- `SECURITY.md` has full supply chain integrity requirements
- OSV Scanner reusable workflows support all ecosystems JS/Python/Go/Rust/Java/.NET/PHP
- Consumer repos: `lint.yml`/`autofix.yml` are reference starting points, not reusable;
  `osv-scanner-smoke.yml` is the reference consumer for the reusable OSV workflows

## cubic review workflow

Before committing, run `cubic review` to review uncommitted changes. Fix the validated issues and
run the local review again. Repeat until the local review is clean or only disputed issues remain.

After pushing, wait for cubic's GitHub review and check its comments. Fix the validated issues and
push the changes. Use GitHub review as the final pass; it may find issues the local review misses.
