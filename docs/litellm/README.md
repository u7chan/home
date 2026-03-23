# LiteLLM 設定の管理

Last reviewed: 2026-03-23

## 概要

LiteLLM 関連の値は、実ファイル名をそのままリポジトリで追わず、`examples` に安全名で置く。

- 実配置先: `~/.litellm-proxy.secrets`, `~/.litellm-models`
- 管理用サンプル:
  - `examples/litellm/proxy.secrets.env.example`
  - `examples/litellm/models.env.example`

## 読み込み関係

- `~/.bashrc.local` から `~/.litellm-proxy.secrets` を読む
- 続けて `~/.litellm-models` を読む
- API キー本体やローカル URL は、実ファイル側で上書きする

## このリポジトリでの参照元

- `examples/litellm/proxy.secrets.env.example`
- `examples/litellm/models.env.example`
- `examples/bash/bashrc.local`

## 運用ルール

- 秘密情報の実体は追跡しない
- モデル名やデフォルトの推奨が変わったら、先にこのページを直す
- その後に `examples/litellm/` と `examples/bash/bashrc.local` の整合を取る
