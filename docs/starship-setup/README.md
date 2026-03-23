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

Bash 設定全体の管理方針は `docs/shell/README.md` を参照。

```bash
# starship init
eval "$(starship init bash)"
```

## 反映

シェルを再起動する。

## Pastel Powerline プリセット

前提:

- シェルで Nerd Font が有効になっていること

公式:

- https://starship.rs/ja-JP/presets/pastel-powerline#pastel-powerline%E3%83%95%E3%82%9A%E3%83%AA%E3%82%BB%E3%83%83%E3%83%88

`Pastel Powerline` プリセットを適用する場合は、以下を実行する。

```sh
starship preset pastel-powerline -o ~/.config/starship.toml
```

このコマンドで見た目が変化する。
