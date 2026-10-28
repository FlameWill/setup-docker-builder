# FlameWill Docker builder

`docker/setup-buildx-action`, with a Docker layer cache on [FlameWill](https://flamewill.com) runners. Everywhere else it is exactly
`docker/setup-buildx-action` and the build runs uncached.

```yaml
steps:
  - uses: flamewill/setup-docker-builder@v1
  - uses: docker/build-push-action@v7
    with:
      push: true
      tags: ghcr.io/you/api:latest
```

Building more than one image? Give each its own `cache-key` (`with: cache-key: api`).

## How it works

- **The cache is the builder itself**: the builder's BuildKit state from the last default-branch build is back on the machine's
  local disk before the builder starts, so unchanged layers are simply there. A build needs no `cache-from` or `cache-to`, and
  spends no time importing or exporting a cache.
- **Only builds on the default branch save it** (push, schedule or manual run of the repository itself), after the job. FlameWill
  decides that from GitHub's own record of the run, so a pull request or another branch reads it but can never put a layer into it
  that a release build would reuse.
- **It never fails your build**: without a cache (another runner) the build runs as it would without it; a cache that cannot be
  saved is skipped.
- A cache keeps its most recently used layers within 10 GB, and one that no default-branch build has saved for 7 days is dropped.
- The `cache-from` and `cache-to` outputs stay, always empty, so workflows written for the earlier version keep working.

The cache needs BuildKit in a container, the builder this action sets up; its other inputs are `docker/setup-buildx-action`'s
(`version`, `driver-opts`, `buildkitd-flags`, `buildkitd-config`, `buildkitd-config-inline`, `platforms`, `name`, `cleanup`).
