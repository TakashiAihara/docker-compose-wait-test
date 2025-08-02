# Docker Compose Wait Test - エラー再現リポジトリ

このリポジトリは、`docker compose up --wait`がexit code 1でエラーになる問題を再現するためのテストケースです。

## 問題の再現

### 構成

- `base-service`: `sleep infinity`で無限に実行されるサービス
- `test-service`: `base-service`に依存し、`exit 0`で正常終了するサービス

### エラーの再現方法

```bash
docker compose up --wait
```

このコマンドを実行すると、**必ずexit code 1でエラーになります**。

### エラーが発生する理由

`docker compose up --wait`は、**すべてのサービス**が正常に終了するか、ヘルスチェックが成功して安定状態になるまで待機します。しかし：

1. `base-service`は`sleep infinity`で永続的に実行されるため、終了しません
2. `base-service`にはヘルスチェックが設定されていないため、"healthy"状態になりません
3. `test-service`は正常に終了しますが、`base-service`が終了しないため、全体としてはエラーとなります

### 実行ログの確認

```bash
# バックグラウンドで実行してログを確認
docker compose up -d
docker compose logs

# または、フォアグラウンドで実行（Ctrl+Cで終了）
docker compose up
```

### クリーンアップ

```bash
docker compose down
```

## Docker バージョン

確認バージョン: Docker version 28.3.3, build 980b856