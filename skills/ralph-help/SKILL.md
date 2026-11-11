---
name: ralph-help
description: Explain the Ralph Loop plugin — what the Ralph Wiggum technique is, how the in-session loop works, available skills (ralph-loop, cancel-ralph), completion promises, and when to use or avoid it. Use when the user asks for help with Ralph Loop, asks "what is ralph", or wants the plugin explained.
---

# Ralph Loop Plugin Help

Explain the following to the user (adapt to what they actually asked):

## What is Ralph Loop?

Ralph Loop implements the Ralph Wiggum technique — an iterative development methodology based on continuous AI loops, pioneered by Geoffrey Huntley.

**Core concept:**

```bash
while :; do
  cat PROMPT.md | your-coding-agent --continue
done
```

The same prompt is fed to the agent repeatedly. The "self-referential" aspect comes from the agent seeing its own previous work in the files and git history, not from feeding output back as input.

**Each iteration:**

1. The agent receives the SAME prompt
2. Works on the task, modifying files
3. Runs the checks (tests, linters) until green
4. Advances the iteration counter and immediately starts the next iteration
5. Sees its previous work in the files
6. Iteratively improves until completion

The technique is described as "deterministically bad in an undeterministic world" — failures are predictable, enabling systematic improvement through prompt tuning.

## Available skills

### `ralph-loop` — start a loop

```
"Run a ralph loop: <PROMPT>"           (ask for --max-iterations / --completion-promise)
"ralph-loop" with --max-iterations 20 --completion-promise "DONE"
```

1. Creates `.kimi/ralph-loop.local.md` state file (via `scripts/setup-ralph-loop.sh`)
2. The agent works on the task
3. After each iteration it re-reads the state and continues with the same prompt — inside the same session
4. Continues until the completion promise is genuinely true or max iterations is reached

### `cancel-ralph` — cancel an active loop

Removes `.kimi/ralph-loop.local.md` and reports the iteration it stopped at.

## Key concepts

### Completion promises

To signal completion, the agent outputs a `<promise>` tag:

```
<promise>TASK COMPLETE</promise>
```

The loop stops only when the promise is detected or `--max-iterations` is reached. Without either, Ralph runs forever. The promise must be **genuinely true** — the agent must never fake it to exit.

### Self-reference mechanism

The "loop" doesn't mean the agent talks to itself. It means:

- Same prompt repeated
- Work persists in files
- Each iteration sees previous attempts
- Builds incrementally toward the goal

### Differences from the original Claude Code plugin

The original used a Stop hook to block session exit and feed the prompt back. Kimi Code has no Stop hooks, so the `ralph-loop` skill carries the loop discipline itself: the agent keeps iterating within one session and only stops on a true completion promise or the iteration limit.

## When to use Ralph

**Good for:** well-defined tasks with clear success criteria; iteration/refinement (getting tests to pass); greenfield projects; tasks with automatic verification.

**Not good for:** tasks needing human judgment; one-shot operations; unclear success criteria; production debugging.

## Learn more

- Original technique: https://ghuntley.com/ralph/
- Ralph Orchestrator: https://github.com/mikeyobrien/ralph-orchestrator
