# VSCode 上の Claude Code CLI で Shift+Enter 改行を有効にする

前提:

- VSCode
- Claude Code CLI
- Remote Development を使っている場合は、ローカル側の VSCode に設定を入れられること

## 概要

VSCode 上で Claude Code CLI をターミナルから起動した場合、既定では `Shift+Enter` を押しても改行されないことがある。

その場合は、VSCode の `keybindings.json` にターミナル向けのキーバインドを追加すると、`Shift+Enter` で改行できるようになる。

`/terminal-setup` を実行したときに次のような案内が出た場合は、この設定が必要である。

```text
Cannot install keybindings from a remote VSCode session.

VSCode keybindings must be installed on your local machine, not the remote server.
```

## 手順

### 1. ローカルの VSCode で `keybindings.json` を開く

リモート接続中ではなく、ローカルの VSCode を開く。

`Cmd/Ctrl+Shift+P` でコマンドパレットを開き、`Preferences: Open Keyboard Shortcuts (JSON)` を実行する。

### 2. `keybindings.json` に設定を追加する

`keybindings.json` が JSON 配列になっていることを確認し、次の設定を追記する。

```json
[
  {
    "key": "shift+enter",
    "command": "workbench.action.terminal.sendSequence",
    "args": { "text": "\u001b\r" },
    "when": "terminalFocus"
  }
]
```

すでに他の設定がある場合は、配列の中にこのオブジェクトを追加する。

### 3. 保存して確認する

`keybindings.json` を保存する。

VSCode のターミナルで Claude Code CLI を開き、`Shift+Enter` を押して改行されることを確認する。

## 補足

- Remote Development で接続している場合でも、設定先はリモート側ではなくローカル側の VSCode である。
- この設定は VSCode ターミナル全体に適用される。
- `when` に `terminalFocus` があるため、エディタ上の `Shift+Enter` には影響しない。
