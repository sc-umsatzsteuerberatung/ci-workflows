# ci-workflows

Reusable GitHub Actions workflows shared across the firm's repositories.

## Workflows

### `sign-release.yml`

Signs the assets of a published GitHub release with [minisign](https://jedisct1.github.io/minisign/) and attaches the resulting `.minisig` files back to the same release.

**Why:** the desktop launcher and every app it manages verify a minisign signature on every download before extraction. The signing key lives only as encrypted secrets here in CI; even a fully compromised app server cannot forge an update.

**Required org-level secrets:**

| Secret | Contents |
|---|---|
| `MINISIGN_PRIVATE_KEY` | The full contents of the `minisign.key` file produced by `minisign -G`. |
| `MINISIGN_KEY_PASSWORD` | The password that protects the secret key. |

**Consume from a release workflow:**

```yaml
on:
  release:
    types: [published]

jobs:
  build:
    # ... your existing build job that uploads release assets ...

  sign:
    needs: build
    uses: <org>/ci-workflows/.github/workflows/sign-release.yml@v1
    with:
      artifact_glob: "*-{linux,windows}-*.{tar.gz,zip}"
      release_tag: ${{ github.event.release.tag_name }}
    secrets: inherit
```

**Pin the version.** Always reference a tag (`@v1`) or commit SHA — never `@main`. A compromise of this repository would otherwise propagate instantly to every consumer's signing pipeline.

## Releasing a new version of a workflow

1. Push the change to `main`.
2. Re-tag: `git tag -f v1 && git push -f origin v1` (for backwards-compatible changes).
3. For breaking changes, create a new major tag (`v2`) and migrate consumers.
