# AGENTS.md — Windlass .github Repository

**Generated:** 2026-04-26
**Commit:** b394f5f
**Branch:** refactor/security-docs-split

---

## OVERVIEW

GitHub Organization Community Health Files repository. Files auto-apply across all Windlass org repos without their own versions.

## STRUCTURE

```text
.
├── CODE_OF_CONDUCT.md              # Contributor Covenant 3.0 (en)
├── CONTRIBUTING.md                 # Contribution guidelines
├── SECURITY.md                     # Security policy + SLSA compliance
├── README.md                       # This repo's documentation
├── docs/                           # Translations + security docs
│   ├── CODE_OF_CONDUCT.ko.md      # Korean
│   ├── CODE_OF_CONDUCT.zh-cn.md   # Chinese Simplified
│   ├── CODE_OF_CONDUCT.de.md      # German
│   ├── CODE_OF_CONDUCT.fr.md      # French
│   └── security/                   # Security companion docs
│       ├── dependency-security.md       # Dependency security policy
│       ├── slsa-compliance-framework.md # SLSA compliance details
│       └── workflow-hardening.md        # CI hardening guidelines
├── .github/
│   ├── PULL_REQUEST_TEMPLATE.md    # PR template
│   ├── dependabot.yml              # Dependency update automation
│   ├── dependency-review-config.yml # Dependency review policy
│   └── workflows/                  # CI workflows
│       ├── markdown-lint.yml            # Markdown linting CI
│       ├── dependency-review.yml        # Supply chain security
│       ├── scorecard.yml                # OpenSSF Scorecard
│       ├── osv-scanner-pr-reusable.yml  # Reusable OSV PR scan
│       ├── osv-scanner-full-reusable.yml # Reusable OSV full scan
│       └── osv-scanner-smoke.yml        # OSV integration smoke test
└── [config files]
```

## WHERE TO LOOK

| Need                          | Location                                          |
| :---------------------------- | :------------------------------------------------ |
| Edit org-wide Code of Conduct | `CODE_OF_CONDUCT.md`                              |
| Add translation               | `docs/CODE_OF_CONDUCT.{lang}.md`                  |
| Edit PR template              | `.github/PULL_REQUEST_TEMPLATE.md`                |
| Modify CI checks              | `.github/workflows/markdown-lint.yml`             |
| Change lint rules             | `.markdownlint-cli2.jsonc`                        |
| Change format rules           | `.prettierrc`                                     |
| Modify pre-commit hooks       | `lefthook.yml`                                    |
| Modify dependency automation  | `.github/dependabot.yml`                          |
| Change dependency review      | `.github/workflows/dependency-review.yml`         |
| Change security scorecard     | `.github/workflows/scorecard.yml`                 |
| Edit dependency security      | `docs/security/dependency-security.md`            |
| Edit SLSA compliance          | `docs/security/slsa-compliance-framework.md`      |
| Edit CI hardening             | `docs/security/workflow-hardening.md`             |
| Modify OSV PR scan            | `.github/workflows/osv-scanner-pr-reusable.yml`   |
| Modify OSV full scan          | `.github/workflows/osv-scanner-full-reusable.yml` |
| Modify OSV smoke test         | `.github/workflows/osv-scanner-smoke.yml`         |

## CONVENTIONS

### Markdown Style (Deviations from Standard)

- **Tables:** Left-aligned headers with `:` prefix (`:---`)
- **Lists:** Dashes only (`-`), not asterisks (MD004)
- **Line length:** No hard limit (MD013 disabled)
- **Trailing spaces:** Allowed (MD009 disabled)
- **No setext headers** — ATX style only (MD003 disabled)

### Table Style (CRITICAL)

Compact tables with minimal spacing. Never align with excessive whitespace.

```markdown
| Header | Header |
| :----- | :----- |
| cell   | cell   |
```

### Linting Stack

- **markdownlint-cli2** (v0.22.0) — `.markdownlint-cli2.jsonc`
- **Prettier** (v3.8.1) — `.prettierrc`
- **Lefthook** — pre-commit automation

## ANTI-PATTERNS

- **Don't add `node_modules/` to git** — Already ignored
- **Don't use `*` bullets** — Use `-` only (MD004: dash style enforced)
- **Don't hard-wrap prose** — Prettier preserves wraps ("proseWrap": "preserve")
- **Don't use setext headers** — ATX style required (MD003 disabled, MD018-020 disabled)
- **Don't commit without signing** — Org requires GPG/SSH/Sigstore signed commits

## UNIQUE STYLES

- Org-wide file precedence: Repo file > `.github` repo default > missing
- Reusable OSV Scanner workflows provided for consumer repos
- SHA-pin required on all GitHub Actions (supply chain)

## COMMANDS

```bash
# Setup
bun install                    # Installs deps + lefthook hooks

# Lint/Format
bun run lint:md               # Check markdown
bun run lint:md:fix           # Auto-fix issues
bun run format                # Format with Prettier
bun run format:check          # Check formatting

# Pre-commit (manual)
bunx lefthook run pre-commit
```

## NOTES

- CI runs on PRs + pushes to `main` touching `**/*.md`
- Uses hard-runner audit mode, SHA-pinned actions
- `SECURITY.md` has full supply chain integrity requirements
- OSV Scanner reusable workflows support all ecosystems JS/Python/Go/Rust/Java/.NET/PHP
