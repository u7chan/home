# Pi で ChatGPT の Codex サブスクリプションを利用する

Last reviewed: 2026-08-16

## 概要

Pi の built-in openai-codex provider を使い、ChatGPT Plus / Pro の Codex サブスクリプション経由で Codex モデルを利用する手順と検証結果をまとめる。

このページの検証は、Pi 0.84.2 から /login のブラウザログインを行った結果に基づく。API key を使う openai provider とは別の認証経路である。

OAuth の認証URL、クエリパラメーター、認証コード、access token、refresh token は安全上の理由から記録しない。

## 前提条件

- ChatGPT Plus または Pro の契約がある
- Pi 0.84.2 をインストールしている
- ブラウザを開ける環境である
- 認証情報や認証URLをリポジトリ、Issue、ログ、スクリーンショットへ保存しない

Pi 公式ドキュメントでは、ChatGPT Plus / Pro の Codex サブスクリプションは /login から利用できる subscription provider として案内されている。OAuth credential は ~/.pi/agent/auth.json に保存され、期限切れ時には自動 refresh される。

- [Pi Providers](https://pi.dev/docs/latest/providers)
- [Pi Quickstart](https://pi.dev/docs/latest/quickstart)
- [Issue #60](https://github.com/u7chan/workstation-notes/issues/60)

## /login でログインする

Pi を起動して、次の順に選択する。

~~~text
/login
  → Sign in with an account
  → OpenAI Codex
  → Browser login (default)
~~~

ブラウザでログインを完了すると、Pi に次のような完了メッセージが表示される。

~~~text
Logged in to OpenAI Codex. Credentials saved to ~/.pi/agent/auth.json
~~~

実際の環境では絶対パスが表示されるが、記録にはホームディレクトリを省略したパスを使う。Pi が表示する認証URLは、ブラウザで開くためだけに使用し、Issue・README・ログへ貼り付けない。

今回選択したのは Browser login (default) である。Device code login (headless) はログイン画面に表示されたが、今回の検証対象にはしていない。

## モデルを選択する

ログイン後、Pi 内で /model を実行し、[openai-codex] と表示されるモデルを選択する。

今回の検証では次のモデルを選択した。

| 項目 | 結果 |
| --- | --- |
| provider | openai-codex |
| model ID | gpt-5.6-luna |
| Pi 上の表示名 | GPT-5.6 Luna |
| 実行時の thinking level | high |

2026-08-16 の /model では、次の Codex モデルが表示された。

~~~text
gpt-5.3-codex-spark
gpt-5.4
gpt-5.4-mini
gpt-5.5
gpt-5.6-luna
gpt-5.6-sol
gpt-5.6-terra
~~~

モデルカタログと利用可能なモデルは変更される可能性があるため、README の一覧を固定的な対応表として扱わず、実行時に /model の表示を確認する。

## reasoning / thinking level の設定

Pi の interactive mode では Shift+Tab で thinking level を順番に切り替えられる。/settings の Thinking level から設定することもできる。起動時に指定する場合は、次のように --thinking を使う。

~~~bash
pi --model openai-codex/gpt-5.6-luna --thinking high
~~~

Pi 0.84.2 の thinking level には off、minimal、low、medium、high、xhigh、max があるが、利用できるレベルはモデルによって異なる。今回の実機検証で確認したのは、ステータスバーに表示された high だけである。他のレベルへ変更した場合の gpt-5.6-luna の挙動は確認していない。

- [Pi Quickstart](https://pi.dev/docs/latest/quickstart)
- [Using Pi](https://pi.dev/docs/latest/usage)
- [Pi Keybindings](https://pi.dev/docs/latest/keybindings)

## 疎通確認

### テキスト応答

gpt-5.6-luna を選択して hello を送信したところ、応答に成功した。

Pi のステータスバーには次のように表示された。

~~~text
(openai-codex) gpt-5.6-luna • high
~~~

### 読み取り専用 tool call

Herdr 経由で隣のPiに依頼し、一時ディレクトリで次の読み取り専用操作を実行した。

~~~bash
probe_dir="$(mktemp -d)"
(
  cd "$probe_dir"
  pwd
  ls -la
)
~~~

結果は成功し、作業ディレクトリには . と .. だけが存在した。リポジトリの状態も clean のままで、ファイルの変更はなかった。

### 新しいPiプロセスでの認証状態

検証では、ログインに使ったPiを終了して再起動したのではなく、保存済みの auth.json credential を使う別の非対話Piプロセスを起動した。/login を再実行せずに openai-codex のモデルを選択して hello を送信し、応答に成功したため、新しいPiプロセスが credential を再利用できることを確認した。

Piを終了して同じ手順で再起動する独立した実機検証は、今回の記録では未確認である。

なお、Pi 公式ドキュメントに記載されている token refresh の動作について、実際に有効期限切れを発生させるテストは行っていない。

### 検証環境と結果

| 項目 | 結果 |
| --- | --- |
| Pi | 0.84.2 |
| 認証方式 | /login → Sign in with an account → OpenAI Codex → Browser login |
| provider | openai-codex |
| モデル | gpt-5.6-luna |
| テキスト応答 | 成功 |
| 読み取り専用 tool call | 成功（temporary directory 内の pwd / ls -la） |
| 新しいPiプロセスでの再利用 | 成功（再ログインなし） |
| Pi終了後の再起動 | 未確認 |
| Pi の表示 | $0.000 (sub) |

Pi のステータスバーに表示された $0.000 (sub) は、今回の画面で確認できた表示である。ChatGPT 側の利用枠・上限への反映を別画面で照合したわけではないため、利用量の詳細は未確認とする。

検証中、一度モデルカタログの更新がタイムアウトし、キャッシュ済みのモデル一覧を使用する警告が表示された。しかし、選択済みモデルの応答と新しいPiプロセスでの認証確認は成功した。

## 画像の扱い

### 画像生成

Pi から現在のモデルに、単純なテスト画像を生成するよう依頼した。ただし、Pi またはこのモデルにネイティブな画像生成機能は公開されていなかった。

結果は次のとおりである。

- 画像生成: 未対応
- bitmap artifact: 作成されなかった
- スクリプト、SVG、ImageMagick などによる代替生成: 検証では使用していない

これは今回の Pi 経由の openai-codex 構成で確認した結果であり、モデルや別のクライアントが持つ画像生成機能全般を否定するものではない。

### 画像入力・読み取り

リポジトリ内の既存画像を、ファイル名やメタデータだけではなく画像コンテンツとしてPiへ渡して読み取らせた。

~~~text
docs/claude-code-newline-setup/open-json-file.png
~~~

Pi 経由の gpt-5.6-luna は画像を受け取り、次のような視覚的特徴を説明できた。

- ダークテーマの日本語設定画面
- 左側のサイドバー
- 角丸の設定パネル
- ドロップダウン、トグル、青・グレーのボタン
- 上部に表示された Ubuntu のタブ

したがって、今回の構成では画像入力・画像内容の読み取りは成功した。対象画像とリポジトリは読み取り専用で、画像ファイルの変更はなかった。

## API key 経路との違い

| provider | 認証 | 請求・利用枠 | 今回の扱い |
| --- | --- | --- | --- |
| openai-codex | /login のOpenAI Codex OAuth | ChatGPT Plus / Pro のCodexサブスクリプション | 実機検証済み |
| openai | OPENAI_API_KEY または auth.json のAPI key | OpenAI API の通常のAPI利用 | このIssueでは未検証 |

/login の Sign in with an API key で設定する openai provider は、openai-codex の subscription login とは別経路である。今回の検証ではAPI keyを使っていない。

## 未確認事項と制約

- Device code login (headless) の実機検証
- token の有効期限切れを発生させた refresh 検証
- reasoning level を変更した場合のモデルごとの挙動
- ChatGPT 側の利用枠・レート制限の詳細
- Codex CLI と Pi が同じ認証情報を直接共有できるか
- openai provider のAPI key経路との比較実測
- 画像生成機能が将来Piやモデルへ追加された場合の再検証

## セキュリティ上の注意

- OAuth 認証URL、クエリパラメーター、認証コード、access token、refresh tokenを記録・共有しない
- ~/.pi/agent/auth.json の内容をREADME、Issue、ログ、スクリーンショットへ貼り付けない
- API keyを使う場合も、環境変数の値や実際のkeyをリポジトリへ保存しない
- 画像入力の検証では、秘密情報や個人情報を含む画像を使わない

## 参考

- [Pi Providers](https://pi.dev/docs/latest/providers)
- [Pi Quickstart](https://pi.dev/docs/latest/quickstart)
- [Issue #60](https://github.com/u7chan/workstation-notes/issues/60)
