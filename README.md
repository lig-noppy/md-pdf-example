# markdown仕様書を追加したプロジェクト構成サンプル

## Usage

```bash
cd doc/spec-md

# 事前準備 (初回のみ)
```
# Linux / WSL2:
cp .env.example.linux .env

# Windows:
cp .env.example.win .env
```

# 事前準備2 (optional)
cp work/style.example.css work/style.css
vi work/style.css

# md -> pdf変換
docker compose run --rm md-pdf convert
```

markdown -> PDF 変換処理の詳細 / オプション等は [Markdown -> PDF変換処理](./doc/spec-md/README.md) を参照.