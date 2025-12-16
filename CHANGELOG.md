# Changelog / 変更履歴

## Unreleased

## 1.0.3
- (EN) Ensure top-level `examples/sketch.yaml` is also committed on the release branch so it ships in the ZIP/tag
- (JA) `examples/sketch.yaml` もリリースブランチでコミットし、ZIP/タグに確実に含めるように対応

## 1.0.2
- (EN) Release workflow now rebuilds the `release` branch and tags it so rewritten sketch.yaml files are part of the tagged release contents
- (JA) リリースワークフローで`release`ブランチを作り直し、書き換え済みsketch.yamlをタグの内容に含めるように変更

## 1.0.0
- (EN) Added tools/sync_release_assets.py to sync bump_version.py and release.yml to sibling repos only when both files exist
- (JA) tools/sync_release_assets.pyを追加し、親ディレクトリ配下のリポジトリでbump_version.pyとrelease.ymlの両方がある場合のみ同期するように対応
- (EN) Added README.md and README.ja.md with cross-links between English and Japanese versions
- (JA) README.mdとREADME.ja.mdを追加し、英日版で相互リンクするように更新

## 0.0.4
- (EN) Added changelog auto-update in bump_version.py to insert a new version section
- (JA) bump_version.pyでCHANGELOG.mdを自動整形し、新バージョンのセクションを追加するように変更

## 0.0.3
- (EN) Updated release flow to swap `dir` entries in sketch.yaml to library name (version) without staging them in Git
- (JA) リリース時にsketch.yamlの`dir`をライブラリ名(バージョン)へ置換し、Gitにはステージしないように変更
- (EN) Initial commit
- (JA) 初期コミット
