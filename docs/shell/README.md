# Bash 設定の管理

Last reviewed: 2026-03-23

## 概要

Bash 関連の設定は、実際の配置先とリポジトリ内の管理名を分ける。

- 実配置先: `~/.bashrc`, `~/.bashrc.local`
- 管理用サンプル: `examples/bash/bashrc`, `examples/bash/bashrc.local`

`docs` では役割と適用先を説明し、設定例そのものは `examples` で管理する。

## 役割分担

### `~/.bashrc`

- 共通の初期化処理を置く
- `~/.bashrc.local` を条件付きで読み込む
- WSL 判定、SSH エージェント、共通 alias、PATH のような土台を持たせる

### `~/.bashrc.local`

- 個人用 alias、関数、プロンプト、任意のツール設定を置く
- 変更頻度が高い設定をこちらへ寄せる
- LiteLLM 関連 env もここから読み込む

## このリポジトリでの参照元

- `examples/bash/bashrc`
- `examples/bash/bashrc.local`

## 関連ドキュメント

- `docs/starship-setup/README.md`
- `docs/windows-terminal-settings/README.md`

これらの手順書で出てくる `.bashrc` は、実際の適用先として読む。

## 更新ルール

1. まずこのページで背景と適用方針を更新する
2. 次に `examples/bash/` 配下の対応ファイルを更新する
3. 過去の断片メモを増やしすぎず、現在の推奨に寄せる
