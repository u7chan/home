# Claude Code / Codex のタスク完了時に音を鳴らす

前提:

- Windows 11
- WSL2 (Ubuntu 24.04)
- Bash

## 共通

`paplay` を使って wav を再生する。

```bash
sudo apt update
sudo apt install pulseaudio-utils -y
```

音量の目安:

- `0`: 無音
- `16384`: 25%
- `32768`: 50%
- `65536`: 100%

再生例:

```bash
paplay --volume=32768 ~/.voicevox/codex_notify.wav
```

通知音ファイルは `~/.voicevox/` に置いておくと扱いやすい。

## Codex CLI

`.codex/config.toml` に `notify` を設定する。

```toml
notify = ["bash", "-lc", "paplay --volume=32768 ~/.voicevox/codex_notify.wav"]
```

`bash -lc` 経由にしておくと `~` をそのまま使える。

## Claude Code

`.claude/settings.json` に `Stop` hook を設定する。

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "paplay --volume=32768 ~/.voicevox/claude_notify.wav"
          }
        ]
      }
    ]
  }
}
```

タスク完了時に `paplay` が実行され、通知音が鳴る。
