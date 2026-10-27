# FlameWill Docker builder

`docker/setup-buildx-action`, with a Docker layer cache on [FlameWill](https://flamewill.com) runners for `docker/build-push-action`.
Everywhere else it is exactly `docker/setup-buildx-action` and the build runs uncached.

```yaml
permissions:
  contents: read
  id-token: write   # lets builds of the default branch save the cache

steps:
  - uses: flamewill/setup-docker-builder@v1
    id: builder
    with:
      cache-key: api          # one key per image you build
  - uses: docker/build-push-action@v7
    with:
      push: true
      tags: ghcr.io/you/api:latest
      cache-from: ${{ steps.builder.outputs.cache-from }}
      cache-to: ${{ steps.builder.outputs.cache-to }}
```

## How it works

- **Every build reads the cache**: it waits on the machine's local disk before the build starts, and unchanged layers are reused
  instead of rebuilt.
- **Only builds on the default branch save it** (push, schedule or manual run), after the job, with `permissions: id-token: write`.
  A pull request or another branch reads it but can never put a layer into it that a release build would reuse.
- **It never fails your build**: without a cache (another runner) both outputs are empty and the build runs as it would without
  them; a cache that cannot be saved is skipped.
- A cache that no default-branch build has saved for 7 days is dropped.

The cache needs BuildKit in a container, the builder this action sets up; its other inputs are `docker/setup-buildx-action`'s
(`version`, `driver-opts`, `buildkitd-flags`, `buildkitd-config`, `buildkitd-config-inline`, `platforms`, `name`, `cleanup`).
