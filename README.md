# Arduino Library Release Toolkit

[日本語](README.ja.md)

## Overview
Tools and workflows to automate Arduino library releases: bump versions, rewrite example sketches to use the released library, update changelogs, and publish GitHub releases with the right artifacts.

## Features
- `tools/bump_version.py` updates `library.properties`, moves `CHANGELOG.md` items from `## Unreleased` into a new `## <version>` section, rewrites `examples/**/sketch.yaml` entries so `dir: ...` becomes `<LibraryName> (<version>)`, and generates `src/<library>_version.h`.
- Release workflow (`.github/workflows/release.yml`) stages `library.properties`, `CHANGELOG.md`, and the generated header but intentionally leaves `sketch.yaml` unstaged; the release ZIP is built from the working tree (via `rsync` + `zip`) so the rewritten sketches are included even though they are not committed.
- `tools/sync_release_assets.py` scans the parent directory for sibling Git repos that contain both `tools/bump_version.py` and `.github/workflows/release.yml`, then copies this repo's versions over; supports `--dry-run` and `--parent`.

## Requirements
- Python 3.11+ (matches the workflow)
- Git

## Usage

### Bump the version locally
- Preview next version without modifying files:
  ```sh
  python tools/bump_version.py --preview
  ```
- Apply the bump (updates `library.properties`, `CHANGELOG.md`, `examples/**/sketch.yaml`, and writes `src/<library>_version.h`):
  ```sh
  python tools/bump_version.py --level patch  # or minor/major
  ```

### Run the release workflow
- Trigger the GitHub Actions workflow `Release` (workflow_dispatch). It bumps the version, commits staged files, builds a ZIP from the working tree so unstaged sketch edits are included, tags, pushes, and publishes a GitHub release.

### Sync release assets to sibling repos
- Show targets without copying:
  ```sh
  python tools/sync_release_assets.py --dry-run
  ```
- Copy to sibling repos under the parent directory:
  ```sh
  python tools/sync_release_assets.py
  ```
- Use another root:
  ```sh
  python tools/sync_release_assets.py --parent /path/to/parent
  ```

## Notes
- Example `sketch.yaml` files keep relative `dir:` paths in Git history but are rewritten to `<LibraryName> (<version>)` for releases. This allows local sketches to use `src/` while release artifacts use the published library version.
