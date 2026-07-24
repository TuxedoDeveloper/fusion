---
name: fusion-link
description: Join the current chat to the fusion linked group (cross-tool session links across Claude Code, Codex, Cursor). Use when the user invokes /fusion-link or asks to link this chat/session into the fusion group.
---

# fusion-link

Join THIS chat to the single global fusion group so other linked sessions can merge from it
(`/fusion-merge`) and review it (`/fusion-review`). Exactly one group exists at a time; it
lives until `/fusion-shutdown` — never expire, rename, or prune it. Run `/fusion-link` in
more chats anytime to grow the group.

State file: `~/.agents/fusion/links.json`

## Step 1 — identify THIS session

You know which harness you are running in from your own system prompt. Follow that branch.
All reads of other tools' stores are READ-ONLY (sqlite always opened with `mode=ro`).

**Claude Code**
1. Session id: shell `echo "$CLAUDE_CODE_SESSION_ID"` (PowerShell: `$env:CLAUDE_CODE_SESSION_ID`).
2. Transcript: the single glob match of `~/.claude/projects/*/<sessionId>.jsonl`.
3. Fallback if the env var is empty: `cat ~/.claude/sessions/$CLAUDE_PID.json` → `sessionId` + `cwd`.
4. Label: `name` from that pid file if available, else the first few words of the chat's first user message.

**Codex**
1. Query the authoritative session index (read-only):
   ```
   python -c "import sqlite3,os;con=sqlite3.connect('file:'+os.path.expanduser('~/.codex/state_5.sqlite').replace(os.sep,'/')+'?mode=ro',uri=True);[print(list(r)) for r in con.execute('SELECT id,rollout_path,cwd,source,updated_at,title,first_user_message FROM threads ORDER BY updated_at DESC LIMIT 10')]"
   ```
2. Pick the newest row whose `cwd` ends with your own cwd (values may carry a `\\?\` prefix — compare suffixes, case-insensitive).
3. Confirm: read line 1 of `rollout_path` — a `session_meta` record whose `payload.cwd` matches and whose timestamp fits this session's start.
4. Tie-break (two recent sessions in the same cwd): grep the candidate rollouts for the exact text of the user's `/fusion-link` message — the one containing it is this session.
5. Label: `title` or `first_user_message` from the row.

**Cursor**
1. Read `~/.agents/fusion/cursor-identity.json` — written by the `beforeSubmitPrompt` hook the moment the user sent this very message.
2. Require its mtime to be within the last 2 minutes (another Cursor chat may overwrite it later; freshness ties it to THIS message). Extract `conversation_id`, `transcript_path` (may be null — record it anyway), `workspace_roots`.
3. Missing or stale file → the identity hook is not wired (setup needs a re-run): REPORT that, then continue with the announced fallback below — never fall back silently.
4. Fallback (say you are using it): newest composer for this workspace from the `composerHeaders` TABLE of the global DB (`%APPDATA%/Cursor/User/globalStorage/state.vscdb` on Windows, `~/Library/Application Support/Cursor/User/globalStorage/state.vscdb` on macOS), read-only:
   `SELECT composerId, workspaceId, recency FROM composerHeaders ORDER BY recency DESC LIMIT 10`
   — match `workspaceId` to this workspace (`workspaceStorage/<id>/workspace.json` folder). Do NOT use the legacy ItemTable key `composer.composerHeaders`: it is frozen at migration and silently stale. Several recent chats in the same workspace → list candidates and ask instead of guessing.
5. Label: the composer `name` from `composerData:<conversation_id>`, else `cursor-<first 8 chars of id>`.

## Step 2 — write links.json

1. Create `~/.agents/fusion/` and the file if absent: `{"created": "<UTC now>", "members": []}`.
2. If a member with the same `tool` + `sessionId` exists, refresh its fields instead of duplicating.
3. Append:
   ```json
   {
     "tool": "claude-code | codex | cursor",
     "sessionId": "<id>",
     "transcript": "<absolute transcript path; empty string if unknown (cursor)>",
     "cwd": "<this session's cwd>",
     "label": "<short label>",
     "linkedAt": "<UTC ISO timestamp>"
   }
   ```
4. Keep the file valid JSON; edit with your file tools.

## Step 3 — report

Print the full group (tool, label, short id, cwd per member) and note that
`/fusion-merge`, `/fusion-review`, `/fusion-status` now see this chat.

## Hard rules

- Self-ID failed or uncertain → report exactly what failed and write NOTHING. Never link a guess.
- Never modify another tool's databases, transcripts, or config.
