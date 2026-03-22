# starship のセットアップ

`starship` はシェルプロンプトをカスタマイズするためのツール。

公式:

- https://starship.rs/ja-JP/

今回の前提:

- Bash

## インストール

最新版をインストールする。

```bash
curl -sS https://starship.rs/install.sh | sh
```

## Bash 設定

`~/.bashrc` の最後に以下を追記する。

```bash
# starship init
eval "$(starship init bash)"
```

## 反映

シェルを再起動する。
