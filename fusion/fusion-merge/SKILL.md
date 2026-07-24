---
name: fusion-merge
description: Pull another linked session's output into this chat and merge it — plans, last answer, or history from fusion group members. Use when the user invokes /fusion-merge or asks to merge/pull/compare work from linked chats or parallel model attempts.
---

# fusion-merge

Pull selected content from OTHER fusion group members into THIS chat and synthesize:
consensus vs divergence, then merge the best. Typical use: two models attacked the same
problem in parallel — combine their answers or plans.

## Arguments

`/fusion-merge [member-label …] [plans|answer|history]`

- Members omitted → all OTHER members.
- Content kind omitted → present a short menu (plans / last answer / whole-history summary)
  and let the user pick.

## Procedure

1. Read `~/.agents/fusion/links.json`. No group or no other members → say so and stop.
2. Exclude THIS chat: match by the quick self-ID (Claude Code `$CLAUDE_CODE_SESSION_ID`;
   Codex newest `threads` row for this cwd; Cursor `cursor-identity.json`). If you cannot
   tell which member is this chat, ask.
3. **State which sessions you are about to read before reading them.**
4. Read each source member SELECTIVELY (never dump a full raw transcript into context;
   read tail-first and extract):
   - **Claude Code** (`<sessionId>.jsonl`): keep `type` user/assistant lines with
     `isSidechain:false`, skip `isMeta`; assistant text lives in `message.content` text
     blocks. *answer* = last assistant text. *history* = per-turn user message + final
     assistant text, then summarize. *plans* = plan-mode output and any plan file paths
     mentioned (e.g. `docs/superpowers/plans/…`) — read those files from the member's cwd.
   - **Codex** (rollout jsonl): `event_msg` records with `payload.type`
     `user_message`/`agent_message`, plus `response_item` `message` records. Same
     answer/history/plans extraction.
   - **Cursor**: read-only sqlite against the global `state.vscdb`
     (Windows `%APPDATA%/Cursor/User/globalStorage/state.vscdb`, macOS
     `~/Library/Application Support/Cursor/User/globalStorage/state.vscdb`; URI `mode=ro`):
     `SELECT value FROM cursorDiskKV WHERE key LIKE 'bubbleId:<sessionId>:%'` — JSON rows
     with `type` (1 user / 2 assistant), `text`, `createdAt`; order by `createdAt` or by
     `fullConversationHeadersOnly` in `composerData:<sessionId>`.
5. A member stale/unreadable → REPORT it and continue with the rest. Never silently drop.
6. Synthesize in this chat:
   - Where sources agree → the consensus, stated once.
   - Where they diverge → show the divergence explicitly and judge it (divergence between
     different vendors' models is signal, not noise).
   - Produce the merged result (answer or plan). Merging plans → offer to write the merged
     plan file.

Read-only toward all other tools' stores, always.
