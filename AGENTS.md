# Agent Guidelines

## Project

**ZenthCloud Actions** — the official GitHub Actions integration for ZenthCloud, maintained by Sky Genesis Enterprise. Derived from `cloudflare/wrangler-action` (MIT, attribution preserved in LICENSE).

## Commands

- `npm install` — install dependencies
- `npm run build` — compile the action with `@vercel/ncc` into `dist/index.mjs` (must be rebuilt and committed with every source change)
- `npm test` — run the vitest suite
- `npm run format` — format with prettier
- `npm run check` — verify formatting with prettier
- `npm run release` — compile and tag a release (driven by changesets, see `scripts/release.sh`)

## Code style

- TypeScript with strict mode enabled, ESM (`"type": "module"`)
- Formatting and linting: prettier (tabs, double quotes), no ESLint config is present
- Tests: vitest, colocated with sources (`src/*.test.ts`, `src/**/*.spec.ts`)
- GitHub Actions Toolkit libraries (`@actions/core`, `@actions/exec`, `@actions/github`) provide inputs, outputs, exec and GitHub API mechanics — reuse them instead of reimplementing
- Input validation: zod schemas (`zenthcloudActionConfig`)
- Node.js 24 (`.nvmrc`), the compiled action must run on the `node24` runner

## Conventions

- The compiled artifact `dist/index.mjs` is committed with releases — GitHub Actions executes it, not the TypeScript sources
- User-facing changes require a changeset (`.changeset/`, via `npx changeset`)
- Never commit secrets, tokens or credentials
- The action is being migrated from the legacy wrangler engine to the ZenthCloud CI/CD API — keep the migration honest in docs and metadata, and do not claim ZenthCloud compatibility for code paths that do not exist yet
