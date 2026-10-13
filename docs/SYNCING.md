# Syncing with upstream

This fork tracks [vercel-labs/portless](https://github.com/vercel-labs/portless). Its own commits sit on top of upstream `main`:

- `feat: add --path flag for path-based routing` (the feature, upstream [PR #165](https://github.com/vercel-labs/portless/pull/165))
- `fix(proxy): route wildcard subdomains to the closest registered parent` (upstream issue [#380](https://github.com/vercel-labs/portless/issues/380), extended to path prefixes)
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

Merge, do not rebase: `main` is public and its history must stay stable.

```bash
git checkout main
git pull origin main
git fetch upstream --tags
git checkout -b sync/upstream-$(date +%Y%m%d)
git merge upstream/main
```

Resolve conflicts (see below), then run the full pipeline before opening a PR:

```bash
pnpm install --frozen-lockfile
pnpm format:check && pnpm lint && pnpm type-check
pnpm build && pnpm test && pnpm test:e2e
```

Open a PR against `main`. CI runs the same steps. Merging the PR does not publish anything unless the version changed; see [RELEASING.md](./RELEASING.md).

## Where conflicts show up

The fork touches a small set of files, so conflicts cluster there:

| File                                    | Why it conflicts                                       | How to resolve                                                                                                                |
| --------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `packages/portless/package.json`        | upstream bumps `version` on every release              | keep the fork's `name`, `version`, and metadata; take upstream's dependencies                                                 |
| `packages/portless/src/cli.ts`          | flag parsing, help text, `runApp` and route helpers    | keep both: upstream's change plus the `pathPrefix` plumbing                                                                   |
| `packages/portless/src/proxy.ts`        | `findRoute`                                            | keep the longest-prefix selection inside whatever matching upstream has, and pick the closest wildcard parent before the path |
| `packages/portless/src/routes.ts`       | route identity is `(hostname, pathPrefix)` in the fork | thread `pathPrefix` through any new add, remove, or update helper                                                             |
| `README.md`, `skills/portless/SKILL.md` | upstream edits docs near the fork's sections           | keep the fork banner, install commands, and the path-based routing section                                                    |
| `turbo.json`                            | `test:e2e` depends on `@variablelab/portless#build`    | keep the scoped name                                                                                                          |
| `.github/workflows/release.yml`         | the fork publishes a different package                 | keep the fork's version                                                                                                       |

Do not modify `CHANGELOG.md` or `LICENSE`: both stay upstream's. The fork's release notes live in `CHANGELOG.fork.md`.

To see exactly what the fork changes relative to upstream at any time:

```bash
git diff upstream/main...main --stat
git diff upstream/main...main -- packages/portless/src
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
