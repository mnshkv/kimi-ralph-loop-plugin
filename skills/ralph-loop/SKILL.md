---
name: ralph-loop
description: Run an iterative, self-referential development loop (the Ralph Wiggum technique) inside a single session — repeatedly work the same prompt, PRD, or checklist, run the checks until they pass, and keep iterating until a completion promise is genuinely true or the iteration limit is hit. Use when the user says "run a ralph loop", "/ralph-loop", "ralph", asks to iterate until tests/checks pass, or grinds a checklist or PRD to completion.
---

# Ralph Loop

Ralph Loop implements the Ralph Wiggum technique for iterative, self-referential AI development: the **same prompt** is applied over and over, and each iteration sees the previous work in files and git history, so the agent autonomously improves until the task is genuinely done. As Geoffrey Huntley puts it: **"Ralph is a Bash loop"**:

```bash
while :; do
  cat PROMPT.md | your-coding-agent
done
```

The loop runs **inside the current session** — no external scripts, no new sessions per iteration.

## Starting a loop

When the user asks to run a ralph loop, extract from their request:

- **PROMPT** — the task description (a PRD, a checklist, or a plain task). If it names a PRD/checklist file, keep working against that file.
- `--max-iterations <n>` — stop after N iterations (default: unlimited). Always agree on a limit as a safety net.
- `--completion-promise "<text>"` — a phrase that signals genuine completion (default: none).

Then initialize the loop state by running:

```bash
bash <plugin-root>/scripts/setup-ralph-loop.sh "<prompt>" --max-iterations <n> --completion-promise "<text>"
```

(or create the state file by hand exactly as the script does). The script writes `.kimi/ralph-loop.local.md` with YAML frontmatter (`iteration`, `max_iterations`, `completion_promise`) followed by the prompt text, and prints the loop rules. If a state file already exists, ask whether to restart or resume before overwriting.

## The loop — one iteration

Each iteration follows the same phases. Do not end your turn between iterations; when an iteration finishes, start the next one immediately (see "Loop discipline" below).

1. **Read the state.** Read `.kimi/ralph-loop.local.md`: the prompt (everything after the closing `---`), the current `iteration`, `max_iterations`, and `completion_promise`.
2. **Survey previous work.** Read the relevant files, git status/diff/history. Your previous iterations persist on disk — that self-reference is the whole mechanism. Build on it; never redo finished work blindly, never regress what already works.
3. **Do the next unit of work.**
   - If the prompt is a PRD/checklist: pick the **next incomplete item** and implement only that (small, reviewable steps).
   - Otherwise: make concrete progress on the prompt.
4. **Run the checks.** Execute the project's checks — tests, linters, builds, type checks — whatever the prompt defines as "done". **Failures are data**: diagnose, fix, and re-run within the same iteration until the checks are green (or the failure is genuinely unfixable by you — see Escape hatches).
5. **Decide completion.**
   - If a completion promise is set: you may output `<promise>TEXT</promise>` **only when the statement is completely and unequivocally TRUE** — all checks green, all checklist items done. Never output a false promise to escape the loop, even if you feel stuck.
   - If `max_iterations` is reached (`max_iterations > 0` and `iteration >= max_iterations`): stop, report final status, and delete the state file.
   - If the promise condition is met: output the promise tag, report, and delete the state file.
6. **Otherwise, continue.** Increment `iteration` in the state file, print a short iteration banner, and go back to phase 1 with the SAME prompt.

```text
🔄 Ralph iteration N | To stop: output <promise>TEXT</promise> (ONLY when TRUE — do not lie to exit!)
```

## Loop discipline (replaces the original Stop hook)

In the original Claude Code plugin, a Stop hook intercepted every attempt to exit the session and fed the prompt back. **Kimi Code has no Stop hooks, so you enforce the loop yourself:**

- After phase 5–6, do **not** stop and wait for the user, do not deliver a "summary and done" message, do not declare the task finished while checks are red or items remain incomplete.
- Immediately begin the next iteration: re-read the state and continue working. Your turn continues across iterations; the loop ends only through a genuine completion promise or the iteration limit.
- Keep the state file accurate: it is the single source of truth for loop progress (the user can watch it with `head -10 .kimi/ralph-loop.local.md`).

## CRITICAL RULE

If a completion promise is set, you may ONLY output it when the statement is completely and unequivocally TRUE. Do not output false promises to exit the loop — even if you think you're stuck, the task is impossible, or you've run too long. The loop is designed to continue until genuine completion. Trust the process; if the loop should stop, the promise becomes true naturally.

## Escape hatches

Build these into the prompt when the task is risky (and honor them as the loop runner):

- Always set `--max-iterations` as the primary safety mechanism (exact string matching means a promise can't express multiple end states like "SUCCESS" vs "BLOCKED").
- If stuck after ~15 iterations: document what blocks progress, list what was attempted, and suggest alternative approaches — then keep looping within the remaining iterations rather than faking completion.

## Prompt quality

The loop is only as good as the prompt. See [references/prompt-writing.md](references/prompt-writing.md) for the full best practices (clear completion criteria, incremental goals, self-correction loops), the when-to-use / when-not guidance, and the philosophy behind the technique.
