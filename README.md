# Docker Compose Wait Test - エラー再現リポジトリ

このリポジトリは、`docker compose up --wait`がexit code 1でエラーになる問題を再現するためのテストケースです。

## 問題の再現

### 構成

- `base-service`: `sleep infinity`で無限に実行されるサービス
- `test-service`: `base-service`に依存し、`WAIT_TIME`環境変数の値だけsleepするサービス（デフォルト: 0秒）

### エラーの再現方法

```bash
# デフォルト（WAIT_TIME=0）の場合 - exit code 1でエラーになる
docker compose up --wait

# または明示的にWAIT_TIME=0を指定
WAIT_TIME=0 docker compose up --wait
```

これらのコマンドを実行すると、**exit code 1でエラーになります**。

### 正常終了させる方法

```bash
# WAIT_TIME > 0 を指定すると exit code 0 で正常終了する
WAIT_TIME=2 docker compose up --wait
```

`WAIT_TIME`に0より大きい値を設定すると、`test-service`が指定秒数sleepした後に正常終了し、`docker compose up --wait`もexit code 0で終了します。

### エラーが発生する理由

`docker compose up --wait`は、**すべてのサービス**が正常に終了するか、ヘルスチェックが成功して安定状態になるまで待機します。

#### WAIT_TIME=0の場合（エラーになる）
1. `test-service`が即座に終了（sleep 0）
2. `base-service`は`sleep infinity`で永続的に実行され続ける
3. `base-service`にはヘルスチェックが設定されていないため、"healthy"状態にならない
4. 全体としてexit code 1でエラーとなる

#### WAIT_TIME > 0の場合（正常終了する）
1. `test-service`が指定秒数sleepしてから終了
2. `base-service`は引き続き実行されるが、`test-service`の終了により全体が正常終了扱いになる

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