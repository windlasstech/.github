# .github

Centralized documents and information for the Windlass

## Development

### Markdown Linting and Formatting

This repository uses [markdownlint-cli2](https://github.com/DavidAnson/markdownlint-cli2) and [Prettier](https://prettier.io/) for consistent Markdown style.

#### Setup

```bash
bun install
```

#### Available Scripts

| Script | Description |
|--------|-------------|
| `bun run lint:md` | Lint all Markdown files |
| `bun run lint:md:fix` | Lint and auto-fix issues |
| `bun run format` | Format all Markdown files with Prettier |
| `bun run format:check` | Check formatting without modifying files |

#### Configuration

- **markdownlint**: `.markdownlint-cli2.jsonc` — Configured to avoid conflicts with Prettier
- **Prettier**: `.prettierrc` — Uses default Prettier settings for Markdown

#### CI/CD

Pull requests and pushes to `main` that modify Markdown files trigger automated linting and formatting checks via GitHub Actions.
