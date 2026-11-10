# Changelog

## 1.0.0

Initial release of **ZenthCloud Actions**, the official GitHub Actions integration for ZenthCloud, maintained by Sky Genesis Enterprise.

This project is derived from [`cloudflare/wrangler-action`](https://github.com/cloudflare/wrangler-action) (MIT). Changes in this release:

- Renamed the project identity to **ZenthCloud Actions** (`skygenesisenterprise/zenthcloud-actions`).
- Rewrote the action metadata (`action.yml`) around the ZenthCloud project surface.
- Renamed the action core (`wranglerAction.ts` → `zenthcloudAction.ts`, `wranglerArtifactManager.ts` → `artifactManager.ts`) and its public configuration type (`ZenthcloudActionConfig`).
- Reset the version numbering to an independent SemVer track starting at `v1`.
- Replaced the repository documentation, community files and CI with ZenthCloud equivalents.

The history of the code derived from the upstream project, up to its `4.1.3` release, is preserved in the [upstream changelog](https://github.com/cloudflare/wrangler-action/blob/main/CHANGELOG.md).
