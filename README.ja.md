# Arduino Library Release Toolkit

[English](README.md)

## 概要
Arduinoライブラリのリリースを自動化するためのツールとワークフローです。バージョン更新、サンプルsketchの書き換え、チェンジログ更新、GitHubリリース作成までをまとめて行います。

## 特長
- `tools/bump_version.py` が `library.properties` を更新し、`CHANGELOG.md` の `## Unreleased` 配下の項目を新しい `## <version>` セクションへ移動し、`examples/**/sketch.yaml` の `dir: ...` を `<ライブラリ名> (<version>)` に書き換え、`src/<library>_version.h` を生成します。
- リリースワークフロー（`.github/workflows/release.yml`）は `library.properties`、`CHANGELOG.md`、生成されたヘッダーをステージしますが、`sketch.yaml` はステージしません。リリースZIPは作業ツリーをもとに（`rsync` + `zip`）作られるため、未ステージのsketch書き換えも含まれます。
- `tools/sync_release_assets.py` は親ディレクトリを走査し、`tools/bump_version.py` と `.github/workflows/release.yml` の両方がある兄弟リポジトリに本リポジトリのファイルをコピーします。`--dry-run` と `--parent` に対応。

## 前提
- Python 3.11+（ワークフローと合わせています）
- Git

## 使い方

### ローカルでバージョンを上げる
- 変更せず次のバージョンを確認:
  ```sh
  python tools/bump_version.py --preview
  ```
- 実際に反映（`library.properties`、`CHANGELOG.md`、`examples/**/sketch.yaml` を更新し、`src/<library>_version.h` を生成）:
  ```sh
  python tools/bump_version.py --level patch  # minor/major も指定可
  ```

### リリースワークフローを実行する
- GitHub Actions の `Release`（workflow_dispatch）を起動します。バージョンを上げ、ステージ済みファイルをコミットし、作業ツリーからZIPを作成（未ステージのsketch書き換えも含む）、タグ付け・プッシュ・GitHubリリースを行います。

### 兄弟リポジトリへツールを同期する
- 対象確認のみ:
  ```sh
  python tools/sync_release_assets.py --dry-run
  ```
- 親ディレクトリ配下の兄弟リポジトリへコピー:
  ```sh
  python tools/sync_release_assets.py
  ```
- 走査ルートを指定:
  ```sh
  python tools/sync_release_assets.py --parent /path/to/parent
  ```

## メモ
- `sketch.yaml` はGit上では相対`dir:`を維持し、リリース時だけ `<ライブラリ名> (<version>)` に書き換えられます。ローカルは`src/`を参照し、リリースアーカイブでは公開ライブラリ版で動作させる意図です。
