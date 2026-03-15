# Windows Terminal settings

Windowsターミナルのキーバインド（カスタムメモ）

`+ボタン` > `設定` > `操作`

![terminal-settings](./images/terminal-settings.png)

|アクション|キーバインド|
|-|-|
|ペインを閉じる| ctrl+w|
|ペインの複製する, split: down| ctrl+shift+d|
|ペインの複製する, split: right| ctrl+d|

## WSLでペイン複製時にディレクトリを引き継ぐ

Windows Terminal + WSL の組み合わせでは、`.bashrc` に次の一行を追加すると、ペイン複製時にカレントディレクトリを引き継げる。

```bash
PROMPT_COMMAND=${PROMPT_COMMAND:+"$PROMPT_COMMAND ; "}'printf "\e]9;9;%s\e\\" "$(wslpath -w "$PWD")"'
```

設定後はシェルを再起動する。

この設定は Windows Terminal + WSL の場合に有効なカスタム。
