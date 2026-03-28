# AGENTS.md — Windlass .github Repository

**Type:** GitHub Organization Community Health Files Repository
**Scope:** Organization-wide defaults (applied to all repos without their own versions)

---

## WHAT THIS IS

Special `.github` repository per [GitHub convention](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file). Files here auto-apply across all Windlass organization repositories.

---

## STRUCTURE

```text
.
├── CODE_OF_CONDUCT.md           # Contributor Covenant 3.0 (en)
├── CONTRIBUTING.md              # Contribution guidelines
├── SECURITY.md                  # Security policy + SLSA compliance
├── README.md                    # This repo's documentation
├── docs/                        # Translations
│   ├── CODE_OF_CONDUCT.ko.md   # Korean
│   ├── CODE_OFDUCT.zh-cn.md    # Chinese Simplified
│   ├── CODE_OF_CONDUCT.de.md   # German
│   └── CODE_OF_CONDUCT.fr.md   # French
├── .github/
│   ├── PULL_REQUEST_TEMPLATE.md # PR template
│   └── workflows/               # CI workflows
│       └── markdown-lint.yml   # Markdown linting CI
└── [config files]
```

---

## WHERE TO LOOK

| Need                          | Look Here                             |
| :---------------------------- | :------------------------------------ |
| Edit org-wide Code of Conduct | `CODE_OF_CONDUCT.md`                  |
| Add translation               | `docs/CODE_OF_CONDUCT.{lang}.md`      |
| Edit PR template              | `.github/PULL_REQUEST_TEMPLATE.md`    |
| Modify CI checks              | `.github/workflows/markdown-lint.yml` |
| Change lint rules             | `.markdownlint-cli2.jsonc`            |
| Change format rules           | `.prettierrc`                         |
| Modify pre-commit hooks       | `lefthook.yml`                        |

---

## CONVENTIONS (PROJECT-SPECIFIC)

### Markdown Style

- **Tables:** Left-aligned headers with `:` prefix (`:---`)
- **Lists:** Dashes only (`-`), not asterisks
- **Headers:** ATX style (`#`), no setext
- **Line length:** No hard limit (MD013 disabled)
- **Trailing spaces:** Allowed (MD009 disabled)

### Linting Stack

- **markdownlint-cli2** (v0.22.0) — rules in `.markdownlint-cli2.jsonc`
- **Prettier** (v3.8.1) — config in `.prettierrc`
- **Lefthook** — pre-commit automation

### Table Style (CRITICAL)

Compact tables use minimal spacing. Example:

```markdown
| Header | Header |
| :----- | :----- |
| cell   | cell   |
```

NOT aligned tables with excessive whitespace.

---

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

---

## ANTI-PATTERNS (NEVER DO)

- **Don't add `node_modules/` to git** — Already ignored
- **Don't use `*` bullets** — Use `-` only (MD004: dash style enforced)
- **Don't hard-wrap prose** — Prettier preserves wraps ("proseWrap": "preserve")
- **Don't use setext headers** — ATX style required (MD003 disabled, MD018-020 disabled)
- **Don't commit without signing** — Org requires GPG/SSH/Sigstore signed commits

---

## ORG-WIDE FILE PRECEDENCE

1. Repo has its own file → Uses that
2. Repo has no file → Inherits from this `.github` repo
3. Visibility: Files appear in GitHub UI across all org repos

---

## CI/CD NOTES

- Runs on: PRs + pushes to `main` that touch `**/*.md`
- Uses: GitHub Actions with hard-runner audit mode
- Must: SHA-pin all actions (supply chain requirement)
- See: `SECURITY.md` for full supply chain integrity requirements
