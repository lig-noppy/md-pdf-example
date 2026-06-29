# Markdown -> PDF変換処理

## 主な機能

- markdownをファイル/フォルダ単位でPDFに変換する。
- 差分ビルド(2回目以降は変更差分のあるファイルのみ変換)
- `style.css`によるテーマCSS
- PDFリンクの相対パス化し、ブラウザでPDFを開いた際にPDF間をリンク移動可能な形式で出力
- インラインmermaidコードブロックの描画に対応

## コンテナイメージ

変換処理は Docker コンテナで実行する。イメージは別リポジトリでビルド・公開され、GHCR から取得する。

| 項目 | 値 |
|------|-----|
| イメージ | `ghcr.io/lig-noppy/md-pdf:latest` |
| 作業ディレクトリのマウント先 | コンテナ内 `/work` |

初回利用時、またはイメージ更新後は pull する。

```bash
docker pull ghcr.io/lig-noppy/md-pdf:latest
```

## 使い方

### Docker Compose（推奨）

`doc/spec-md/` フォルダで実行する。`compose.yaml` は上記 GHCR イメージを参照する。

```bash
cd doc/spec-md

# イメージ取得（初回または更新後）
docker compose pull

# 変換（全対象）
docker compose run --rm md-pdf

# オプション付き
docker compose run --rm md-pdf --no-cache
docker compose run --rm md-pdf --target 09_example/sample01.md
docker compose run --rm md-pdf --dry-run
```

Linux / WSL2 では出力ファイルの所有者を揃えるため、`.env` にホストの UID/GID を設定する（未設定時は `1000:1000`）。`.env.example` をコピーして利用できる。

```env
UID=1000
GID=1000
```

### docker run

`work/` ディレクトリを `/work` にマウントして実行する。`.env` は `--env-file` で渡し、コンテナ内ロケール（日本語フォントのフォールバック・漢字字形）を設定する。

```bash
cd doc/spec-md/

# 全ファイル変換
docker run --rm -u "$(id -u):$(id -g)" --env-file .env -v "$PWD/work":/work ghcr.io/lig-noppy/md-pdf:latest convert

# 単一ファイル指定
docker run --rm -u "$(id -u):$(id -g)" --env-file .env -v "$PWD/work":/work ghcr.io/lig-noppy/md-pdf:latest convert --target 09_example/sample01.md

# 変換対象の一覧 (PDF 生成は行わない)
docker run --rm -u "$(id -u):$(id -g)" --env-file .env -v "$PWD/work":/work ghcr.io/lig-noppy/md-pdf:latest convert --dry-run
```

## Options / Configs

オプション名について、コマンドオプションはkebab-case. configはsnake_caseである前提で記載する。
(ex: option:`--no-relative-link`, config:`relative_link`)

設定の優先度: CLI オプション > config.yaml > デフォルト

| CLI | config | 型 | デフォルト | 説明 |
|-----|--------|-----|-----------|------|
| `--target PATH` | `target` | list | 全 `.md` | 処理対象 (複数指定可) |
| `--ignore PATH` | `exclude` | list | `[]` | 除外対象 (複数指定可) |
| `--output DIR` | `output` | string | `output` | 出力フォルダ |
| `--no-recurse` | `recurse: false` | bool | `true` | サブディレクトリを処理しない |
| `--no-relative-link` | `relative_link: false` | bool | `true` (有効) | リンクの相対パス化を無効化 |
| `--no-readme-index` | `readme_index: false` | bool | `true` | README.md の index.pdf 化を無効化 |
| `--css FILE` | `css` | string | `style.css` | 適用するCSSテーマ |
| `--dry-run` | (なし) | flag | `false` | 対象一覧のみ表示 |
| `--no-cache` | (なし) | flag | `false` | 差分チェックをせず全対象を変換 |

## Sub Commands

| command | 説明 |
| :--- | :---|
| convert | markdownをPDFに変換する |

## ディレクトリ構成

```
.env.example            環境変数のサンプル (.env にコピーして利用)
compose.yaml            Docker Compose 定義 (GHCR イメージ参照)
work/                   Dockerコンテナにマウントするフォルダ
  .cache/               DLしたパッケージやライブラリ、中間ファイル等をキャッシュするフォルダ
  assets/               アセット系ファイルを格納するフォルダ
    fonts/              fontファイルの格納フォルダ. style.cssから `@font-face` 経由で適用参照される.
  markdown/             マークダウンファイルを格納するフォルダ
  output/               PDFファイルの出力先
    .meta/              差分処理などに必要なデータを格納
  config.yaml           実行時の設定を記載するファイル。ファイルがない場合はデフォルト値を利用。
  style.css             適用するCSSテーマ
```
