# Syncing with upstream

This fork tracks the releases of [vercel-labs/portless](https://github.com/vercel-labs/portless). Its own changes on top of upstream:

- `feat: add --path flag for path-based routing` (the feature, upstream [PR #165](https://github.com/vercel-labs/portless/pull/165))
- `fix(proxy): route wildcard subdomains to the closest registered parent` (upstream issue [#380](https://github.com/vercel-labs/portless/issues/380)): upstream's [#446](https://github.com/vercel-labs/portless/pull/446) picks the closest parent, and the fork keeps path selection inside it
- `feat(cli): add --json output to list, get, doctor, and service status`
- `fix(routes): warn when the routes file cannot be read`
- `chore: rename package to @variablelab/portless` (packaging)
- fork docs, `CHANGELOG.fork.md`, and the release workflow
- `chore: move the fork to withvariablelab` (repository URLs)

Everything else should be identical to upstream. Keep it that way: the smaller the fork's diff, the cheaper each sync.

## One-time setup

```bash
git clone git@github.com:withvariablelab/portless.git
cd portless
git remote add upstream https://github.com/vercel-labs/portless.git
git fetch upstream --tags
```

## Pull in upstream changes

Merge, do not rebase: `main` is public and its history must stay stable. Sync to an upstream release tag, so each fork release can name the upstream version it tracks.

```bash
git checkout main
git pull origin main
git fetch upstream --tags
git tag -l 'v*' --sort=-v:refname | head -1   # the latest upstream release, e.g. v0.15.11
git checkout -b sync/upstream-v0.15.11
git merge v0.15.11
```

Resolve conflicts (see below), then run the full pipeline before opening a PR:

```bash
pnpm install --frozen-lockfile
pnpm format:check && pnpm lint && pnpm type-check
pnpm build && pnpm test && pnpm test:e2e
pnpm --filter @portless/docs test:routes
```

Run it from a regular clone, not from a linked git worktree on a branch: portless prefixes hostnames with the worktree's branch name, so the CLI tests that expect a bare `myapp.localhost` fail there. CI is not affected.

Open a PR against `main`. CI runs the same steps. Merging the PR does not publish anything unless the version changed; see [RELEASING.md](./RELEASING.md).

## How PRs land

- **A sync PR lands as a merge commit**, never as a squash or a rebase. A squash folds upstream's commits into one fork commit and a rebase replays them under new SHAs: either way git no longer knows they came from upstream, the next sync's merge base stays behind, and every later sync conflicts on them again.
- **The fork's own features and fixes** (like `--path` and `--json`) **land as a squash**, one commit per change.

The reason is that `main` requires signed commits. GitHub signs the squash commit it creates, and the merge commit of a sync PR, while a rebase merge lands the branch's commits unsigned (`326f4c2`, `967f097` and `ba79a64` show as unsigned on GitHub for that reason). A merge commit brings the branch's commits into `main` as they are, so sign every commit you push to a sync branch; upstream's commits arrive already verified. This needs merge commits and squash merging allowed in the repository settings, and rebase merging off.

## Where conflicts show up

The fork touches a small set of files, so conflicts cluster there:

| File                                    | Why it conflicts                                                       | How to resolve                                                                                                                       |
| --------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `packages/portless/package.json`        | upstream bumps `version` on every release                              | keep the fork's `name`, `version`, and metadata; take upstream's dependencies                                                        |
| `packages/portless/src/cli.ts`          | flag parsing, help text, `runApp`, the tunnel table, and route helpers | keep both: upstream's change plus the `pathPrefix` plumbing; `--path` stays in the known-flag lists and `--json` in the global flags |
| `packages/portless/src/proxy.ts`        | `findRoute` and the 404 page                                           | keep upstream's matching and closest-parent pick, then the fork's longest-prefix selection among that hostname's routes              |
| `packages/portless/src/routes.ts`       | route identity is `(hostname, pathPrefix)` in the fork                 | thread `pathPrefix` through any new add, remove, or update helper                                                                    |
| `packages/portless/src/*.test.ts`       | both sides add tests next to each other                                | keep both; drop a fork test only when an upstream test now covers the same case                                                      |
| `README.md`, `skills/portless/SKILL.md` | upstream edits docs near the fork's sections                           | keep the fork banner, install commands, the path-based routing section, and the JSON output section                                  |
| `turbo.json`                            | `test:e2e` depends on `@variablelab/portless#build`                    | keep the scoped name                                                                                                                 |
| `.github/workflows/release.yml`         | the fork publishes a different package                                 | keep the fork's version                                                                                                              |

Some breaks are not conflicts, because upstream's new code merges cleanly next to the fork's. After resolving, check:

- every `store.addRoute`, `store.updateRoute`, `store.removeRoute`, and `removeRoutes` call in upstream's new code passes the route's `pathPrefix`. Without it the call misses a `--path` route, or changes the root route of the same hostname, which may belong to another app.
- a new route field (like `registeredAt`) or a new tunnel (like `--cloudflare`) also reaches the fork's `--json` output, its docs, and the fork tests that compare whole routes.

Do not modify `CHANGELOG.md` or `LICENSE`: both stay upstream's. The fork's release notes live in `CHANGELOG.fork.md`.

To see exactly what the fork changes relative to the upstream release it tracks:

```bash
git diff v0.15.11...main --stat
git diff v0.15.11...main -- packages/portless/src
```

## If upstream merges PR #165

```bash
gh pr view 165 --repo vercel-labs/portless --json state,mergedAt
```

If it was merged, the feature commit is redundant. Resolve the merge in favor of upstream for the feature files, keep only the packaging and docs commits, and consider deprecating the package on npm with a pointer to upstream.

## Rules

- Never force-push `main`.
- Never push to `upstream`. Changes for upstream go through PRs on vercel-labs/portless.
- Keep upstream's `LICENSE` and copyright intact (Apache-2.0, Vercel Inc.).
