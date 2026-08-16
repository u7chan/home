# Pi で OpenCode Go の Anthropic Messages 互換エンドポイントを利用する

Last reviewed: 2026-08-16

## 概要

Pi の built-in `opencode-go` provider から、OpenCode Go の Anthropic Messages 互換エンドポイントを利用する手順と検証結果をまとめる。

今回の検証対象は `minimax-m3` である。OpenAI Chat Completions 互換エンドポイントの検証は [Issue #59](https://github.com/u7chan/workstation-notes/issues/59) で扱う。

## 前提条件

- OpenCode Go のサブスクリプションを有効化している
- OpenCode Zen で OpenCode Go 用の API key を発行済み
- Pi 0.56.0 以降を利用する
- API key は環境変数または Pi の `auth.json` に設定する
- API key をリポジトリ、Issue、ログ、スクリーンショットへ保存しない

Pi 0.56.0 で OpenCode Go provider のサポートが追加された。現在の Pi では provider ID は `opencode-go`、環境変数は `OPENCODE_API_KEY` である。

## OpenCode Go 側の情報

OpenCode Go の公式ドキュメントで、モデル一覧、エンドポイント、利用制限を確認できる。

- [OpenCode Go 公式ドキュメント](https://opencode.ai/docs/ja/go/)
- [モデル一覧エンドポイント](https://opencode.ai/zen/go/v1/models)

今回の検証対象：

| 項目 | 値 |
| --- | --- |
| Pi provider | `opencode-go` |
| Pi model | `opencode-go/minimax-m3` |
| API 形式 | Anthropic Messages 互換 |
| Pi の model 定義 | `api=anthropic-messages` |
| base URL | `https://opencode.ai/zen/go` |
| POST エンドポイント | `https://opencode.ai/zen/go/v1/messages` |

Pi の Anthropic SDK はモデルの base URL に `/v1/messages` を付加してリクエストする。モデル一覧と model ID は変更される可能性があるため、実行時に Pi の表示を優先する。

## Pi 側の認証設定

### 方法 1: 環境変数で一時的に設定する

API key をシェルの環境変数に設定して、対象モデルを指定して Pi を起動する。

```bash
export OPENCODE_API_KEY='ここに OpenCode Go の API key'
pi --model opencode-go/minimax-m3
```

API key をシェル履歴やプロセス一覧に残したくない場合は、シェルの secret input 機能や環境変数管理ツールを使う。実際の key をコマンド例やリポジトリへ記録しない。

認証状態は key の値を表示せずに確認できる。

```bash
pi auth check --provider opencode-go --no-refresh
```

### 方法 2: `auth.json` に保存する

Pi の標準設定ディレクトリは `~/.pi/agent` である。`auth.json` は所有者だけが読み書きできる権限にする。

```json
{
  "opencode-go": {
    "type": "api_key",
    "key": "ここに OpenCode Go の API key"
  }
}
```

```bash
chmod 600 ~/.pi/agent/auth.json
pi auth check --provider opencode-go --no-refresh
```

Pi では、`auth.json` に保存した credential が環境変数より優先される。環境変数と `auth.json` に異なる key を設定した場合は、どちらが使われているかを確認する。

今回の実機検証では、Issue #59 の検証で `/login` から `OpenCode Go` を選び、保存確認済みの credential を再利用した。Anthropic モデル選択後に `/login` と再起動を独立して再実施したわけではないため、その条件での保存テストは未確認として残す。

## モデル選択

モデル一覧は provider ID の部分一致で確認する。

```bash
pi --list-models opencode-go
```

起動済みの Pi では、次のコマンドからモデルを選択できる。

```text
/model
```

一覧から `opencode-go/minimax-m3` を選ぶ。コマンドラインで明示する場合は次のとおりである。

```bash
pi --model opencode-go/minimax-m3
```

## Anthropic Messages 互換の注意点

### 認証ヘッダー

Pi 0.84.2 の built-in Anthropic Messages 実装は、API key を Anthropic SDK に渡す。SDK の実装上、主な認証・バージョンヘッダーは次のとおりである。

| ヘッダー | 用途 |
| --- | --- |
| `X-Api-Key` | API key 認証 |
| `anthropic-version: 2023-06-01` | Anthropic Messages API のバージョン指定 |

この検証ではネットワーク上の生ヘッダーを採取していないため、上表は Pi と SDK の実装確認に基づく記録である。

### ストリーミング

Pi 0.84.2 の Anthropic Messages 実装は、リクエストに `stream: true` を設定し、SSE のイベントを処理する。`minimax-m3` からの応答取得には成功したが、ネットワーク上の SSE イベントを個別に保存したわけではない。

### reasoning / thinking

Pi の built-in model 定義では `minimax-m3` は reasoning 対応として扱われ、実行時のステータスバーには thinking level `high` が表示された。一方、今回のターミナル表示では独立した reasoning / thinking 内容は観測しなかった。モデル定義上の対応と、UI 上の思考内容の表示は別に扱う。

## 疎通確認

### 1. テキスト応答

セッションを保存せず、短い prompt で応答を確認する。

```bash
pi --no-session \
  --model opencode-go/minimax-m3 \
    -p 'Anthropic Messages 互換エンドポイントの疎通確認です。OK とだけ返してください。'
```

記録する項目:

- 実行日時
- `pi --version` の結果
- provider と model ID
- 成功 / 失敗
- 失敗した場合の HTTP status とエラーメッセージ

### 2. 読み取り専用 tool call

書き込みを避けるため、一時ディレクトリで実行する。一時ディレクトリの準備自体を除き、ディレクトリ内では `pwd` と `ls -la` などの読み取り専用操作だけを行う。

```bash
probe_dir="$(mktemp -d)"
(
  cd "$probe_dir"
  pi --no-session \
    --model opencode-go/minimax-m3 \
    -p 'このディレクトリで pwd と読み取り専用のファイル一覧の確認を行い、結果を短く報告してください。ファイルの作成・変更・削除・ネットワークアクセスはしないでください。'
)
```

確認する項目:

- Pi が tool call を発行したか
- tool call が読み取り専用操作で完了したか
- モデルが tool の結果を受け取って応答したか
- tool call 中にエラーや無限リトライが発生しなかったか

## 利用状況（usage）の確認

OpenCode の利用履歴（usage）で、実行前後の利用状況を確認する。

記録する項目:

- 実行前の usage 表示
- 実行日時
- 使用したモデル ID
- 実行後の usage 表示
- 表示された枠、入力、出力、表示コスト

利用履歴への反映に時間がかかる場合があるため、API 応答の成功とサブスクリプション枠への計上を分けて記録する。数値の変化を確認できない場合は「計上未確認」と明記する。

## 検証結果

### 環境確認

| 項目 | 結果 |
| --- | --- |
| Pi バージョン | 0.84.2（確認済み） |
| built-in provider | `opencode-go`（確認済み） |
| 対象モデル | `minimax-m3`（確認済み） |
| API 形式 | `anthropic-messages`（Pi の built-in 定義で確認） |
| base URL | `https://opencode.ai/zen/go`（Pi の built-in 定義で確認） |
| POST エンドポイント | `https://opencode.ai/zen/go/v1/messages`（SDK の呼び出し経路から確認） |
| 認証 | Issue #59 で保存確認済みの `auth.json` credential を再利用 |

### テキスト応答の実行結果

Pi の `/model` で `minimax-m3` を選び、`hello` を送信した。

- Pi のステータス表示: `(opencode-go) minimax-m3`
- モデルからの応答取得: 成功
- 判定: Pi の built-in `opencode-go` provider 経由で Anthropic Messages 互換モデルを呼び出せることを確認

### 読み取り専用 tool call の実行結果

Herdr 経由で隣の Pi に依頼した。最初の試行では作業ディレクトリ上で追加の読み取り操作が実行されたが、リポジトリへの変更はなかった。その後、条件を明確にして一時ディレクトリで次の操作だけを実行し、成功した。

```bash
probe_dir=$(mktemp -d)
(cd "$probe_dir" && pwd && ls -la)
```

- `pwd`: `/tmp/tmp.SyUbJo4g5L`
- `ls -la`: `.` と `..` のみ
- tool call: 成功
- ファイルの作成・変更・削除、ネットワーク操作: なし
- モデルの応答: tool の結果を受けた報告を確認

### reasoning / thinking の実行結果

- ステータスバーに thinking level `high` が表示された
- 独立した reasoning / thinking 内容の表示は観測しなかった
- 本文では、reasoning 対応の実装確認と、thinking 内容の表示未確認を分けて扱う

### OpenCode Go の利用履歴（usage）

OpenCode の利用履歴で、セッション `e-39d7ad` に紐づく `minimax-m3` の6件を確認した。6件すべてが `Go` 枠だった。

| 時刻 | モデル | 入力 | 出力 | 表示された枠 | 表示コスト |
| --- | --- | ---: | ---: | --- | ---: |
| 8月16日 15:55 | `minimax-m3` | 2,213 | 34 | Go | $0.0006 |
| 8月16日 15:56 | `minimax-m3` | 2,368 | 853 | Go | $0.0017 |
| 8月16日 15:56 | `minimax-m3` | 3,436 | 40 | Go | $0.0003 |
| 8月16日 15:56 | `minimax-m3` | 3,986 | 620 | Go | $0.0011 |
| 8月16日 15:56 | `minimax-m3` | 4,787 | 140 | Go | $0.0005 |
| 8月16日 15:57 | `minimax-m3` | 5,002 | 123 | Go | $0.0005 |

表示上の合計は、入力 21,792、出力 1,810、コスト $0.0047 だった。

### 認証の確認結果と未確認事項

今回の実機検証では、Issue #59 の検証で `/login` により保存確認済みの `auth.json` credential を引き継いだ。`minimax-m3` 選択後に `/login` と再起動を独立して再実施したわけではないため、Anthropic モデル選択を起点にした保存テストは未確認として残す。

## 制約・注意点

- OpenCode Go のモデル一覧、model ID、利用制限は変更される可能性がある
- `auth.json` と環境変数に異なる credential がある場合、Pi は `auth.json` を優先する
- API key を Git 管理、Issue、ログ、スクリーンショットへ含めない
- built-in provider が正常に動作する場合、custom provider の設定は不要
- 生の認証ヘッダー、SSE イベント、thinking 内容は今回の実機検証では個別に採取していない
- Issue #59 で確認済みの認証保存結果を引き継いだため、Anthropic モデル選択後の独立した再起動テストは未確認である

## 参考

- [OpenCode Go](https://opencode.ai/docs/ja/go/)
- [Pi Providers](https://pi.dev/docs/latest/providers)
- [Pi 0.56.0 release](https://pi.dev/news/releases/0.56.0)
- [Issue #61](https://github.com/u7chan/workstation-notes/issues/61)
