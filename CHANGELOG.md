# Changelog / 変更履歴

## Unreleased

## 1.0.0
- (JA) tools/sync_release_assets.pyを追加し、親ディレクトリ配下のリポジトリでbump_version.pyとrelease.ymlの両方がある場合のみ同期するように対応
- (EN) Added tools/sync_release_assets.py to sync bump_version.py and release.yml to sibling repos only when both files exist
- (JA) README.mdとREADME.ja.mdを追加し、英日版で相互リンクするように更新
- (EN) Added README.md and README.ja.md with cross-links between English and Japanese versions

## 0.0.4
- (JA) bump_version.pyでCHANGELOG.mdを自動整形し、新バージョンのセクションを追加するように変更
- (EN) Added changelog auto-update in bump_version.py to insert a new version section

## 0.0.3
- (JA) リリース時にsketch.yamlの`dir`をライブラリ名(バージョン)へ置換し、Gitにはステージしないように変更
- (EN) Updated release flow to swap `dir` entries in sketch.yaml to library name (version) without staging them in Git
- (JA) 初期コミット
- (EN) Initial commit
