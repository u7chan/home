# Pi で OpenCode Go のサブスクリプション枠を利用する

Last reviewed: 2026-08-16

## 概要

Pi の `opencode-go` built-in provider を使い、OpenCode Go のサブスクリプション枠でモデルを利用する手順をまとめる。

このページでは OpenAI Chat Completions 互換の `deepseek-v4-flash` を検証対象とする。Anthropic Messages 互換エンドポイントは [Issue #61](https://github.com/u7chan/workstation-notes/issues/61) で別途検証する。

## 前提条件

- OpenCode Go のサブスクリプションを有効化している
- OpenCode Zen で OpenCode Go 用の API key を発行済み
- Pi 0.56.0 以降を利用する
- API key は環境変数または Pi の `auth.json` に設定する
- API key をリポジトリへ保存しない

Pi 0.56.0 で OpenCode Go provider のサポートが追加された。現在の Pi では provider ID は `opencode-go`、環境変数は `OPENCODE_API_KEY` である。

## OpenCode Go 側の情報

OpenCode Go の公式ドキュメントで、モデル一覧、エンドポイント、利用制限を確認できる。

- [OpenCode Go 公式ドキュメント](https://opencode.ai/docs/ja/go/)
- [モデル一覧エンドポイント](https://opencode.ai/zen/go/v1/models)

今回の検証対象：

| 項目 | 値 |
| --- | --- |
| Pi provider | `opencode-go` |
| Pi model | `opencode-go/deepseek-v4-flash` |
| API 形式 | OpenAI Chat Completions 互換 |
| base URL | `https://opencode.ai/zen/go/v1` |
| エンドポイント | `https://opencode.ai/zen/go/v1/chat/completions` |

モデル一覧は変更される可能性があるため、実行時に Pi のモデル一覧と公式エンドポイントの結果を確認する。

2026-08-16 に `GET https://opencode.ai/zen/go/v1/models` を確認したところ、HTTP 200 が返り、26 モデルの中に `deepseek-v4-flash` が含まれていた。モデル一覧の確認自体には API key は要求されなかった。

## Pi 側の設定

### 方法 1: 環境変数で一時的に設定する

API key をシェルの環境変数に設定して、Pi を起動する。

```bash
export OPENCODE_API_KEY='ここに OpenCode Go の API key'
pi --model opencode-go/deepseek-v4-flash
```

API key をシェル履歴やプロセス一覧に残したくない場合は、シェルの secret input 機能や環境変数管理ツールを使う。実際の key をコマンド例やリポジトリへ記録しない。

key の値を表示せずに認証状態を確認するには、以下を実行する。

```bash
pi auth check --provider opencode-go --no-refresh
```

### 方法 2: `auth.json` に保存する

Pi の標準設定ディレクトリは `~/.pi/agent` である。`auth.json` は所有者だけが読み書きできる権限にする。

今回の実機検証では、`auth.json` を直接編集せず、Pi の `/login` で `Sign in with an API key` と `OpenCode Go` を選択して API key を登録した。この操作は、Pi の認証フロー経由で `auth.json` に credential を保存する方法 2 に相当する。

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

## モデル選択

```bash
pi --list-models opencode-go
```

対象モデルを明示して起動する。

```bash
pi --model opencode-go/deepseek-v4-flash
```

モデル ID は公式ドキュメントと Pi の表示を実行時に確認する。古い記事に記載されたモデル名をそのまま固定しない。

## 疎通確認

### 1. テキスト応答

セッションを保存せず、短い prompt で応答を確認する。

```bash
pi --no-session \
  --model opencode-go/deepseek-v4-flash \
  -p 'OpenCode Go の疎通確認です。OK とだけ返してください。'
```

記録する項目:

- 実行日時
- `pi --version` の結果
- provider と model ID
- 成功 / 失敗
- 失敗した場合の HTTP status とエラーメッセージ

### 2. 読み取り専用 tool call

書き込みを避けるため、一時ディレクトリで実行する。

```bash
probe_dir="$(mktemp -d)"
cd "$probe_dir"

pi --no-session \
  --model opencode-go/deepseek-v4-flash \
  -p 'このディレクトリで pwd と読み取り専用のファイル一覧の確認を行い、結果を短く報告してください。ファイルの作成・変更・削除はしないでください。'
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
- 表示が変化したか

利用履歴への反映に時間がかかる場合があるため、API 応答の成功とサブスクリプション枠への計上を分けて記録する。数値の変化を確認できない場合は「計上未確認」と明記する。

## 検証結果

### 環境確認

| 項目 | 結果 |
| --- | --- |
| Pi バージョン | 0.84.2（確認済み） |
| built-in provider | `opencode-go`（確認済み） |
| 対象モデルの定義 | `deepseek-v4-flash` / `openai-completions` / `https://opencode.ai/zen/go/v1`（確認済み） |
| `/v1/models` エンドポイント | HTTP 200、26 モデル、`deepseek-v4-flash` を確認（2026-08-16） |
| `OPENCODE_API_KEY` | `/login` の API key 認証で設定（実機検証） |
| `auth.json` の `opencode-go` credential | Pi 再起動後も `OpenCode Go ✓ stored` を確認 |

### 実機検証

手順例では `--no-session` と一時ディレクトリを使うが、今回の実機検証では Herdr の横のペインで起動済みの対話セッションを使い、リポジトリの作業ディレクトリを対象に実行した。以下はその実測結果であり、ファイルの作成・変更・削除は行っていない。

| 項目 | 結果 |
| --- | --- |
| テキスト応答 | 成功（2026-08-16、`opencode-go` / `deepseek-v4-flash`） |
| 読み取り専用 tool call | 成功（`pwd` / `ls -la`、2026-08-16） |
| usage 計上 | 確認済み（OpenCode の利用履歴で表示枠が `Go`） |
| `auth.json` による再起動後の再利用 | 成功（Pi 終了後に再起動して `OpenCode Go ✓ stored` を確認） |

### テキスト応答の実行結果

Pi の `/login` で `OpenCode Go` を選び、API key を設定した後、`deepseek-v4-flash` で `hello` を送信した。

- Pi のステータス表示: `(opencode-go) deepseek-v4-flash`
- モデルからの応答取得：成功
- 判定：Pi の built-in `opencode-go` provider 経由の実呼び出しを確認

Pi のステータスバーに表示される `$0.000` は、OpenCode の利用履歴におけるサブスクリプション枠への計上確認とは別の情報として扱う。枠への計上は OpenCode の利用履歴で確認する。

### 読み取り専用 tool call の実行結果

Herdr 経由で、横のペインで起動している Pi に以下の指示を送り、`opencode-go/deepseek-v4-flash` で実行した。

- `pwd` の結果：`/home/u7dev/workspace/workstation-notes`
- `ls -la` の結果：`.git/`、`.gitignore`、`README.md`、`docs/`、`examples/` などを確認
- ファイルの作成・変更・削除：なし
- ネットワークアクセス：なし
- 判定：読み取り専用 tool call と、その結果を受けた応答を確認

### OpenCode Go の利用履歴（usage）

OpenCode の利用履歴で、Pi の現在のセッション `d-db630a` に紐づく次の3件を確認した。

| 時刻 | モデル | 入力 | 出力 | 表示された枠 | 表示コスト |
| --- | --- | ---: | ---: | --- | ---: |
| 8月16日 15:22 | `deepseek-v4-flash` | 2,271 | 78 | Go | $0.0003 |
| 8月16日 15:24 | `deepseek-v4-flash` | 2,416 | 155 | Go | $0.0001 |
| 8月16日 15:24 | `deepseek-v4-flash` | 2,791 | 121 | Go | $0.0001 |

表示上の合計は、入力 7,478、出力 354、コスト $0.0005 で、3件すべてが `Go` 枠として記録されていた。これにより、Pi の `opencode-go` 経由の呼び出しが OpenCode Go のサブスクリプション枠として処理されたことを確認した。

### `/login` の永続保存に関する確認

初回の Pi セッションでは API key の認証と API 呼び出しは成功したが、`/login` 後に以下のエラーが表示された。

```text
Error: Failed to save API key for OpenCode Go: This operation was aborted
```

その後、Pi を終了して再起動したところ、provider 選択画面に `OpenCode Go ✓ stored` が表示された。したがって、現在の設定では credential が保存され、再起動後に再利用できることを確認した。初回の保存エラーの原因と、その後解消した条件は未確認として残す。

## 制約・注意点

- OpenCode Go のモデル一覧、model ID、利用制限は変更される可能性がある
- `auth.json` と環境変数に異なる credential がある場合、Pi は `auth.json` を優先する
- API key を Git 管理、Issue、ログ、スクリーンショットへ含めない
- built-in provider が正常に動作する場合、custom provider の設定は不要
- Anthropic Messages 互換エンドポイントは [Issue #61](https://github.com/u7chan/workstation-notes/issues/61) の対象とする

## 参考

- [OpenCode Go](https://opencode.ai/docs/ja/go/)
- [Pi Providers](https://pi.dev/docs/latest/providers)
- [Pi 0.56.0 release](https://pi.dev/news/releases/0.56.0)
