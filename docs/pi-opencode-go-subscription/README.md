# Pi で OpenCode Go のサブスクリプション枠を利用する

Last reviewed: 2026-08-16

## 概要

Pi の built-in `opencode-go` provider を使い、OpenCode Go のサブスクリプション枠で複数の API 形式のモデルを利用する手順と検証結果をまとめる。

OpenAI Chat Completions 互換の `deepseek-v4-flash` は [Issue #59](https://github.com/u7chan/workstation-notes/issues/59)、Anthropic Messages 互換の `minimax-m3` は [Issue #61](https://github.com/u7chan/workstation-notes/issues/61) で検証した。

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

| API 形式 | モデル | base URL | POST エンドポイント | Issue |
| --- | --- | --- | --- | --- |
| OpenAI Chat Completions | `deepseek-v4-flash` | `https://opencode.ai/zen/go/v1` | `https://opencode.ai/zen/go/v1/chat/completions` | [#59](https://github.com/u7chan/workstation-notes/issues/59) |
| Anthropic Messages | `minimax-m3` | `https://opencode.ai/zen/go` | `https://opencode.ai/zen/go/v1/messages` | [#61](https://github.com/u7chan/workstation-notes/issues/61) |

モデル一覧と model ID は変更される可能性があるため、実行時に Pi の表示と公式エンドポイントの結果を確認する。

Issue #59 の検証では、2026-08-16 に `GET https://opencode.ai/zen/go/v1/models` を確認したところ、HTTP 200 が返り、26 モデルの中に `deepseek-v4-flash` が含まれていた。モデル一覧の確認自体には API key は要求されなかった。

## Pi 側の認証設定

### 方法 1: 環境変数で一時的に設定する

API key をシェルの環境変数に設定して、対象モデルを指定して Pi を起動する。

```bash
export OPENCODE_API_KEY='ここに OpenCode Go の API key'
pi --model opencode-go/deepseek-v4-flash
```

Anthropic Messages 互換モデルを確認する場合は、モデル ID を `opencode-go/minimax-m3` に置き換える。

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

実機検証では、`auth.json` を直接編集せず、Pi の `/login` で `Sign in with an API key` と `OpenCode Go` を選択して API key を登録した。これは Pi の認証フロー経由で `auth.json` に credential を保存する方法 2 に相当する。Issue #59 で保存確認した credential は、Issue #61 の `minimax-m3` 検証でも再利用した。

## モデル選択

モデル一覧は provider ID の部分一致で確認する。

```bash
pi --list-models opencode-go
```

起動済みの Pi では、次のコマンドからモデルを選択できる。

```text
/model
```

一覧から対象モデルを選ぶ。

| 検証対象 | 選択するモデル |
| --- | --- |
| OpenAI Chat Completions（#59） | `opencode-go/deepseek-v4-flash` |
| Anthropic Messages（#61） | `opencode-go/minimax-m3` |

コマンドラインで明示する場合は、次のように指定する。

```bash
pi --model opencode-go/deepseek-v4-flash
pi --model opencode-go/minimax-m3
```

## API 形式ごとの注意点

### OpenAI Chat Completions

`deepseek-v4-flash` は Pi 0.84.2 の built-in 定義で `api=openai-completions`、base URL は `https://opencode.ai/zen/go/v1` である。リクエスト先は `/chat/completions` になる。

### Anthropic Messages

`minimax-m3` は Pi 0.84.2 の built-in 定義で `api=anthropic-messages`、base URL は `https://opencode.ai/zen/go` である。Anthropic SDK が base URL に `/v1/messages` を付加してリクエストする。

#### 認証ヘッダー

Pi 0.84.2 の built-in Anthropic Messages 実装は API key を Anthropic SDK に渡す。SDK の実装上、主な認証・バージョンヘッダーは次のとおりである。

| ヘッダー | 用途 |
| --- | --- |
| `X-Api-Key` | API key 認証 |
| `anthropic-version: 2023-06-01` | Anthropic Messages API のバージョン指定 |

今回の検証ではネットワーク上の生ヘッダーを採取していないため、上表は Pi と SDK の実装確認に基づく記録である。

#### ストリーミング

Pi 0.84.2 の Anthropic Messages 実装は、リクエストに `stream: true` を設定し、SSE のイベントを処理する。`minimax-m3` からの応答取得には成功したが、ネットワーク上の SSE イベントを個別に保存したわけではない。

#### reasoning / thinking

Pi 0.84.2 の built-in model 定義では `minimax-m3` は reasoning 対応として扱われ、実行時のステータスバーには thinking level `high` が表示された。一方、今回のターミナル表示では独立した reasoning / thinking 内容は観測しなかった。モデル定義上の対応と、UI 上の思考内容の表示は別に扱う。

## 疎通確認

### テキスト応答

セッションを保存せず、対象モデルごとに短い prompt で応答を確認する。

```bash
pi --no-session \
  --model opencode-go/deepseek-v4-flash \
  -p 'OpenCode Go の疎通確認です。OK とだけ返してください。'

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

### 読み取り専用 tool call

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

OpenAI Chat Completions 互換モデルで確認する場合は、`--model` の値を `opencode-go/deepseek-v4-flash` に置き換える。

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

### Issue #59: OpenAI Chat Completions

#### 環境確認

| 項目 | 結果 |
| --- | --- |
| Pi バージョン | 0.84.2（確認済み） |
| built-in provider | `opencode-go`（確認済み） |
| 対象モデルの定義 | `deepseek-v4-flash` / `openai-completions` / `https://opencode.ai/zen/go/v1`（確認済み） |
| `/v1/models` エンドポイント | HTTP 200、26 モデル、`deepseek-v4-flash` を確認（2026-08-16） |
| `OPENCODE_API_KEY` | `/login` の API key 認証で設定（実機検証） |
| `auth.json` の `opencode-go` credential | Pi 再起動後も `OpenCode Go ✓ stored` を確認 |

#### 実機検証

| 項目 | 結果 |
| --- | --- |
| テキスト応答 | 成功（2026-08-16、`opencode-go` / `deepseek-v4-flash`） |
| 読み取り専用 tool call | 成功（`pwd` / `ls -la`、2026-08-16） |
| usage 計上 | 確認済み（OpenCode の利用履歴で表示枠が `Go`） |
| `auth.json` による再起動後の再利用 | 成功（Pi 終了後に再起動して `OpenCode Go ✓ stored` を確認） |

テキスト応答では、Pi の `/login` で `OpenCode Go` を選び、API key を設定した後、`deepseek-v4-flash` で `hello` を送信した。Pi のステータス表示は `(opencode-go) deepseek-v4-flash` で、モデルからの応答取得に成功した。

読み取り専用 tool call は、Herdr 経由で横のペインの Pi に依頼し、`pwd` / `ls -la` の結果を受けて応答することを確認した。今回の実測では対象がリポジトリの作業ディレクトリだったが、ファイルの作成・変更・削除とネットワークアクセスはなかった。

#### OpenCode Go usage の実行結果

OpenCode の利用履歴で、Pi のセッション `d-db630a` に紐づく次の3件を確認した。

| 時刻 | モデル | 入力 | 出力 | 表示された枠 | 表示コスト |
| --- | --- | ---: | ---: | --- | ---: |
| 8月16日 15:22 | `deepseek-v4-flash` | 2,271 | 78 | Go | $0.0003 |
| 8月16日 15:24 | `deepseek-v4-flash` | 2,416 | 155 | Go | $0.0001 |
| 8月16日 15:24 | `deepseek-v4-flash` | 2,791 | 121 | Go | $0.0001 |

表示上の合計は、入力 7,478、出力 354、コスト $0.0005 で、3件すべてが `Go` 枠として記録されていた。

Pi のステータスバーに表示される `$0.000` は、OpenCode の利用履歴におけるサブスクリプション枠への計上確認とは別の情報として扱う。枠への計上は OpenCode の利用履歴で確認する。

#### `/login` の永続保存に関する確認

初回の Pi セッションでは API key の認証と API 呼び出しは成功したが、`/login` 後に以下のエラーが表示された。

```text
Error: Failed to save API key for OpenCode Go: This operation was aborted
```

その後、Pi を終了して再起動したところ、provider 選択画面に `OpenCode Go ✓ stored` が表示された。したがって、credential が保存され、再起動後に再利用できることを確認した。初回の保存エラーの原因と、その後解消した条件は未確認として残す。

### Issue #61: Anthropic Messages

#### 環境確認

| 項目 | 結果 |
| --- | --- |
| Pi バージョン | 0.84.2（確認済み） |
| built-in provider | `opencode-go`（確認済み） |
| 対象モデル | `minimax-m3`（確認済み） |
| API 形式 | `anthropic-messages`（Pi の built-in 定義で確認） |
| base URL | `https://opencode.ai/zen/go`（Pi の built-in 定義で確認） |
| POST エンドポイント | `https://opencode.ai/zen/go/v1/messages`（SDK の呼び出し経路から確認） |
| 認証 | Issue #59 で保存確認済みの `auth.json` credential を再利用 |

#### 実機検証

| 項目 | 結果 |
| --- | --- |
| テキスト応答 | 成功（2026-08-16、`opencode-go` / `minimax-m3`） |
| 読み取り専用 tool call | 成功（一時ディレクトリで `pwd` / `ls -la`、2026-08-16） |
| usage 計上 | 確認済み（6件すべて OpenCode の利用履歴で `Go` と表示） |
| reasoning / thinking | ステータスバーの `high` は確認、独立した内容表示は未確認 |
| Anthropic モデル選択後の認証保存・再起動 | 未確認（#59 で確認済みの credential を再利用） |

#### テキスト応答の実行結果

Pi の `/model` で `minimax-m3` を選び、`hello` を送信した。

- Pi のステータス表示: `(opencode-go) minimax-m3`
- モデルからの応答取得: 成功
- 判定: Pi の built-in `opencode-go` provider 経由で Anthropic Messages 互換モデルを呼び出せることを確認

#### 読み取り専用 tool call の実行結果

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

#### OpenCode Go usage の実行結果

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

#### 認証の確認結果と未確認事項

今回の実機検証では、Issue #59 の検証で `/login` により保存確認済みの `auth.json` credential を引き継いだ。`minimax-m3` 選択後に `/login` と再起動を独立して再実施したわけではないため、Anthropic モデル選択を起点にした保存テストは未確認として残す。

## 制約・注意点

- OpenCode Go のモデル一覧、model ID、利用制限は変更される可能性がある
- `auth.json` と環境変数に異なる credential がある場合、Pi は `auth.json` を優先する
- API key を Git 管理、Issue、ログ、スクリーンショットへ含めない
- built-in provider が正常に動作する場合、custom provider の設定は不要
- 生の認証ヘッダー、SSE イベント、thinking 内容は今回の Anthropic 検証では個別に採取していない
- Anthropic モデル選択後の独立した再起動テストは未確認である

## 参考

- [OpenCode Go](https://opencode.ai/docs/ja/go/)
- [Pi Providers](https://pi.dev/docs/latest/providers)
- [Pi 0.56.0 release](https://pi.dev/news/releases/0.56.0)
- [Issue #59](https://github.com/u7chan/workstation-notes/issues/59)
- [Issue #61](https://github.com/u7chan/workstation-notes/issues/61)
