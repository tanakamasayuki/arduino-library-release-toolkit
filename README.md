# Arduino Library Release Toolkit

[日本語](README.ja.md)

## Overview
Tools and workflows to automate Arduino library releases: bump versions, rewrite example sketches to use the released library, update changelogs, and publish GitHub releases with the right artifacts.

## Features
- `tools/bump_version.py` updates `library.properties`, moves `CHANGELOG.md` items from `## Unreleased` into a new `## <version>` section, rewrites `examples/**/sketch.yaml` entries so `dir: ...` becomes `<LibraryName> (<version>)`, and generates `src/<library>_version.h`.
- Release workflow (`.github/workflows/release.yml`) bumps the version on the repository's default branch, pushes it, then recreates a `release` branch where the rewritten `sketch.yaml` files are committed and `tests/` is removed so tags and ZIPs include only release-ready files.
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
- Trigger the GitHub Actions workflow `Release` (workflow_dispatch). It bumps the version and commits staged files on the default branch, pushes that branch, recreates the `release` branch with committed `sketch.yaml` rewrites and without `tests/`, builds a ZIP from that branch excluding `tests/`, tags it, pushes the branch/tag, and publishes a GitHub release.

### Project-specific release hooks

> This section defines the release hook specification. The workflow integration that invokes these hooks has not been implemented yet.

Each project may provide either of the following Python scripts. A missing hook is skipped, preserving the current release behavior.

```text
tools/release_hooks/
  pre_version_commit.py
  pre_release_commit.py
```

#### `pre_version_commit.py`

Runs after the version bump and immediately before the release commit on the default branch. This hook performs final checks against the development state and records information that should be preserved in the repository at release time.

Typical uses:

- Build verification using the new release version
- Generating and preserving map files, memory usage reports, or other release-time information
- Updating project-specific version files
- Other validation or generation steps

The hook must explicitly run `git add` for generated or updated files that should be included in the release commit. Avoid `git add -A`, because `examples/**/sketch.yaml` changes are intentionally not committed to the default branch.

#### `pre_release_commit.py`

Runs on the `release` branch after the standard transformations (`tests/` removal and `sketch.yaml` staging) and immediately before the release-only commit. This hook performs the final project-specific adjustment of the contents used for the tag and ZIP.

Typical uses:

- Removing large or development-only files that should not be distributed
- Adding release-only files or documentation
- Replacing configuration files with distribution-specific versions
- Validating the final distribution contents

The hook must run `git add` for files it adds or updates and `git rm` for tracked files it removes from the distribution. The shared workflow performs no additional staging after this hook and commits the resulting index as the release-only commit.

#### Common behavior

- Hooks are optional; only hooks that exist are invoked.
- A non-zero exit status aborts the release before the commit or publication.
- Hooks may generate or validate files and run `git add` or `git rm`; the shared workflow remains responsible for `git commit`, `git push`, tagging, and publication.
- Release information such as `RELEASE_VERSION`, `RELEASE_TAG`, `RELEASE_LEVEL`, and `RELEASE_PHASE` is passed to hooks through environment variables.
- Hook scripts are project-specific and are not copied by `tools/sync_release_assets.py`.

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
- Example `sketch.yaml` files keep relative `dir:` paths on the default branch, but the workflow rewrites and commits them on the `release` branch so tags/releases match the release-ready artifacts while keeping local development pointed at `src/`.
