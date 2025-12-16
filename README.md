# Arduino Library Release Toolkit

[日本語](README.ja.md)

## Overview
Tools and workflows to automate Arduino library releases: bump versions, rewrite example sketches to use the released library, update changelogs, and publish GitHub releases with the right artifacts.

## Features
- `tools/bump_version.py` updates `library.properties`, moves `CHANGELOG.md` items from `## Unreleased` into a new `## <version>` section, rewrites `examples/**/sketch.yaml` entries so `dir: ...` becomes `<LibraryName> (<version>)`, and generates `src/<library>_version.h`.
- Release workflow (`.github/workflows/release.yml`) bumps the version on `main`, pushes it, then recreates a `release` branch where the rewritten `sketch.yaml` files are committed so tags and ZIPs include the release-ready examples.
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
- Trigger the GitHub Actions workflow `Release` (workflow_dispatch). It bumps the version and commits staged files on `main`, pushes `main`, recreates the `release` branch with committed `sketch.yaml` rewrites, builds a ZIP from that branch, tags it, pushes the branch/tag, and publishes a GitHub release.

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
- Example `sketch.yaml` files keep relative `dir:` paths on `main`, but the workflow rewrites and commits them on the `release` branch so tags/releases match the release-ready artifacts while keeping local development pointed at `src/`.
