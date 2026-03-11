---
description: Interactive Claude Code Mastery Course — learn, quiz, and build your way to agentic engineering
argument-hint: [module-number | topic | status | "quiz N" | "build N"]
model: haiku
---

# Claude Code Mastery Course

Launch an interactive learning session by invoking the `learn-agent`, which has the full curriculum preloaded.

## Argument Options

`$ARGUMENTS` tells the agent what to do:

- No argument → resume from current module in progress.json
- A number (`3`) → start Module 3
- A topic keyword (`hooks`, `skills`, `memory`, `agents`) → jump to that module
- `status` → show progress summary without starting a lesson
- `quiz N` → run just the quiz for Module N
- `build N` → run just the build exercise for Module N

## Step 1: Invoke the Learn Agent

Use the Task tool to invoke the `learn-agent`:
- `subagent_type`: `learn-agent`
- `description`: Interactive Claude Code course session
- `prompt`: The user wants: "$ARGUMENTS". (If $ARGUMENTS is empty, resume from their current progress in course/progress.json.) You have the full curriculum in your preloaded course-instructor skill. Follow it to run the session: check progress → greet → teach concepts → quiz → build exercise → save progress → preview next.

## Critical Requirements

1. **Use Task tool** — invoke `learn-agent` via Task, not bash
2. **Pass the argument** — include `$ARGUMENTS` in the prompt
3. **Let the agent run** — the learn-agent handles all interactivity via its preloaded course-instructor skill

## Output

After the agent session completes, provide a brief summary of what module was covered and what was saved to progress.json.
