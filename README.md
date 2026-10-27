# FlameWill Docker builder

`docker/setup-buildx-action`, with a Docker layer cache on [FlameWill](https://flamewill.com) runners for `docker/build-push-action`.
Everywhere else it is exactly `docker/setup-buildx-action` and the build runs uncached.

```yaml
permissions:
  contents: read
  id-token: write   # the cache's credentials

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

- **Every build reads the cache**: unchanged layers are reused instead of rebuilt.
- **Only builds on the default branch write it** (push, schedule or manual run). A pull request or another branch reads it but can
  never put a layer into it that a release build would reuse.
- **It never fails your build**: without a cache (another runner, or no `id-token: write`) both outputs are empty and the build runs
  as it would without them; a cache that cannot be written is skipped.
- Layers no build has used for 7 days are dropped.

The cache needs BuildKit in a container, the builder this action sets up; its other inputs are `docker/setup-buildx-action`'s
(`version`, `driver-opts`, `buildkitd-flags`, `buildkitd-config`, `buildkitd-config-inline`, `platforms`, `name`, `cleanup`).
