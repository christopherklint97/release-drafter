# CLAUDE.md

## Overview

Fork of [release-drafter/release-drafter](https://github.com/release-drafter/release-drafter). A GitHub Action (and Probot app) that automatically drafts release notes as pull requests are merged. It supports autolabeling PRs, categorizing changes, and semantic version resolution.

Upstream: `https://github.com/release-drafter/release-drafter`
Fork: `https://github.com/christopherklint97/release-drafter`

## Project Structure

```
release-drafter/
├── index.js              # Main app entry: Probot event handlers (autolabeler + drafter)
├── action.js             # GitHub Actions adapter entry point
├── action.yml            # GitHub Action metadata (inputs, outputs, branding)
├── lib/
│   ├── commits.js        # Find commits and associated PRs via GitHub API
│   ├── config.js         # Load and merge config from .github/release-drafter.yml
│   ├── default-config.js # Default configuration values
│   ├── log.js            # Logging helper
│   ├── pagination.js     # GitHub API pagination utilities
│   ├── releases.js       # Find, create, update releases; generate release info
│   ├── schema.js         # Joi schema for config validation
│   ├── sort-pull-requests.js # Sort PRs by merged_at or title
│   ├── template.js       # Template variable replacement engine
│   ├── triggerable-reference.js # Check if git ref should trigger a draft
│   ├── utils.js          # Utility helpers (e.g., runnerIsActions check)
│   └── versions.js       # Semantic version parsing and resolution
├── dist/                 # Compiled action bundle (built via ncc)
├── test/
│   ├── fixtures/         # Test fixtures (configs, API responses)
│   ├── helpers/          # Test helper utilities
│   ├── index.test.js     # Main integration tests (~112KB)
│   └── *.test.js         # Unit tests for lib modules
├── bin/
│   ├── generate-schema.js
│   └── generate-fixtures.js
├── schema.json           # Generated JSON schema for config
├── docker-compose.yml    # Docker-based test runner
└── Dockerfile            # Container for running as Probot app
```

## Tech Stack

- **Runtime:** Node.js >= 20 (CommonJS modules)
- **Framework:** [Probot](https://probot.github.io/) (GitHub App framework)
- **Action adapter:** `@probot/adapter-github-actions`
- **Build:** `@vercel/ncc` (bundles to `dist/index.js`)
- **Test:** Jest with `--experimental-vm-modules`, nock for HTTP mocking
- **Lint:** ESLint + Prettier
- **Git hooks:** Husky + lint-staged

## Key Commands

```sh
npm install          # Install dependencies
npm run test         # Run tests (Jest)
npm run test:watch   # Watch mode with coverage
npm run build        # Bundle action with ncc → dist/
npm run lint         # ESLint
npm run prettier     # Format code
npm run generate-schema  # Regenerate schema.json from Joi schema
```

## How It Works

1. **Autolabeler mode** (`pull_request` / `pull_request_target` events): Matches PR files, branch, title, or body against rules in `autolabeler` config and adds labels.
2. **Drafter mode** (`push` events / GitHub Actions): Finds the last release, collects merged PRs since then, generates release notes from templates, and creates or updates a draft release.

Config is read from `.github/release-drafter.yml` in the target repo. Action inputs (in workflow YAML) override config file values.

## Development Workflow

- Default branch: `master`
- Tests must pass before pushing (`postversion` script runs tests)
- The `dist/` folder contains the compiled action; rebuild with `npm run build` after changing source
- Releasing: `git checkout master && git pull && npm version [major|minor|patch]`
