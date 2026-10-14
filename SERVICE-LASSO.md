# Fork-hosted server builds

`service-lasso/fork-nextcloud-server` is a fork of `nextcloud/server`. It hosts
the Service Lasso build pipeline for a **runnable** Nextcloud server archive.

Why a fork is needed:

- Nextcloud's official release archives bundle the `3rdparty` composer
  dependencies, but `3rdparty` is a **git submodule** in the source repository,
  so a plain GitHub tag tarball is not runnable.
- Windows needs small compatibility patches (migration discovery path
  separators and drive-qualified data-directory validation). Applying them in
  the fork keeps the packaged service reproducible and reviewable.

## Branch model

- Upstream branches and the `v*` tags stay a **pristine mirror**. Do not commit
  Service Lasso changes there.
- **All Service Lasso branches use the `servicelasso/` prefix** so they never
  collide with upstream. Examples: `servicelasso/master`,
  `servicelasso/nextcloud-34`, `servicelasso/feat/fix-webdav`.
- Version branches are created from the matching upstream tag (for example
  `servicelasso/nextcloud-34` from `v34.0.4`); `servicelasso/master` tracks
  upstream `master` and is the integration branch for Service Lasso work.
- This directory holds the reviewable source of the workflow and notes that are
  landed on those branches.

## Build and release

`service-lasso-build-server.yml` (triggers on `servicelasso/**` and
`workflow_dispatch`):

1. checks out the branch with `submodules: recursive` (fetches `nextcloud/3rdparty`);
2. applies the two documented Windows compatibility patches at build time;
3. stages the working tree as `nextcloud/` and produces
   `nextcloud-<version>-service-lasso.zip` plus a SHA-256;
4. uploads the archive as a workflow artifact, and on a `publish=true`
   `workflow_dispatch` publishes it as a GitHub release.

## Consuming the fork archive

Set these in the Service Lasso release environment for `lasso-nextcloud`:

```text
NEXTCLOUD_FORK_SERVER_ARCHIVE=<url-or-local-path-to-nextcloud-<version>-service-lasso.zip>
NEXTCLOUD_FORK_SERVER_FORMAT=zip
```

`scripts/package.mjs` then packages the fork archive directly and skips the
in-repo compatibility patches (the archive already carries them), recording
`server.source = fork` in `SERVICE-LASSO-PACKAGE.json`. Without
`NEXTCLOUD_FORK_SERVER_ARCHIVE`, packaging falls back to the upstream release
plus the guarded in-repo patches.
