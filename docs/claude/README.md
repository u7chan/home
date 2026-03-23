# Claude 設定の管理

Last reviewed: 2026-03-23

## 概要

Claude 関連の設定は、`.claude/` をそのままリポジトリの正本にしない。

理由:

- ツールが `.claude/` を実ディレクトリとして誤認しやすい
- 実行環境用のローカル設定と、共有したい参照設定を分けたい

このため、追跡対象は `examples/claude/settings.json` に置き、実際の適用先だけをドキュメントで示す。

## 実配置先と管理場所

- 実配置先: `.claude/settings.json`
- 管理用サンプル: `examples/claude/settings.json`

## このリポジトリでの使い分け

- `docs/claude/README.md`: 管理方針
- `examples/claude/settings.json`: 共有したい設定例
- ローカルの `.claude/`: 実行環境用。必要なら個別に持つ

## 関連ドキュメント

- `docs/task-complete-sound/README.md`
- `docs/claude-code-newline-setup/README.md`
- `docs/claude-code-vscode-newline-setup/README.md`

## 更新ルール

1. `.claude/` 実ディレクトリへ直接説明や共有設定を溜め込まない
2. 共有したい変更は `examples/claude/settings.json` に寄せる
3. 挙動や背景の説明は `docs/claude/README.md` に寄せる
