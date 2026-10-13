# Releasing @variablelab/portless

Releases are driven by the version in `packages/portless/package.json`, the same mechanism upstream uses. The workflow in `.github/workflows/release.yml` runs on every push to `main` and on manual dispatch:

1. It compares the `version` in `packages/portless/package.json` with the latest version on npm.
2. If they differ, it builds and runs `npm publish --provenance` from `packages/portless`.
3. It then creates the GitHub release `v<version>` with the notes found between `<!-- release:start -->` and `<!-- release:end -->` in `CHANGELOG.fork.md`.

Pushing a new version to `main` is what publishes. There is no separate approval step, so treat a merge to `main` that changes the version as a release.

## One-time setup

- Publishing uses [npm trusted publishing](https://docs.npmjs.com/trusted-publishers): the `@variablelab/portless` package on npm trusts the `release.yml` workflow of `withvariablelab/portless` running in the `Release` environment, and the job exchanges its GitHub OIDC token for a short-lived npm token. No long-lived token is stored.
- A trusted publisher cannot be edited, only revoked and created again. The old binding names `variableland/portless`, which no longer matches the workflow's OIDC claims, so replace it, revoking first because the `npm trust` docs say a new one fails while another exists (interactive 2FA, npm 11.15.0 or later):

  ```bash
  # 1. Find the id of the current binding (it names variableland/portless)
  npm trust list @variablelab/portless

  # 2. Remove it
  npm trust revoke --id <id> @variablelab/portless

  # 3. Trust the release workflow of the new repository
  npm trust github @variablelab/portless --file release.yml --repo withvariablelab/portless --env Release --allow-publish
  ```

  Cut a release right after step 3 (see below) to confirm the new binding works. Use the same three steps if the repository, workflow file, or environment ever changes again.

- The `Release` GitHub environment only accepts deployments from `main` (deployment branches: selected branches, `main`). The trusted publisher is bound to this environment, so a `release.yml` run dispatched from any other branch is rejected before it can request an npm token. Keep that restriction if you recreate the environment, and add required reviewers there if you want a manual gate before every publish.
- No npm token is stored anywhere. The `NPM_TOKEN` secret used for the first release (0.0.1) is gone from the repository; if a token from that release still exists on npmjs.com, revoke it.

## Cut a release

1. Branch from `main`: `git checkout -b release/v0.0.2`.
2. Bump `version` in `packages/portless/package.json`.
3. Add an entry at the top of `CHANGELOG.fork.md` wrapped in `<!-- release:start -->` and `<!-- release:end -->`, and remove the markers from the previous entry. Record which upstream version the release is based on.
4. Run `pnpm build && pnpm test`, then `cd packages/portless && npm publish --dry-run` to review the tarball.
5. Open a PR, get CI green, merge to `main`. The release workflow publishes and creates the tag and release.

`prepublishOnly` copies `README.md`, `LICENSE`, and `NOTICE` into `packages/portless/`. The copies are gitignored; delete them after a local dry run so they are not confused with the root files.

## Versioning

Fork versions are independent from upstream's. `0.0.1` is the first release, based on upstream `v0.15.6` plus path-based routing. Each changelog entry records the upstream version it tracks.
