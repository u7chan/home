# VSCode で Windows 向けにターミナル分割とクローズのショートカットを設定する

前提:

- Windows
- VSCode

## 概要

VSCode のターミナルで、次のショートカットを使えるようにするメモ。

- `Ctrl+D`: ターミナルを左右分割
- `Ctrl+W`: 現在のターミナルを閉じる

`keybindings.json` に `terminalFocus` 条件付きで設定するため、エディタ側のショートカットには影響しない。

## 手順

### 1. `keybindings.json` を開く

`Ctrl+Shift+P` でコマンドパレットを開き、`Preferences: Open Keyboard Shortcuts (JSON)` を実行する。

### 2. 設定を追加する

`keybindings.json` が JSON 配列になっていることを確認し、次の設定を追加する。

```json
[
  {
    "key": "ctrl+d",
    "command": "workbench.action.terminal.split",
    "when": "terminalFocus"
  },
  {
    "key": "ctrl+w",
    "command": "workbench.action.terminal.kill",
    "when": "terminalFocus"
  }
]
```

すでに他の設定がある場合は、配列の中にこの 2 件を追加する。

## 設定内容

|ショートカット|動作|条件|
|-|-|-|
|`Ctrl+D`|ターミナルを左右分割|ターミナルにフォーカスがあるとき|
|`Ctrl+W`|現在のターミナルを閉じる|ターミナルにフォーカスがあるとき|

## 補足

- `when: terminalFocus` を付けているため、エディタ上の `Ctrl+D` や `Ctrl+W` には影響しない。
- ターミナル内では `Ctrl+D` の既定動作（EOF 送信）はこの設定で上書きされる。
