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

| Script                 | Description                              |
| ---------------------- | ---------------------------------------- |
| `bun run lint:md`      | Lint all Markdown files                  |
| `bun run lint:md:fix`  | Lint and auto-fix issues                 |
| `bun run format`       | Format all Markdown files with Prettier  |
| `bun run format:check` | Check formatting without modifying files |

#### Configuration

- **markdownlint**: `.markdownlint-cli2.jsonc` — Configured to avoid conflicts with Prettier
- **Prettier**: `.prettierrc` — Uses default Prettier settings for Markdown

#### Pre-commit Hooks

This repository uses [Lefthook](https://lefthook.dev/) to automatically lint and format Markdown files before each commit.

When you commit changes:

1. Staged `.md` and `.mdx` files are automatically linted with `markdownlint-cli2 --fix`
2. Files are formatted with `prettier --write`
3. Fixed files are re-staged automatically
4. If there are unfixable errors, the commit is blocked

The hooks are installed automatically when you run `bun install` via the `prepare` script.

To manually run the pre-commit hook:

```bash
bunx lefthook run pre-commit
```

#### CI/CD

Pull requests and pushes to `main` that modify Markdown files trigger automated linting and formatting checks via GitHub Actions.
