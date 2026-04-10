# Zed で Windows 向けにターミナル分割とクローズのショートカットを設定する

Last reviewed: 2026-04-10

前提:

- Windows
- Zed

## 概要

Zed のターミナルで、次のショートカットを使えるようにするメモ。

- `Ctrl+D`: ターミナルを左右分割
- `Ctrl+W`: アクティブなターミナルを閉じる

`keymap.json` に `Terminal` コンテキスト付きで設定するため、ターミナルにフォーカスがあるときだけ効く。エディタ側のショートカットには影響しない。

## 手順

### 1. `keymap.json` を開く

Zed のユーザー設定ファイル `%APPDATA%\\Zed\\keymap.json` を開く。

実体のパスは通常次のようになる。

`C:\\Users\\<username>\\AppData\\Roaming\\Zed\\keymap.json`

### 2. `Terminal` コンテキストの設定を追加する

`keymap.json` が JSON 配列になっていることを確認し、次の設定を追加する。

```jsonc
{
  "context": "Terminal",
  "bindings": {
    "ctrl-d": ["workspace::SendKeystrokes", "ctrl-shift-5"],
    "ctrl-w": "pane::CloseActiveItem"
  }
}
```

すでに他の設定がある場合は、配列の中にこのオブジェクトを追加する。

## 設定内容

|ショートカット|動作|条件|
|-|-|-|
|`Ctrl+D`|ターミナルを左右分割|ターミナルにフォーカスがあるとき|
|`Ctrl+W`|アクティブなターミナルを閉じる|ターミナルにフォーカスがあるとき|

## 補足

- `Terminal` コンテキスト付きのため、エディタ上の `Ctrl+D` や `Ctrl+W` には影響しない。
- ターミナル内では `Ctrl+D` の既定動作がこの設定で上書きされる。
- `Ctrl+D` は `workspace::SendKeystrokes` で Zed 既定の分割ショートカット `Ctrl+Shift+5` を送っている。
- 反映されない場合は Zed の再読み込みまたは再起動を試す。
