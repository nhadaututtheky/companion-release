# Companion Releases

Download the latest Companion desktop app from [Releases](https://github.com/nhadaututtheky/companion-release/releases).

More info: https://companion.theio.vn

---

## Why this repo exists

Source code lives in the private `nhadaututtheky/companion` repo, but all CI
runs here. Public repos get unlimited free Actions minutes, private repos
don't — so anything that builds/tests/publishes the product runs from this
repo by pulling source at a pinned ref.

### Workflows

| Workflow | Purpose | Trigger |
|---|---|---|
| `tauri-build.yml` | Build Windows/macOS/Linux desktop installers | `workflow_dispatch { tag }` |
| `ci-quality.yml` | Lint, typecheck, tests, web build | `workflow_dispatch { ref? }` |
| `docker-publish.yml` | Build + push server Docker image to GHCR | `workflow_dispatch { ref?, image_tag? }` |

### Required secrets

- `SOURCE_REPO_TOKEN` — fine-grained PAT with `contents: read` on `nhadaututtheky/companion`. Used to check out source into each workflow.
- `GHCR_PUBLISH_TOKEN` — classic PAT with `write:packages, read:packages`. Used by `docker-publish.yml` to push `ghcr.io/nhadaututtheky/companion` (the image kept its original path for backwards compatibility with existing `docker-compose` consumers).
- `TAURI_SIGNING_PRIVATE_KEY` + `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` — Tauri updater signing key. Used by `tauri-build.yml`.

### Typical ship flow

```bash
# 1. Tag + release from the source repo (D:\Project\Companion) via /ship skill
git tag v0.X.Y
git push origin v0.X.Y

# 2. Trigger each build here
gh workflow run "CI Quality Gates"          --repo nhadaututtheky/companion-release --field ref=v0.X.Y
gh workflow run "Tauri Desktop Build"       --repo nhadaututtheky/companion-release --field tag=v0.X.Y
gh workflow run "Build & Publish Docker Image" --repo nhadaututtheky/companion-release --field ref=v0.X.Y
```
