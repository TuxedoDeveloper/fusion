---
name: fusion-status
description: Show whether fusion is active and list all linked chats/sessions with health info. Use when the user invokes /fusion-status or asks which sessions are linked / whether fusion is on.
---

# fusion-status

Report the state of the fusion group from `~/.agents/fusion/links.json`. Read-only — this
skill changes nothing and never prunes.

1. File missing or `members` empty → "fusion inactive (no linked group)" / "group active, 0 members". Done.
2. Per member, gather:
   - tool, label, short sessionId, cwd, linkedAt.
   - Last activity: transcript file mtime (Claude Code jsonl / Codex rollout). Cursor:
     `lastUpdatedAt` via read-only query
     `SELECT json_extract(value,'$.lastUpdatedAt') FROM cursorDiskKV WHERE key='composerData:<sessionId>'`
     against the global `state.vscdb` (`mode=ro`); if that fails, mark "unknown".
   - Health: OK, or STALE with the reason (transcript missing/unreadable, DB locked, …).
3. Print a compact table plus the group's `created` date. STALE members are reported, never
   removed — only `/fusion-shutdown` or `/fusion-remove` change membership.
4. If unread review files exist in `~/.agents/fusion/reviews/` (mtime newer than the newest
   member activity you can attribute them to, or simply any from the last 24h), mention them.
