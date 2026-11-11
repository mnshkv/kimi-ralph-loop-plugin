---
name: cancel-ralph
description: Cancel the active Ralph Loop in the current project — checks for the loop state file, removes it, and reports at which iteration the loop stopped. Use when the user says "cancel ralph", "/cancel-ralph", or "stop the ralph loop".
---

# Cancel Ralph

To cancel the Ralph loop in the current project:

1. Check if `.kimi/ralph-loop.local.md` exists using Bash: `test -f .kimi/ralph-loop.local.md && echo "EXISTS" || echo "NOT_FOUND"`

2. **If NOT_FOUND**: say "No active Ralph loop found."

3. **If EXISTS**:

   - Read `.kimi/ralph-loop.local.md` to get the current iteration number from the `iteration:` field
   - Remove the file using Bash: `rm .kimi/ralph-loop.local.md`
   - Report: "Cancelled Ralph loop (was at iteration N)" where N is the iteration value

Note: cancelling removes the state file, which ends the loop — the `ralph-loop` skill checks for this file's presence when deciding whether to continue. If the user wants to resume later, the only way back is starting a fresh loop (previous iterations persist in the files and git history).
