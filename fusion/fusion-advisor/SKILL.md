---
name: fusion-advisor
description: One-shot cross-vendor advisor consult — ask another frontier model (GPT via Codex, Claude via claude -p, any model via Cursor) a question or for guidance, headlessly, right now. Use when the user invokes /fusion-advisor, asks for a second opinion from another model, or when fusion-executor mode triggers a consult.
---

# fusion-advisor

Consult another vendor's frontier model ONCE, headlessly, and bring the advice into this
chat. No linked group needed. The skill invocation itself is the audit trail — the user sees
in the transcript that an advisor was called.

## Arguments

`/fusion-advisor [target …] [question]`

- Targets: `claude` | `gpt` | `cursor:<model>` | `both` (= claude + gpt); several targets
  = a panel.
- Omitted → the other vendor: Anthropic chat → `gpt`; OpenAI chat → `claude`; any other
  model → `both`. (You know your own model from your system prompt.)
- Forcing a same-vendor advisor → warn that it defeats cross-vendor consulting, but obey.
- No explicit question → derive it from the current decision point (what you are about to
  do, what you are unsure about).

## Build the consult prompt

Package MINIMAL context — the advisor is smart, not psychic, and dumping history is an
anti-goal. Aim under ~2000 words:

1. One paragraph of project/task context.
2. The concrete question or decision, with the options you are weighing.
3. Only the excerpts that matter (plan section, error output, key constraints, short diff).
4. Ask for: a recommendation, the reasoning, and what the advisor would check that you
   might have missed.

Long prompts: write to a temp file and pipe via stdin instead of an argument (Windows
command-length limits).

## Recipes (per target)

- **gpt** — `codex exec --skip-git-repo-check --sandbox read-only -o <tmp>/advice.txt "<prompt>"`
  then read `advice.txt` (cleaner than parsing the banner). Never rely on Codex's own shell
  inside the call — all context goes in the prompt (Windows sandbox helper is flaky).
- **claude** — `claude -p "<prompt>" --model claude-fable-5 --no-session-persistence`; if
  the model id is rejected, retry with `--model opus`.
- **cursor:<model>** — Cursor is API-priced: print a cost note BEFORE running anything.
  Needs the STANDALONE Cursor CLI — probe BOTH binary names (`cursor-agent` and `agent`;
  the docs name it `agent`, installers ship both) and invoke whichever resolves. The IDE
  launcher's `cursor agent` subcommand does not forward flags (verified 2026-07-24).
  If present: `<binary> -p "<prompt>" --model <model>`. If absent: report the two options —
  install the Cursor CLI (cursor.com/docs/cli), or consult via a live linked Cursor chat
  instead.

## Present the advice

- Single advisor: the advice, then YOUR position — where you agree, where you differ, and
  why. Cross-vendor disagreement is signal; never rubber-stamp.
- Panel: consensus vs divergence, then your synthesis and the decision you take.
