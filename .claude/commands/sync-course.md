---
description: Pull upstream changes and automatically update course modules
argument-hint: [--dry-run]
model: haiku
---

Pull the latest changes from the upstream repository (shanraisshan/claude-code-best-practice) and update the course modules with any new content.

## What to do

Use the Agent tool to invoke the course-sync-agent:

```
Agent(subagent_type="course-sync-agent", description="Sync upstream and update course", prompt="Pull from upstream, analyze changes, and update affected course modules. $ARGUMENTS")
```

After the agent completes, summarize:
1. How many files changed upstream
2. Which modules were updated
3. Any items needing manual review

If `$ARGUMENTS` contains `--dry-run`, tell the agent to only analyze and report — don't make any edits.
