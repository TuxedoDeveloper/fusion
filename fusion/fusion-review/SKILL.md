---
name: fusion-review
description: Cross-vendor code/work review — the default review gate. Headless mode has the other vendor's model (GPT via Codex, Claude via claude -p, any model via Cursor) review the current diff or plan; linked mode adversarially reviews another fusion-linked chat's work. Also consumes reviews ("fix the fusion review"). Use when the user invokes /fusion-review, asks for a cross-model review/validation of a diff, plan, or linked chat, or asks to apply/fix a fusion review.
---

# fusion-review

The default review gate: work authored by one vendor's model gets reviewed by another
vendor's. Three modes — pick by context:

- **Consume** — the user says "fix the fusion review" / "apply the fusion review".
- **Linked** — a fusion group with at least one OTHER member exists and the user didn't say
  `diff`: review that member's work.
- **Headless** — no group (or arg `diff`, or an explicit plan/file to review): drive the
  other vendor's CLI on the current diff (or the named plan file). No links needed.

## Reviewer selection (headless)

Explicit targets win: `claude` | `gpt` | `cursor:<model>` | `both`, several allowed
(e.g. `/fusion-review claude cursor:grok-4.5`). No target → strictly cross-vendor default:

| This chat's model | Default reviewer |
|---|---|
| Anthropic (Claude) | `gpt` |
| OpenAI (GPT) | `claude` |
| Anything else (Cursor third-vendor chat) | `both` |

Forcing a same-vendor review → warn that it defeats the purpose (native `/code-review`
covers same-vendor) but obey. Every `cursor:` target: print an API-cost note BEFORE running.

## Headless mode

1. Capture the material yourself: `git diff` + `git diff --staged` (mention untracked
   files), or read the named plan file. Huge diff (thousands of lines) → say so and review
   per area or narrow scope with the user.
2. Run each reviewer:
   - **gpt** — from the repo root: `codex review --uncommitted` (or `--base <branch>` /
     `--commit <sha>`; append a focus prompt if the user gave one). If the output shows the
     Windows sandbox failure (`orchestrator_helper_launch_failed` /
     `codex-windows-sandbox-setup.exe`), fall back to
     `codex exec --skip-git-repo-check --sandbox read-only -o <tmp>/review.txt "<review instructions + the diff embedded>"`
     — write the prompt to a temp file and pipe via stdin if long; never rely on Codex's
     own shell. For plan review, always the `codex exec` form with the plan text embedded.
   - **claude** — pipe the material:
     `git diff | claude -p "Adversarially review this diff: hunt real bugs, correctness, security. Findings with file:line and severity, then a verdict." --model claude-fable-5 --no-session-persistence`
     (model id rejected → retry `--model opus`).
   - **cursor:<model>** — needs the STANDALONE Cursor CLI: probe BOTH binary names
     (`cursor-agent` and `agent` — docs name it `agent`, installers ship both) and use
     whichever resolves; the IDE launcher's `cursor agent` does not forward flags
     (verified 2026-07-24). If installed:
     `git diff | <binary> -p "<review instructions>" --model <model>`. If not:
     report it and offer linked mode via a live Cursor chat (or install per
     cursor.com/docs/cli).
3. Present the findings in this chat. Multiple reviewers → consensus vs divergence, one
   merged list. Verify findings against the actual code before proposing fixes — reviewers
   can be plausibly wrong; cross-check, don't rubber-stamp.

## Linked mode

1. Read `~/.agents/fusion/links.json`; exclude this chat (quick self-ID: Claude Code
   `$CLAUDE_CODE_SESSION_ID`; Codex newest `threads` row for this cwd, `mode=ro`; Cursor
   `cursor-identity.json`). Exactly one other member → auto-target it; more → take the
   label arg or ask. **State which session you are about to review before reading it.**
2. Gather the member's work selectively (never dump full history):
   - Recent transcript turns (same per-tool readers as the fusion-merge skill: Claude Code
     jsonl / Codex rollout / Cursor bubbleId rows, tail-first).
   - The REAL changes: `git -C <member.cwd> status` + `git -C <member.cwd> diff` (+
     `--staged`), read-only. Files claimed edited outside git → read them.
3. Review adversarially: verify the member's claims against the actual files, hunt real
   bugs, check the work against its own plan if one exists. A stale/unreadable member →
   report and stop (nothing to review); never guess.
4. Output BOTH:
   - The full review in this chat (prioritized findings, file:line, severity, verdict).
   - A findings file `~/.agents/fusion/reviews/<UTC yyyyMMdd-HHmmss>-<this tool>-<target label>.md`
     containing: reviewer session (tool + id), target session (tool + id + cwd), scope
     reviewed, the prioritized findings, verdict. Then tell the user: in the reviewed chat,
     say **"fix the fusion review"** to apply it.

## Consume mode

1. List `~/.agents/fusion/reviews/*.md`, newest first. Prefer a file whose target matches
   this session (id or cwd); else take the newest and say so. None → report and stop.
2. Read it, verify each finding against the current code (it may be outdated or wrong —
   check before changing anything), fix what is real, and report finding-by-finding:
   fixed / already fine / disputed (with reasoning).

Read-only toward other tools' stores, always. Fusion writes only under `~/.agents/fusion/`.
