# fusion
Cross-session/cross-model fusion across Claude Code, Codex, and Cursor: linked live sessions merge and review each other's work, and any chat consults the other vendor's frontier model headlessly.

## Commands

| Command | What it does |
|---|---|
| `/fusion-advisor [target] [question]` | One-shot headless consult with the other vendor's model — works with zero setup |
| `/fusion-review [target \| diff \| member]` | Cross-vendor review of your git diff, a plan file, or a linked chat's work; "fix the fusion review" applies it |
| `/fusion-executor [target]` | Standing advisor mode for this chat — auto-consults at decision points (`off` to disable) |
| `/fusion-link` | Join this chat to the global linked group |
| `/fusion-status` | Show group members + health |
| `/fusion-merge [members] [plans\|answer\|history]` | Pull work from linked chats and synthesize consensus vs divergence |
| `/fusion-remove` | Unlink this chat only |
| `/fusion-shutdown` | End the group (the only way links expire) |

Targets: `claude` \| `gpt` \| `cursor:<model>` \| `both`. Omitted → automatically the *other*
vendor's model.

## Use it

Skills in `fusion/` — plain Markdown (Agent Skills format), folder name = slash command.

1. Copy each `fusion-*` folder flat into your skills dir:
   `~/.claude/skills/` (Claude Code; Cursor reads it too), `~/.agents/skills/` (Codex).
2. Try it: `/fusion-advisor <question>` asks the other vendor's model — needs its CLI
   installed (`claude` / `codex`). `/fusion-review` reviews your git diff cross-vendor.
3. Link two live chats with `/fusion-link` in each → `/fusion-merge`, `/fusion-review`,
   `/fusion-status`, `/fusion-shutdown`.

State lives in `~/.agents/fusion/`; other tools' data is only ever read.
