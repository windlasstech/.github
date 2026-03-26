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
   > [!NOTE]
   > Please keep changes small and focused.
5. Implement changes with tests/docs updates as needed.
6. Ensure CI is green.
7. Open a PR using the PR template.

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

### Versioning and Compatibility

- Breaking protocol changes must be explicit and versioned.
- When behavior changes, update `protocol-spec` and `conformance-tests` together.
- Include migration notes for maintainers of third-party clients when applicable.

### CI/CD and Supply Chain Integrity

When contributing workflow changes or release-related code, you **must** follow the supply chain integrity requirements documented in [`SECURITY.md`](SECURITY.md#supply-chain-integrity).

Key requirements include:

- **SHA-pinned actions**: All `uses:` references in GitHub Actions workflows must be pinned to the full 40-character commit SHA (add `# vX.Y.Z` comment for readability).
- **Harden-runner**: Every job must start with `step-security/harden-runner` in audit mode.
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
