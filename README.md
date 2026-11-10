# ZenthCloud Actions

[![License: MIT](https://img.shields.io/github/license/skygenesisenterprise/zenthcloud-actions)](LICENSE)
[![Node.js 24](https://img.shields.io/badge/Node.js-24-339933?logo=nodedotjs&logoColor=white)](https://nodejs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)](https://typescriptlang.org)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-node24-2088FF?logo=githubactions&logoColor=white)](https://docs.github.com/en/actions)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](.github/CONTRIBUTING.md)
[![Discord](https://img.shields.io/badge/Discord-join-5865F2?logo=discord&logoColor=white)](https://discord.gg/GUkDugVh34)

> Deploy, manage and integrate applications with ZenthCloud — from GitHub Actions.

**ZenthCloud Actions** is the official GitHub Actions integration for [ZenthCloud](https://github.com/skygenesisenterprise/zenthcloud), the open-source, self-hostable cloud platform for deploying and operating applications, databases and infrastructure. It turns a GitHub workflow into a deployment pipeline: trigger builds, follow their status and surface the results in the repository, across all three ZenthCloud modes — **Self**, **Hybrid** and **Cloud**.

**Perfect for:** platform and DevOps teams deploying to ZenthCloud from CI, agencies automating client infrastructure, and any repository that wants deployments driven by pull requests, pushes or manual dispatch instead of the dashboard.

> **Status:** this repository is derived from the open-source [`cloudflare/wrangler-action`](https://github.com/cloudflare/wrangler-action) project and is being migrated to the ZenthCloud CI/CD API (`resolve`, `deploy`, `status`, `logs`, `cancel`, `redeploy`). Until that integration lands, the deploy action still runs through the bundled legacy build engine inherited from the upstream project. Only the behavior documented in this README is operational — nothing here claims ZenthCloud compatibility for code paths that do not exist yet.

## Contents

- [ZenthCloud Actions](#zenthcloud-actions)
  - [Contents](#contents)
  - [Why ZenthCloud Actions matters](#why-zenthcloud-actions-matters)
  - [What you can do](#what-you-can-do)
    - [Deploy](#deploy)
    - [Report](#report)
  - [Quick start](#quick-start)
  - [Authentication](#authentication)
  - [Configuration](#configuration)
    - [Inputs](#inputs)
    - [Outputs](#outputs)
    - [Permissions](#permissions)
  - [Examples](#examples)
  - [Troubleshooting](#troubleshooting)
  - [Development](#development)
  - [FAQ](#faq)
  - [Related resources](#related-resources)
    - [Official](#official)
    - [Community](#community)
    - [Commercial](#commercial)
  - [Contributing](#contributing)
  - [Security](#security)
  - [License and provenance](#license-and-provenance)
  - [Note](#note)

## Why ZenthCloud Actions matters

🎯 **The official integration** — maintained by Sky Genesis Enterprise alongside the ZenthCloud platform, aligned with the ZenthCloud CI/CD API instead of an unrelated vendor's tooling.

🚀 **Deploy from any event** — push to `main`, pull requests, schedules or manual `workflow_dispatch`, targeting the environment your project defines.

📦 **Environments, secrets and vars** — bind values from the workflow `env` to your project, so credentials never live in the repository.

🔒 **Least privilege by default** — a deploy workflow needs `contents: read` only, and the token is passed through GitHub secrets, never printed into logs.

🧩 **One integration, three modes** — the same workflow targets ZenthCloud Self, Hybrid and Cloud installations.

🤝 **Open source** — MIT licensed, with the provenance of the derived code preserved and attributable.

## What you can do

### Deploy

- Trigger a deployment of your project from a GitHub workflow, on push, pull request, schedule or manual dispatch.
- Deploy to a named environment and bind secrets and vars from the workflow environment.
- Run commands before and after the deployment with `preCommands` and `postCommands`.
- Pick the package manager (`npm`, `yarn`, `pnpm`, `bun`) or let the action infer it from your lockfile.

### Report

- Expose the command output, standard error and deployment URL as step outputs for later workflow steps.
- Create GitHub Deployments and job summaries when a `gitHubToken` is provided.
- Keep workflow logs quiet with the `quiet` input when the output is not needed.

## Quick start

Create a workflow file in your repository (`.github/workflows/deploy.yml`):

```yaml
name: Deploy

on:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Deploy
    steps:
      - uses: actions/checkout@v6
      - name: Deploy to ZenthCloud
        uses: skygenesisenterprise/zenthcloud-actions@v1
        with:
          apiToken: ${{ secrets.DEPLOY_API_TOKEN }}
```

The token lives in the repository secrets (see [Authentication](#authentication)); the deployment runs on every push to `main`.

## Authentication

The action authenticates with an API token stored as a [GitHub secret](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions):

1. In your repository, open "Settings → Secrets and variables → Actions".
2. Create a secret (for example `DEPLOY_API_TOKEN`) holding your deployment token.
3. Reference it in the `with` block of the workflow.

```yaml
- name: Deploy to ZenthCloud
  uses: skygenesisenterprise/zenthcloud-actions@v1
  with:
    apiToken: ${{ secrets.DEPLOY_API_TOKEN }}
```

Never paste tokens directly into a workflow file. The action never prints the token into logs.

> The dedicated ZenthCloud CI/CD token flow — scoped project tokens created from the dashboard, with `ZENTHCLOUD_API_TOKEN` and `ZENTHCLOUD_API_URL` inputs — is part of the ongoing migration to the ZenthCloud CI/CD API and is not available yet.

## Configuration

### Inputs

| Input              | Description                                                                                     | Required |
| ------------------ | ----------------------------------------------------------------------------------------------- | -------- |
| `apiToken`         | Deployment API token, provided through GitHub secrets                                           | No       |
| `accountId`        | Account identifier forwarded to the legacy build engine                                         | No       |
| `environment`      | Target environment — must be defined in your project configuration                              | No       |
| `quiet`            | Suppresses output from deployment commands (default `false`)                                    | No       |
| `workingDirectory` | Relative path the deployment commands run from                                                  | No       |
| `command`          | Deployment command to run, defaults to `deploy`; multiple commands can be separated by newlines | No       |
| `secrets`          | Environment variable names, newline separated, bound to your project as secrets                 | No       |
| `vars`             | Environment variable names, newline separated, bound to your project as vars                    | No       |
| `preCommands`      | Commands to execute before deploying                                                            | No       |
| `postCommands`     | Commands to execute after deploying                                                             | No       |
| `packageManager`   | `npm`, `yarn`, `pnpm` or `bun` — inferred from the lockfile when omitted                        | No       |
| `wranglerVersion`  | Legacy engine only: version of the bundled build tool to install                                | No       |
| `gitHubToken`      | GitHub token, used to create GitHub Deployments and job summaries                               | No       |

### Outputs

| Output                       | Description                                                             |
| ---------------------------- | ----------------------------------------------------------------------- |
| `command-output`             | Standard output of the deployment command                               |
| `command-stderr`             | Standard error output of the deployment command                         |
| `deployment-url`             | URL of the deployment, when the command produced one                    |
| `pages-deployment-alias-url` | Pages deployment alias URL, when applicable (legacy engine)             |
| `pages-deployment-id`        | Pages deployment ID, when applicable (legacy engine)                    |
| `pages-environment`          | Pages deployment environment, when applicable (legacy engine)           |
| `preview-url`                | Stable preview URL, when the command produced a preview (legacy engine) |
| `preview-deployment-url`     | Immutable preview deployment URL (legacy engine)                        |
| `preview-name`               | Preview name (legacy engine)                                            |
| `preview-id`                 | Preview resource ID (legacy engine)                                     |
| `preview-deployment-id`      | Preview deployment ID (legacy engine)                                   |

### Permissions

Minimal permissions for a deploy workflow:

```yaml
permissions:
  contents: read
```

Add `deployments: write` only when you pass `gitHubToken`, so the action can publish GitHub Deployments and job summaries:

```yaml
permissions:
  contents: read
  deployments: write
```

## Examples

### Deploy on merge to main

```yaml
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - name: Deploy to ZenthCloud
        uses: skygenesisenterprise/zenthcloud-actions@v1
        with:
          apiToken: ${{ secrets.DEPLOY_API_TOKEN }}
```

### Deploy to an environment with secrets and vars

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - name: Deploy to ZenthCloud
        uses: skygenesisenterprise/zenthcloud-actions@v1
        with:
          apiToken: ${{ secrets.DEPLOY_API_TOKEN }}
          environment: production
          secrets: |
            SECRET1
            SECRET2
          vars: |
            MY_VAR
        env:
          SECRET1: ${{ secrets.SECRET1 }}
          SECRET2: ${{ secrets.SECRET2 }}
          MY_VAR: production-value
```

Every name listed in `secrets` or `vars` must have a matching entry in the workflow `env` block.

### Run a custom command

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - name: Deploy to ZenthCloud
        uses: skygenesisenterprise/zenthcloud-actions@v1
        with:
          apiToken: ${{ secrets.DEPLOY_API_TOKEN }}
          command: deploy --dry-run
```

### Use outputs in a later step

```yaml
- name: Deploy to ZenthCloud
  id: deploy
  uses: skygenesisenterprise/zenthcloud-actions@v1
  with:
    apiToken: ${{ secrets.DEPLOY_API_TOKEN }}

- name: Print deployment URL
  run: echo "${{ steps.deploy.outputs.deployment-url }}"
```

### GitHub Deployments and job summaries

```yaml
permissions:
  contents: read
  deployments: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - name: Deploy to ZenthCloud
        uses: skygenesisenterprise/zenthcloud-actions@v1
        with:
          apiToken: ${{ secrets.DEPLOY_API_TOKEN }}
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}
```

### Manually triggering a deployment

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Choose an environment to deploy to: <dev|staging|prod>"
        required: true
        default: "dev"

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - name: Deploy to ZenthCloud
        uses: skygenesisenterprise/zenthcloud-actions@v1
        with:
          apiToken: ${{ secrets.DEPLOY_API_TOKEN }}
          environment: ${{ github.event.inputs.environment }}
```

### Deploying on a schedule

```yaml
on:
  schedule:
    - cron: "0 * * * *"

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - name: Deploy to ZenthCloud
        uses: skygenesisenterprise/zenthcloud-actions@v1
        with:
          apiToken: ${{ secrets.DEPLOY_API_TOKEN }}
```

## Troubleshooting

- **"Value for secret X not found in environment"** — every name listed in `secrets` must have a matching entry in the workflow `env` block.
- **"Directory X does not exist"** — `workingDirectory` must be a path relative to the repository root.
- **Authentication failures** — check that the token secret exists and has not expired.
- **No `deployment-url` in outputs** — the command did not produce a deployment URL; only deployment commands emit one.

## Development

The repository uses **Node.js 24** (see `.nvmrc`). Install dependencies from the repository root:

```bash
npm install
```

### Root commands

| Command           | Purpose                                                     |
| ----------------- | ----------------------------------------------------------- |
| `npm run build`   | Compile the action with `@vercel/ncc` into `dist/index.mjs` |
| `npm test`        | Run the vitest suite                                        |
| `npm run check`   | Verify formatting with prettier                             |
| `npm run format`  | Format the repository with prettier                         |
| `npm run release` | Compile and tag a release (driven by changesets)            |

The compiled artifact `dist/index.mjs` is what GitHub Actions executes, not the TypeScript sources — rebuild and commit it together with every source change. Tests use mocks only; no deployment is ever triggered from the test suite.

## FAQ

### Is ZenthCloud Actions free?

Yes. The project is open source and licensed under the MIT License. ZenthCloud itself is open source; a commercial edition with additional features and premium support is maintained by [Sky Genesis Enterprise](https://skygenesisenterprise.com).

### Which ZenthCloud installations does it support?

The same workflow targets all three modes: **ZenthCloud Self** (self-hosted), **ZenthCloud Hybrid** and **ZenthCloud Cloud**. Today the action still runs through the bundled legacy build engine; targeting ZenthCloud instances directly through the ZenthCloud CI/CD API is the next milestone of the migration.

### What do I need to use it?

A GitHub repository, a deployment token stored in repository secrets, and a project configured in your ZenthCloud installation. The action itself runs on the `node24` runner — no other runtime is required.

### Where is the documentation?

This README documents the action. The ZenthCloud product documentation lives at [docs.zenthcloud.com](https://docs.zenthcloud.com).

### How do I get help or report a bug?

Open a [GitHub issue](https://github.com/skygenesisenterprise/zenthcloud-actions/issues), or use the support routes listed at [zenthcloud.com/resources/support](https://zenthcloud.com/resources/support). Never report a security vulnerability in a public issue — see [Security](#security).

## Related resources

### Official

- ZenthCloud platform: [github.com/skygenesisenterprise/zenthcloud](https://github.com/skygenesisenterprise/zenthcloud)
- Product docs: [docs.zenthcloud.com](https://docs.zenthcloud.com)
- Website: [zenthcloud.com](https://zenthcloud.com)
- Console: [dash.zenthcloud.com](https://dash.zenthcloud.com)
- Status: [status.zenthcloud.com](https://status.zenthcloud.com)
- Support: [zenthcloud.com/resources/support](https://zenthcloud.com/resources/support)

### Community

- Bugs and feature requests: [GitHub Issues](https://github.com/skygenesisenterprise/zenthcloud-actions/issues)
- Contribution guide: [`.github/CONTRIBUTING.md`](.github/CONTRIBUTING.md)
- Code of Conduct: [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)
- Discord: [join the ZenthCloud community](https://discord.gg/GUkDugVh34)

### Commercial

- Commercial edition and premium support: [Sky Genesis Enterprise](https://skygenesisenterprise.com)
- Licensing and legal: [legal@skygenesisenterprise.com](mailto:legal@skygenesisenterprise.com)

## Contributing

Contributions and suggestions are welcome. Please read the [Code of Conduct](CODE_OF_CONDUCT.md) before participating, then follow the [contribution guide](.github/CONTRIBUTING.md) for setup and review expectations.

- Keep PR titles in [Conventional Commits](https://www.conventionalcommits.org) format (`fix: …`, `feat: …`, `docs: …`).
- Add a changeset with `npx changeset` when your change is user-facing.
- Run `npm test`, `npm run build` and `npm run check` before opening a pull request.

## Security

Do not report security vulnerabilities in public issues. Follow the private reporting instructions in [SECURITY.md](SECURITY.md), or email **security@skygenesisenterprise.com**.

## License and provenance

ZenthCloud Actions is licensed under the [MIT License](LICENSE).

It is derived from [`cloudflare/wrangler-action`](https://github.com/cloudflare/wrangler-action), also MIT licensed. The upstream copyright notice is preserved in the [LICENSE](LICENSE) file, and the history of the derived code, up to upstream release `4.1.3`, lives in the [upstream changelog](https://github.com/cloudflare/wrangler-action/blob/main/CHANGELOG.md).

## Note

ZenthCloud Actions is an open-source project maintained by [Sky Genesis Enterprise](https://skygenesisenterprise.com). Links to external services are provided for convenience and do not imply endorsement. Check each service's own terms before relying on it in production.
