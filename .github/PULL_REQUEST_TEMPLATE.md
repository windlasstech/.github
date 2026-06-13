## Summary

<!-- Describe what this PR does and why. Include how to test the changes. -->

- What changed?
- Why is this needed?

## Related Issues

<!-- Link issues using keywords to auto-close them when this PR is merged: -->
<!-- closes #123, fixes #456, resolves #789 -->

- Closes #
- Related #

## Change Type

- [ ] Bug fix
- [ ] Feature
- [ ] Refactor
- [ ] Documentation
- [ ] Test/CI
- [ ] Breaking change
- [ ] Other: <!-- describe -->

## Changelog

<!--
Use this section to decide whether this PR needs a CHANGELOG.md update.

Follow Keep a Changelog:
- Write entries for meaningful user-facing changes, not raw commit history.
- During development, update only the [Unreleased] section.
- Group entries by category: Added, Changed, Deprecated, Removed, Fixed, Security.
- Do not create empty category sections.
- Use "None" for test cleanup, internal refactoring, CI-only changes, or other changes with no direct user-facing impact.

For release PRs:
- Move [Unreleased] entries into the new version section using Human Era five-digit years, e.g. ## [0.1.0] - 12026-06-13.
- Recreate an empty [Unreleased] section at the top.
- Update comparison links at the bottom.
- Use the finalized version section as the GitHub Release body.
-->

- Category: Added / Changed / Deprecated / Removed / Fixed / Security / None
- User-facing note:

Changelog update:

- [ ] `CHANGELOG.md` `[Unreleased]` updated
- [ ] Not needed because this change is not user-facing

## Checklist

### General

- [ ] PR title follows [Conventional Commits](https://www.conventionalcommits.org/) format: `type(scope): Summary`
- [ ] This PR does not expose backend/internal implementation details in a public repo.
- [ ] No secrets, tokens, keys, or private endpoints are included.
- [ ] Changes stay within this repository's intended scope.

### CI/Workflow Changes (if applicable)

If this PR modifies GitHub Actions workflows or CI/CD configuration, it must comply with our [Supply Chain Integrity requirements](SECURITY.md#supply-chain-integrity):

- [ ] All `uses:` references are pinned to full 40-character commit SHAs (with `# vX.Y.Z` comment)
- [ ] `step-security/harden-runner` is included as the first step in every job
- [ ] Job-level `permissions` are used instead of top-level `permissions`

### Protocol / Compatibility Impact

- [ ] No protocol/spec impact
- [ ] Protocol/spec updated
- [ ] Conformance tests updated
- [ ] Breaking change is versioned and migration notes are included

If impacted, describe compatibility impact:

## Testing

- [ ] Unit tests added/updated
- [ ] Integration or conformance tests added/updated
- [ ] Tests pass
- [ ] Lint and format pass
- [ ] Type check passes
- [ ] Manual verification performed

Describe test evidence:

## Documentation

- [ ] README updated
- [ ] Spec/docs updated
- [ ] Changelog decision completed above

## Rollout / Risk

- Risk level: Low / Medium / High
- Rollback plan:

## Reviewer Checklist

- [ ] Scope is clear and minimal
- [ ] Security and boundary checks passed
- [ ] Tests and docs are sufficient
- [ ] Compatibility impact is correctly handled
