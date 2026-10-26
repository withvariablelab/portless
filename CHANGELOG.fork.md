# Changelog (fork)

Release notes for `@variablelab/portless`. Upstream's own history is in [CHANGELOG.md](./CHANGELOG.md).

## 0.0.6

<!-- release:start -->

Based on upstream [portless v0.15.11](https://github.com/vercel-labs/portless/releases/tag/v0.15.11), merged in [#10](https://github.com/withvariablelab/portless/pull/10). Releases up to 0.0.5 were based on v0.15.6.

### Changed

- **Upstream v0.15.7 to v0.15.11**: everything in upstream's [changelog](https://github.com/vercel-labs/portless/blob/v0.15.11/CHANGELOG.md) for these versions is in this release. Among others:
  - Closing the terminal stops the dev server, and `portless run` and `portless prune` stop dev servers left behind by a crashed session.
  - App ports are only assigned when they are free on every local address.
  - Child commands use the node a version manager put first on `PATH`.
  - Certificate fixes: the CA's Basic Constraints, two-label host certificates, clock skew, and certificates only for the configured TLDs.
  - The proxy stops forwarding client hop-by-hop headers, keeps reading the request body after a backend error, and answers non-browser clients with JSON errors.
  - `--cloudflare` shares an app through a Cloudflare quick tunnel. `portless list` and the 404 page show each route's uptime, and the 404 page can filter its app list.
- **Wildcard subdomains**: upstream now routes an unregistered subdomain to the closest registered parent too ([vercel-labs/portless#446](https://github.com/vercel-labs/portless/pull/446)). The fork builds on it and still picks the path prefix among that parent's routes only, so the behavior from 0.0.3 is unchanged.
- **Crashed-session cleanup with `--path`**: the process group portless records to stop an orphaned dev server is stored on the app's own route, the hostname and path prefix it registered, never on another app's route for the same hostname.

### Added

- `portless list --json` includes `cloudflareUrl`, with the route's path prefix, for apps shared with `--cloudflare`.

<!-- release:end -->

## 0.0.5

Based on upstream [portless v0.15.6](https://github.com/vercel-labs/portless/releases/tag/v0.15.6) (upstream `main` at `1ad573b`), the same base as 0.0.4.

### Changed

- The repository moved to [withvariablelab/portless](https://github.com/withvariablelab/portless). The package name and the CLI are unchanged.

## 0.0.4

Based on upstream [portless v0.15.6](https://github.com/vercel-labs/portless/releases/tag/v0.15.6) (upstream `main` at `1ad573b`), the same base as 0.0.3.

### Added

- **JSON output for scripts and agents**: `--json` on `portless list`, `portless get`, `portless doctor`, and `portless service status`. stdout carries only the JSON document, warnings and errors go to stderr, keys are camelCase, and other commands reject the flag. `list --json` returns the routes the proxy serves (live processes and aliases) with their public URLs, and exits 1 when `routes.json` cannot be read or parsed instead of printing an empty list. See the JSON output section of the README. Added in [#6](https://github.com/withvariablelab/portless/pull/6).

### Changed

- When `routes.json` cannot be read (for example because of file permissions), commands now print a warning on stderr instead of silently treating it as empty.

## 0.0.3

Based on upstream [portless v0.15.6](https://github.com/vercel-labs/portless/releases/tag/v0.15.6) (upstream `main` at `1ad573b`), the same base as 0.0.2.

### Fixed

- **Wildcard subdomains go to the closest registered parent**: with `--wildcard`, when several registered hostnames are parents of a request (for example `acme.localhost` and `feat-x.acme.localhost` for `tenant.feat-x.acme.localhost`), the closest one now serves it regardless of registration order, and a longer `--path` prefix on a farther parent no longer takes the request from it. If the closest parent has no route for the path, the proxy answers 404 instead of falling through to a farther parent. Upstream issue [vercel-labs/portless#380](https://github.com/vercel-labs/portless/issues/380), fixed in [#4](https://github.com/withvariablelab/portless/pull/4).

## 0.0.2

Based on upstream [portless v0.15.6](https://github.com/vercel-labs/portless/releases/tag/v0.15.6) (upstream `main` at `1ad573b`), unchanged since 0.0.1.

### Changed

- Releases are published through npm trusted publishing (GitHub OIDC) instead of a long-lived token. This release is the first to use it.
- Fixed the `contributors` entry in `package.json` so npm shows the maintainer's name and profile url correctly.

## 0.0.1

First release of the fork, based on upstream [portless v0.15.6](https://github.com/vercel-labs/portless/releases/tag/v0.15.6) (upstream `main` at `1ad573b`).

### Added

- **Path-based routing**: `--path <prefix>` on `portless run` and `portless <name>`, the `PORTLESS_PATH` environment variable, and the `path` field in `portless.json` and in the `"portless"` key of `package.json`. Several apps can share one hostname; the proxy dispatches by the longest prefix that matches at a `/` boundary and forwards the full path to the backend. `alias`, `get`, and `list` understand prefixes, and the 404 page lists them. Proposed upstream in [vercel-labs/portless#165](https://github.com/vercel-labs/portless/pull/165).

### Changed

- Published as `@variablelab/portless`. The installed command is still `portless`.
- The npm tarball now includes `LICENSE` and `NOTICE`.
