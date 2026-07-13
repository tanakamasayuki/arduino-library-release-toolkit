# Arduino Library Release Toolkit

[English](README.md)

## 概要
Arduinoライブラリのリリースを自動化するためのツールとワークフローです。バージョン更新、サンプルsketchの書き換え、チェンジログ更新、GitHubリリース作成までをまとめて行います。

## 特長
- `tools/bump_version.py` が `library.properties` を更新し、`CHANGELOG.md` の `## Unreleased` 配下の項目を新しい `## <version>` セクションへ移動し、`examples/**/sketch.yaml` の `dir: ...` を `<ライブラリ名> (<version>)` に書き換え、`src/<library>_version.h` を生成します。
- リリースワークフロー（`.github/workflows/release.yml`）はリポジトリのデフォルトブランチでバージョンを上げてコミット・プッシュし、書き換え済み `sketch.yaml` をコミットして `tests/` を削除した `release` ブランチを作り直し、そのブランチからタグとZIPを作成します。
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
- GitHub Actions の `Release`（workflow_dispatch）を起動します。バージョンを上げてデフォルトブランチにコミット/プッシュし、`release` ブランチを作り直して `sketch.yaml` の書き換えをコミットし、`tests/` を除外した状態でZIP・タグ・GitHubリリースを作成します。

### プロジェクト固有のリリースフック

各プロジェクトは、必要に応じて次のPythonスクリプトを配置できます。ファイルがなければスキップし、現在と同じリリース処理を継続します。

```text
tools/release_hooks/
  pre_bump.py
  pre_version_commit.py
  pre_release_commit.py
```

#### `pre_bump.py`

バージョン関連ファイルを変更する前に実行します。特に、バージョン更新時に書き換えられる`examples/**/sketch.yaml`を含め、変更前の開発状態に対して検証や保全処理を行うためのフックです。

主な用途:

- バージョン更新前のソースとサンプルを使ったビルドチェック
- 変更前のファイルや生成物の保全
- リリースを開始できる状態かどうかの検証

この時点ではファイルをステージする共通処理はまだ始まっていません。検証に失敗した場合は非ゼロで終了し、バージョン更新を中止します。

#### `pre_version_commit.py`

バージョン更新後、デフォルトブランチのリリースコミット直前に実行します。開発状態を対象としたリリース前の最終確認や、リリース時点の情報をリポジトリに保存するためのフックです。

主な用途:

- 新しいリリースバージョンでのビルドチェック
- mapファイルやメモリ使用量など、リリース時点の情報の生成・保存
- プロジェクト固有のバージョンファイルの更新
- その他の検証や生成処理

フックが生成・更新したファイルをリリースコミットへ含める場合は、フック自身が対象を明示して `git add` します。`examples/**/sketch.yaml` はデフォルトブランチにコミットしないため、意図せずステージしないよう `git add -A` は避けてください。

#### `pre_release_commit.py`

`release` ブランチで共通の加工（`tests/` の削除と `sketch.yaml` のステージ）が終わった後、配布用コミットの直前に実行します。実際にタグとZIPの元になる配布内容を、プロジェクト固有の要件に合わせて最終調整するためのフックです。

主な用途:

- リリースに不要な大容量ファイルや開発専用ファイルの削除
- 配布専用ファイルやドキュメントの追加
- 設定ファイルの配布用への差し替え
- 配布内容の最終検証

追加・更新するファイルはフック自身が `git add` し、追跡済みファイルを配布対象から外す場合は `git rm` します。共通ワークフローはフック実行後に追加のステージ処理を行わず、ステージ済みの内容を配布用コミットにします。

#### 共通動作

- フックは任意で、存在するものだけを実行します。
- フックが非ゼロで終了した場合は、コミットや公開を行わずリリースを中止します。
- フックは `git add`、`git rm`、ファイル生成、検証を担当できますが、`git commit`、`git push`、タグ作成は共通ワークフローが担当します。
- `RELEASE_OLD_VERSION`、`RELEASE_VERSION`、`RELEASE_TAG`、`RELEASE_LEVEL`、`RELEASE_PHASE`を環境変数でフックへ渡します。`RELEASE_VERSION`は更新後のバージョンで、`pre_bump.py`でも予定バージョンとして参照できます。
- ワークフローのログには、旧バージョンから新バージョンへの変更内容を出力します。
- フックはプロジェクト固有ファイルであり、`tools/sync_release_assets.py` の同期対象には含めません。

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
- `sketch.yaml` はGit上ではデフォルトブランチで相対 `dir:` を維持し、ワークフローが `release` ブランチ上で `<ライブラリ名> (<version>)` に書き換えてコミットします。ローカルは `src/` を参照しつつ、タグ/リリースはリリース用の内容を指すようにしています。
