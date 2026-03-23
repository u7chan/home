# workstation-notes

個人用の開発環境メモと、再利用しやすい設定例をまとめるリポジトリ。

## 方針

- `docs/`: 現在の推奨手順、背景、適用先
- `examples/`: コピーベースの設定例
- ルート直下: 入口と案内だけを置く

特に、ツールが実ディレクトリとして誤認しやすい設定や、配置先にだけ意味がある設定は、安全名で `examples/` 側に置いて管理する。

## 入口

- `docs/shell/README.md`: Bash 設定の役割分担と適用先
- `docs/litellm/README.md`: LiteLLM 関連 env の扱い
- `docs/claude/README.md`: Claude 設定の管理方針

## 主要な設定例

- `examples/bash/bashrc`
- `examples/bash/bashrc.local`
- `examples/litellm/models.env.example`
- `examples/litellm/proxy.secrets.env.example`
- `examples/claude/settings.json`

## 運用ルール

1. ベストプラクティス変更時は、先に `docs/` を更新する
2. 次に対応する `examples/` を更新する
3. `docs/` には適用先と見直し日を残す

`examples/` はそのままのファイル名で実配置する前提ではなく、必要な場所へコピーして使うための参照実装として扱う。
