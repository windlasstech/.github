# Contributing Guide

Thank you for your interest in contributing.

This project follows a controlled-open contribution model:

- Public contribution is encouraged for clients, protocol specs, conformance tests, and documentation.
- Backend implementation and internal operations remain private.

Please review this guide before opening a pull request.

All of your interactions in the project are subject to our [Code of Conduct](CODE_OF_CONDUCT.md). This includes creation of issues or pull requests, commenting on issues or pull requests, and extends to all interactions in any real-time space.

## Scope and Contribution Boundaries

This repository accepts contributions for:

- Client libraries and official client behavior
- Protocol/spec clarifications
- Conformance tests and fixtures
- Documentation and examples
- Developer tooling for interoperability

This repository does not accept direct contributions for:

- Private backend service internals
- Internal infrastructure design and operational runbooks

If your change depends on non-public backend behavior, open an issue first so maintainers can guide the correct path.

## Contribution Categories

### Direct PRs Welcome

- Documentation improvements
- Non-security bug fixes
- SDK/client integrations
- Tests and benchmarks

### Require Prior Discussion (Issue or RFC)

- API behavior changes
- Protocol-level changes
- Dependency upgrades with broad impact
- Performance-critical logic changes

### May Be Declined Without Prior Alignment

- Core architecture redesigns
- Security-sensitive logic modifications without private review path
- Major refactors not tied to an approved issue/RFC
- Features that conflict with current roadmap direction

## Contribution Process

1. Check existing issues and discussions to avoid duplicate work.
2. For non-trivial changes, open an issue with problem statement and approach.
3. For substantial changes, submit an RFC issue before implementation.
4. Fork and create a short-lived branch:
   - `feature/<short-description>`
   - `fix/<short-description>`
   - `docs/<short-description>`
5. Implement changes with tests/docs updates as needed.
6. If the change is meaningful to users or downstream integrators, update `CHANGELOG.md` under `[Unreleased]` in the same PR.
7. Ensure CI is green.
8. Open a PR using the PR template, including the changelog category and user-facing note.

> [!NOTE]
> Please keep changes in each PR small and focused.

## RFC Path (Required for Substantial Changes)

Open an RFC issue for changes that alter behavior or cross repository boundaries.

An RFC should include:

- Problem statement
- Proposed solution
- Alternatives considered
- Compatibility and migration impact

PRs implementing substantial changes without prior RFC discussion may be closed and redirected to the RFC path.

## Development Expectations

### Code Quality

- Follow existing style and repository conventions.
- Keep dependencies minimal and justified.
- Prefer incremental, reviewable changes over large rewrites.

### Testing

- Add tests for new behavior and bug fixes.
- Keep existing tests passing.
- Update conformance tests when protocol behavior changes.

### Commit and PR Quality

- Use clear, structured commit messages.
- Explain why the change is needed, not only what changed.
- Link related issue(s) in the PR.

### Documentation and Prose Conventions

- Dates in human-facing documents: use Holocene Era / Human Era years for prose, changelog headings, ADR dates, and other human-reader dates (for example, `12026-06-23`). Machine-readable timestamps and protocol/schema fields, such as JSON `generated_at`, keep the applicable technical standard, normally ISO 8601 with a four-digit Gregorian year (for example, `2026-06-23T12:00:00Z`).
- Em dashes: in English prose the em dash is grammatically valid, but prefer an alternative construction (comma, parentheses, colon, or a sentence split) when it offers no clear readability advantage. Keep the em dash where it is genuinely clearer, such as around an interjection that already contains parentheses or a comma-heavy list.

### Changelog Management

Projects in the organization should maintain a `CHANGELOG.md` following [Keep a Changelog 2.0.0](https://keepachangelog.com/en/2.0.0/). Pin the format link to the version you follow so it stays accurate as the specification evolves.

Use changelog entries to summarize what users, operators, or downstream integrators need to know when upgrading. Do not generate changelog entries by listing commits. Curate, don't accumulate: record notable changes only, written for the reader.

File structure:

- Open with a `# Changelog` heading and a short, fixed preamble stating what the file is, that the format is based on Keep a Changelog (pinned to 2.0.0), and which versioning scheme the project adheres to.
- Keep an `[Unreleased]` section at the top to collect upcoming changes.
- Start each version with its number and date, for example `## [0.1.0] - 12026-06-13`, and resolve each version heading as a reference-style comparison link at the bottom of the file.

> [!NOTE]
> Windlass convention: changelog dates use Holocene Era five-digit years (e.g. `12026-06-13` rather than `2026-06-13`), keeping the largest-to-smallest ISO 8601 ordering. Keep a Changelog 2.0.0 requires a consistent, stated date convention; this is ours.

For every PR, complete the PR template's `Changelog` section:

- **Category**: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`, or `None`
- **User-facing note**: A short description of the impact, or why no note is needed

Use `None` for changes with no direct user-facing impact, such as test cleanup, internal refactoring, formatting, or CI-only maintenance.

The six categories are fixed by the format — do not invent new ones (for example, there is no `Performance` or `Dependencies` type; describe the effect under the right existing type). Choosing between the ambiguous three: use `Fixed` when the old behavior was a bug, `Changed` when the old behavior was intentional and now works differently, and `Security` when the change addresses a vulnerability.

Changelog categories describe changes relative to a previous release. While a project is pre-release and no earlier version exists, do not add categories that presuppose previously released behavior, such as `Fixed`: correcting unreleased behavior revises the relevant `[Unreleased]` entry text instead, and PRs whose changes are not user-facing use `None`.

During development:

- Update only the `[Unreleased]` section when a PR has user-facing impact.
- Group entries by `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, and `Security`.
- Do not create empty category sections.
- Mark breaking changes inline with a `**Breaking:**` marker within `Changed` or `Removed`, and keep them with their type rather than collecting them in a separate section. State which interface breaks (CLI, API, protocol, file format, configuration schema); put substantial upgrade steps in a linked migration guide, not in the entry.
- Record a dependency vulnerability fix under `Security` if the vulnerable package is a production dependency, or, even for `devDependencies`, if its exploitation could theoretically affect build or release artifacts (for example, code execution inside the test or release pipeline).
- Lead a `Security` entry with its CVE identifier when one is assigned (`- CVE-XXXX-YYYY: ...`); when no CVE exists, lead with the applicable alternative identifier instead, such as a GHSA or a Go vulnerability database ID (`- GHSA-XXXX-XXXX-XXXX: ...`, `- GO-XXXX-YYYY: ...`). Link each leading identifier to its OSV entry, for example `- [CVE-2026-12345](https://osv.dev/vulnerability/CVE-2026-12345): out-of-bounds read when parsing malformed input.`
- A version may open with a short optional summary paragraph on the theme of the release before the typed sections.
- Announce deprecations before acting on them: mark a feature `Deprecated` in one release and `Removed` only in a later one, naming the version that will remove it.
- List yanked releases rather than hiding them, and mark them like `## [0.0.5] - 12014-12-13 [YANKED]`.
- You may fix a changelog after release (a missed entry, an unrecorded breaking change); when you do, note the date the entry was updated.

For release PRs:

- Move `[Unreleased]` entries into the new version section, for example `## [0.1.0] - 12026-06-13`.
- Recreate an empty `[Unreleased]` section at the top, pointing at `HEAD`.
- Update comparison links at the bottom of `CHANGELOG.md`.
- Use the finalized version section as the GitHub Release body. The changelog is the canonical record; derive release notes from it rather than maintaining two documents. Host-generated release notes live in the host's database, not your repository. Treat them as output, never as the source.

Automation and AI assistance:

- Machines can draft, but humans curate. An AI tool may draft entries from a diff, but a maintainer must decide what is notable, group it, and write it for the reader before merge.
- If coding agents work on the repository, record the changelog brief (summarize notable user-facing changes; sort into the six types; mark breaking changes; drop anything not worth reading) where agents read it, such as `AGENTS.md`.
- Conventional Commits and CI tooling play a supporting role: they may propose versions or check formatting, but a changelog edit must never be a required check on every PR; that fills the changelog with noise.

### Versioning and Compatibility

- Breaking protocol changes must be explicit and versioned.
- When behavior changes, update `protocol-spec` and `conformance-tests` together.
- Include migration notes for maintainers of third-party clients when applicable.

### CI/CD and Supply Chain Integrity

When contributing workflow changes or release-related code, you **must** follow the supply chain integrity requirements documented in [`SECURITY.md`](SECURITY.md#supply-chain-integrity).

Key requirements include:

- **SHA-pinned actions**: All `uses:` references in GitHub Actions workflows must be pinned to the full 40-character commit SHA (add `# vX.Y.Z` comment for readability).
- **Harden Runner (StepSecurity Enterprise)**: Every job in every repository, public or private, must start with `step-security/harden-runner`, at minimum in audit mode (`egress-policy: audit`). Once a workflow's baseline of outbound destinations is established, switching to `egress-policy: block` with an `allowed-endpoints` allow-list is recommended.
- **StepSecurity Maintained Actions**: When a third-party action is needed, prefer the [StepSecurity Maintained Action](https://docs.stepsecurity.io/github-actions/actions/stepsecurity-maintained-actions) replacement where one exists.
- **GPG-signed commits**: All commits must be GPG-signed.
- **Job-level permissions**: Prefer job-level `permissions` over top-level `permissions`.

Dependabot automatically updates SHA-pinned action references via weekly PRs.

## Security Reporting

> [!IMPORTANT]
> Do not report vulnerabilities in public issues or PRs.

Follow [`SECURITY.md`](SECURITY.md) for private disclosure and vulnerability reporting procedures.
Public vulnerability details may be removed until coordinated remediation is complete.

## Legal: License and DCO/CLA

By contributing, you agree that your contribution can be distributed under this repository's license.

If this repository enforces DCO, sign off commits (for example: `git commit -s`).
If this repository requires a CLA, complete CLA requirements before merge.

## Maintainer Review and Discretion

Maintainers may request changes, defer, or close PRs that are:

- Out of scope
- In conflict with project direction
- Inactive for an extended period

A closed PR is not necessarily final; a revised proposal with new context is welcome.

## Support Boundary

This repository is for contribution work.
General product support or private backend questions should use the designated support channel.
