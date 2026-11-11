# Ralph Loop: prompt writing best practices

The quality of a Ralph loop depends on the prompt, not just the model. "Deterministically bad in an undeterministic world" — failures are predictable, so tune the prompt until the loop converges.

## 1. Clear completion criteria

Bad: "Build a todo API and make it good."

Good:

```markdown
Build a REST API for todos.

When complete:
- All CRUD endpoints working
- Input validation in place
- Tests passing (coverage > 80%)
- README with API docs
- Output: <promise>COMPLETE</promise>
```

## 2. Incremental goals

Bad: "Create a complete e-commerce platform."

Good:

```markdown
Phase 1: User authentication (JWT, tests)
Phase 2: Product catalog (list/search, tests)
Phase 3: Shopping cart (add/remove, tests)

Output <promise>COMPLETE</promise> when all phases done.
```

The loop runner implements one phase per step; each iteration sees prior phases in the code.

## 3. Self-correction

Bad: "Write code for feature X."

Good:

```markdown
Implement feature X following TDD:
1. Write failing tests
2. Implement feature
3. Run tests
4. If any fail, debug and fix
5. Refactor if needed
6. Repeat until all green
7. Output: <promise>COMPLETE</promise>
```

## 4. Escape hatches

Always use `--max-iterations` as a safety net against infinite loops on impossible tasks:

```
/ralph-loop "Try to implement feature X" --max-iterations 20
```

And include in the prompt what to do when stuck:

```
After 15 iterations, if not complete:
- Document what's blocking progress
- List what was attempted
- Suggest alternative approaches
```

Note: `--completion-promise` uses exact string matching, so it cannot express multiple completion conditions (e.g. "SUCCESS" vs "BLOCKED"). Rely on `--max-iterations` as the primary safety mechanism.

## When to use Ralph

Good for:

- Well-defined tasks with clear success criteria
- Tasks requiring iteration and refinement (e.g., getting tests to pass)
- Greenfield projects where you can walk away
- Tasks with automatic verification (tests, linters)

Not good for:

- Tasks requiring human judgment or design decisions
- One-shot operations
- Tasks with unclear success criteria
- Production debugging (use targeted debugging instead)

## Philosophy

1. **Iteration > perfection** — don't aim for perfect on the first try; let the loop refine the work.
2. **Failures are data** — "deterministically bad" means failures are predictable and informative; use them to tune prompts.
3. **Operator skill matters** — success depends on writing good prompts, not just having a good model.
4. **Persistence wins** — keep trying until success; the loop handles retry logic automatically.

Real-world results reported for the technique: 6 repositories generated overnight in YC hackathon testing, a $50k contract completed for $297 in API costs, and an entire programming language built over 3 months.

## Learn more

- Original technique: https://ghuntley.com/ralph/
- Ralph Orchestrator: https://github.com/mikeyobrien/ralph-orchestrator
