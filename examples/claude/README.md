## settings.json

```jsonc
{
    "$schema": "https://json.schemastore.org/claude-code-settings.json",
    "env": {
        // 別のLLMにホスティングする際にパラメーターエラーが出る場合、
        // これをOFFにすると回避できる。もしくはMCPをOFFにしておく。
        "ENABLE_TOOL_SEARCH": "false"
    },
    "permissions": {
        "allow": [
            "Bash(pwd)",
            "Bash(ls *)",
            "Bash(echo *)",
            "Bash(find *)",
            "Bash(mkdir *)",
            "Bash(touch *)",
            "Bash(which *)",
            "Bash(date *)",
            "Bash(git show:*)",
            "Bash(git log:*)",
            "Bash(git diff:*)",
            "Bash(git status:*)",
            "Bash(git add:*)",
            "Bash(git commit:*)",
            "Bash(git stash:*)",
            "Bash(git switch *)",
            "Bash(gh pr view:*)",
            "Bash(gh pr diff:*)",
            "Bash(gh issue view:*)",
            "Bash(npm run *)",
            "Bash(yarn *)",
            "Bash(pnpm *)",
            "Bash(bun run *)"
        ],
        "deny": [
            "Read(//**/.env)",
            "Read(//**/.env.development)",
            "Read(//**/.env.local)",
            "Read(//**/.env.production)",
            "Read(//**/*.key)",
            "Read(//**/*.p12)",
            "Read(//**/*.pem)",
            "Read(//**/*secret*)",
            "Read(//**/*token*)",
            "Read(//etc/passwd)",
            "Read(//id_ed25519)",
            "Read(//id_rsa*)",
            "Read(~/.aws/**)",
            "Read(~/.config/**)",
            "Read(~/.ssh/**)",
            "Write(//**/.env)",
            "Write(//**/.env.development)",
            "Write(//**/.env.local)",
            "Write(//**/.env.production)",
            "Write(//**/*secret*)",
            "Bash(git clean -fd *)",
            "Bash(git push -f *)",
            "Bash(git reset --hard *)",
            "Bash(gh repo delete:*)",
            "Bash(nc *)",
            "Bash(netcat *)",
            "Bash(npm publish *)",
            "Bash(rm -rf /)",
            "Bash(rm -rf /*)",
            "Bash(rm -rf *)",
            "Bash(rm -rf ~)",
            "Bash(rm -rf ~/*)",
            "Bash(rsync *)",
            "Bash(scp *)",
            "Bash(ssh *)",
            "Bash(sudo *)"
        ],
        "ask": [
            "Bash(cat *)",
            "Bash(grep *)",
            "Bash(rm *)",
            "Bash(mv *)",
            "Bash(chmod *)",
            "Bash(chown *)",
            "Bash(curl *)",
            "Bash(wget *)",
            "Bash(git checkout:*)",
            "Bash(git merge *)",
            "Bash(git push *)",
            "Bash(gh pr create:*)",
            "Read(~/.zshrc)",
            "Read(~/.bashrc)"
        ]
    },
    "hooks": {
        // タスク終了時に音を鳴らす場合
        "Stop": [
            {
                "hooks": [
                    {
                        "type": "command",
                        "command": "paplay ~/.voicevox/claude_notify.wav"
                    }
                ]
            }
        ]
    },
    // ステータスラインのカスタマイズ
    "statusLine": {
        "type": "command",
        "command": "python3 /home/u7dev/.claude/statusline.py"
    },
    "language": "Japanese"
}
```

## statusline.py

```py
#!/usr/bin/env python3
"""Pattern 2: Sparkline gauge - vertical block characters"""
import json, sys, os, subprocess
if sys.platform == 'win32':
    sys.stdout.reconfigure(encoding='utf-8')

data = json.load(sys.stdin)

SPARKS = ' ▁▂▃▄▅▆▇█'
R = '\033[0m'
DIM = '\033[2m'

def gradient(pct):
    if pct < 50:
        r = int(pct * 5.1)
        return f'\033[38;2;{r};200;80m'
    else:
        g = int(200 - (pct - 50) * 4)
        return f'\033[38;2;255;{max(g, 0)};60m'

def spark_gauge(pct, width=8):
    pct = min(max(pct, 0), 100)
    level = pct / 100
    gauge = ''
    for i in range(width):
        seg_start = i / width
        seg_end = (i + 1) / width
        if level >= seg_end:
            gauge += SPARKS[8]
        elif level <= seg_start:
            gauge += SPARKS[0]
        else:
            frac = (level - seg_start) / (seg_end - seg_start)
            gauge += SPARKS[int(frac * 8)]
    return gauge

def fmt(label, pct):
    p = round(pct)
    return f'{DIM}{label}{R} {gradient(pct)}{spark_gauge(pct)}{R} {p}%'

def get_cwd_and_branch():
    cwd = data.get('cwd') or os.getcwd()
    dirname = os.path.basename(cwd)
    try:
        branch = subprocess.check_output(
            ['git', '-C', cwd, 'rev-parse', '--abbrev-ref', 'HEAD'],
            stderr=subprocess.DEVNULL
        ).decode().strip()
    except Exception:
        branch = '--'
    return dirname, branch

model = data.get('model', {}).get('display_name', 'Claude')
dirname, branch = get_cwd_and_branch()
parts = [f'{model}  \033[36m{dirname}\033[0m \033[35m{branch}\033[0m']

ctx = data.get('context_window', {}).get('used_percentage')
if ctx is not None:
    parts.append(fmt('ctx', ctx))

five = data.get('rate_limits', {}).get('five_hour', {}).get('used_percentage')
if five is not None:
    parts.append(fmt('5h', five))

week = data.get('rate_limits', {}).get('seven_day', {}).get('used_percentage')
if week is not None:
    parts.append(fmt('7d', week))


# Token counts
def format_tokens(n):
    if n >= 1_000_000:
        return f'{n/1_000_000:.1f}M'
    elif n >= 1_000:
        return f'{n/1_000:.1f}k'
    return str(n)

usage = data.get('context_window', {}).get('current_usage', {})
total_in = (usage.get('input_tokens', 0)
            + usage.get('cache_creation_input_tokens', 0)
            + usage.get('cache_read_input_tokens', 0))
total_out = usage.get('output_tokens', 0)
in_fmt = format_tokens(total_in)
out_fmt = format_tokens(total_out)

# Tokens
parts.append(f'\033[0;37min:{in_fmt} out:{out_fmt}\033[0m')

# Cost
cost_usd = data.get('cost', {}).get('total_cost_usd', 0)

if cost_usd and cost_usd > 0:
    parts.append(f'\033[0;33m${cost_usd:.2f}\033[0m')

print(f' {DIM}│{R} '.join(parts), end='')

```
