# kimi-ralph-loop-plugin

**Run iterative, self-referential development loops (the Ralph Wiggum technique) in [Kimi Code](https://www.kimi.com/code) — the agent keeps working the same prompt until all checks pass.**

A port of Anthropic's `ralph-loop` plugin (from the Claude Code plugin marketplace) to the Kimi Code plugin format. Original technique by Geoffrey Huntley: ["Ralph is a Bash loop"](https://ghuntley.com/ralph/) — a `while true` that feeds an agent the same prompt until the task is genuinely done.

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Platform: macOS / Windows / Linux](https://img.shields.io/badge/Platform-macOS%20%2F%20Windows%20%2F%20Linux-black.svg)]()
[![Kimi Code](https://img.shields.io/badge/Kimi%20Code-plugin-orange.svg)](https://www.kimi.com/code/docs/en/kimi-code-cli/customization/plugins.html)

## Why

Some tasks are finished not by a brilliant first answer but by relentless iteration: write code, run tests, see failures, fix, repeat. Ralph Loop automates exactly that cycle. The loop happens **inside your current session** — the same prompt is applied over and over, and each iteration sees the previous work in files and git history, so the agent self-corrects until completion criteria are met.

## Features

| Skill | What it does |
|---|---|
| `ralph-loop` | Runs the loop: reads the state file each iteration, implements the next incomplete item (PRD/checklist), runs checks until green, immediately continues to the next iteration — no waiting for you between iterations |
| `ralph-help` | Explains the technique, the skills, completion promises, and when (not) to use Ralph |
| `cancel-ralph` | Cancels an active loop by removing its state file, reporting the iteration it stopped at |

Plus:

- **Completion promises** — the loop ends only when the agent outputs `<promise>YOUR_PHRASE</promise>` *and the statement is genuinely true*, or when `--max-iterations` is reached
- **Iteration safety** — state tracked in a project-scoped `.kimi/ralph-loop.local.md` file you can inspect at any time
- **Prompt-writing reference** — best practices for loop prompts (clear criteria, incremental goals, escape hatches) bundled with the `ralph-loop` skill

## Install

In any Kimi Code session:

```
/plugins install https://github.com/mnshkv/kimi-ralph-loop-plugin
```

Then `/plugins reload` (or start a new session).

## Usage

Just ask:

- "Run a ralph loop: build a REST API for todos with CRUD, validation, and tests — completion promise DONE, max 50 iterations"
- "Ralph loop on PRD.md, iterate until every checklist item is done and `npm test` passes"
- "Keep iterating on the auth fix until all tests are green"
- "What is ralph / how does this plugin work?" (the `ralph-help` skill answers)
- "Cancel the ralph loop" (the `cancel-ralph` skill stops it)

A typical loop looks like:

```text
You:    Run a ralph loop: "Build a REST API for todos. When complete: all CRUD
        endpoints work, validation in place, tests passing, README with API docs.
        Output <promise>COMPLETE</promise>." — max-iterations 50

Agent:  🔄 Ralph loop activated (iteration 1, max 50, promise COMPLETE)
        …works on the task, runs tests, fixes failures…
        🔄 Ralph iteration 2 | checks: 3 failing → fixed, all green…
        …continues until every criterion is met…
        ✅ <promise>COMPLETE</promise> — loop finished at iteration 7
```

## How it works

```
You ──▶ ralph-loop skill ──▶ .kimi/ralph-loop.local.md   (state: iteration, max, promise, prompt)
                                  ▲                              │
                                  └──────── each iteration ──────┘
```

1. `scripts/setup-ralph-loop.sh` creates the state file with your prompt, iteration counter, limit, and completion promise.
2. Each iteration: the agent surveys its previous work in files/git, does the next unit of work, runs the checks until green, then decides — promise genuinely true? limit reached? — otherwise it increments the counter and **immediately starts the next iteration** without stopping.
3. The loop ends when the promise condition is met or the limit is hit; the state file is then removed. `cancel-ralph` ends it early by deleting the state file.

## Differences from the original

The upstream plugin (Anthropic, Apache-2.0) targets Claude Code. This port changes:

- **Stop hook is not ported.** The original enforces the loop with a `Stop` hook (`hooks/hooks.json` + `hooks/stop-hook.sh`) that blocks session exit and re-feeds the prompt. Kimi Code has no hook mechanism, so the loop discipline lives in the `ralph-loop` skill: the agent itself re-reads the state and continues iterations within one session. This also removes the need for the hook's transcript parsing (jq/perl) and per-session isolation logic.
- **Promise detection is a self-check.** Upstream scans the session transcript for `<promise>` tags; here the agent decides itself whether the promise is genuinely true, under the same "never lie to exit" rule.
- **State file moves from `.claude/` to `.kimi/`** and drops the Claude session id; the setup script is plain bash with no `claude` CLI dependency.
- **Slash commands become skills**: `/ralph-loop`, `/help`, `/cancel-ralph` → `ralph-loop`, `ralph-help`, `cancel-ralph`.
- **Prompt-writing best practices** from the upstream README are preserved as `skills/ralph-loop/references/prompt-writing.md` rather than inline docs.

What is intentionally unchanged: the core loop semantics, the completion-promise rule, the setup script's argument handling and safety messaging, and the help content.

## Requirements

- `bash` for `scripts/setup-ralph-loop.sh` (pre-installed on macOS/Linux; on Windows use Git for Windows / Git Bash)

## License & attribution

- Upstream: [ralph-loop plugin by Anthropic](https://github.com/anthropics/claude-code) (Claude Code plugin marketplace), licensed under [Apache-2.0](LICENSE).
- This port: © mnshkv, also Apache-2.0. Changes listed in *Differences from the original* above.
- Technique credit: Geoffrey Huntley — https://ghuntley.com/ralph/
