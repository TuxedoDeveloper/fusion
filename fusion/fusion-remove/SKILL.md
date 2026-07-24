---
name: fusion-remove
description: Remove only the current chat from the fusion linked group. Use when the user invokes /fusion-remove or asks to unlink this chat/session from fusion.
---

# fusion-remove

Remove THIS chat from `~/.agents/fusion/links.json`. Other members stay linked; the group
itself keeps existing (only `/fusion-shutdown` ends it — an empty group is fine).

1. Read `~/.agents/fusion/links.json`. Missing or no members → report "no active fusion group" and stop.
2. Identify this session the quick way:
   - Claude Code: `$CLAUDE_CODE_SESSION_ID`.
   - Codex: newest `threads` row for this cwd in `~/.codex/state_5.sqlite` (read-only, `mode=ro`).
   - Cursor: `conversation_id` from `~/.agents/fusion/cursor-identity.json` (fresh mtime).
3. Remove the member matching `tool` + `sessionId`. If no exact match, show the member list
   and ask which one is this chat — never guess-delete.
4. Write the file back (valid JSON) and print the remaining group.
