---
name: fusion-executor
description: Turn on this chat's permanent advisor mode — auto-consult a cross-vendor model at decision points for the rest of the chat. Use when the user invokes /fusion-executor (or /fusion-executor off) or asks for standing/automatic cross-model advice while working.
---

# fusion-executor

Set THIS chat's standing advisor mode. Invoked once per chat; from then on you consult the
advisor model automatically at decision points, without being asked each time. Needs no
linked group — advisors are headless CLI calls (see the fusion-advisor skill; follow its
recipes and context rules for every consult).

## Arguments

`/fusion-executor [target …]` — which advisor(s) the auto-consults use, fusion target
syntax (`claude` | `gpt` | `cursor:<model>` | `both`). Omitted → the other vendor
(Anthropic chat → `gpt`; OpenAI chat → `claude`; anything else → `both`). Same-vendor
forced → warn but obey. Multiple targets = a panel at every decision point — allowed, but
noisy.

`/fusion-executor off` — disable the mode; announce it and stop consulting.

## Behavior while ON

Consult the advisor (per the fusion-advisor skill) at these decision points:

1. A plan is drafted or approved, before implementation starts.
2. Before a risky or hard-to-reverse step.
3. After a failed review gate, or when stuck (debugging that isn't converging).
4. Task complete, before declaring it done.

Keep each consult focused (fusion-advisor's context rules). Weigh the advice openly — state
where you agree and where you differ, then proceed. Every consult is visible in the chat as
a skill invocation; that is the audit trail the user relies on.

On activation announce: "Executor mode ON — advisor: <targets>. Auto-consulting at decision
points." The mode is per-chat, not global. If the chat is very long and compaction seems to
have dropped this standing instruction, the user re-invokes `/fusion-executor`.
