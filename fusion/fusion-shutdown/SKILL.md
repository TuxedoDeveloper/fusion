---
name: fusion-shutdown
description: End the fusion linked group entirely (unlink all chats). Use when the user invokes /fusion-shutdown or asks to end/reset/wipe the fusion group or links.
---

# fusion-shutdown

End the fusion group. Callable from ANY chat — member or not. This is the ONLY way links
die (no expiry, no auto-pruning anywhere else).

1. If `~/.agents/fusion/links.json` does not exist → "no active fusion group". Done.
2. Read it, note the member count, then delete ONLY that file (use your file tools where
   possible; deletion may need a shell — say so). Leave everything else in
   `~/.agents/fusion/` untouched: `reviews/` history and `cursor-identity.json` stay.
3. Confirm: "fusion group ended — N member(s) unlinked."
