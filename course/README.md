# Claude Code Mastery Course

A structured, progressive curriculum for mastering Claude Code — from basic usage to full agentic engineering. Built from the source material in this repository.

---

## How to Learn

**Two modes:**

1. **Self-paced reading** — Work through module files directly. Each has concepts, quiz questions (with answers), and a build exercise.
2. **Interactive mode** — Run `/learn` for Claude-as-instructor: guided concept walkthroughs, live quiz sessions, and step-by-step build exercises.

**Always** do the build exercise for each module. Reading alone won't make this stick.

---

## The Journey: Vibe Coding → Agentic Engineering

```
0%  ─── No config, no context, no conventions. Every session starts cold.
                            ↓
20% ─── Better prompts + /plan. You're steering effectively.
                            ↓
40% ─── CLAUDE.md + Rules. Claude knows your project across every session.
                            ↓
50% ─── Structured workflows, model selection. Systematic approaches.
                            ↓
65% ─── Skills. Reusable, injectable knowledge units.
                            ↓
100% ── Agents + Commands + Hooks + MCP. Full agentic engineering.
```

---

## Modules

| # | Module | Mode | Build Exercise | Status |
|---|--------|------|----------------|--------|
| 0 | [Setup & Orientation](module-0-orientation.md) | Quick-start | Explore the .claude/ directory | ☐ |
| 1 | [Better Prompting](module-1-prompting.md) | Tutorial + Quiz | Write 3 before/after prompt pairs | ☐ |
| 2 | [Project Memory](module-2-memory.md) | Build + Quiz | Write a CLAUDE.md + one Rule | ☐ |
| 3 | [Configuration Mastery](module-3-config.md) | Reference + Quiz | Set up settings.json + 1 MCP | ☐ |
| 4 | [Skills](module-4-skills.md) | Build + Quiz | Build invocable + preloaded skill | ☐ |
| 5 | [Subagents & Commands](module-5-agents-commands.md) | Build + Quiz | Build Command→Agent→Skill from scratch | ☐ |
| 6 | [Hooks & MCP](module-6-hooks-mcp.md) | Build + Lab | Custom hook + MCP integration | ☐ |
| 7 | [Advanced Orchestration](module-7-orchestration.md) | Capstone | Full multi-agent system for a real use case | ☐ |
| 8 | [Advanced Skills](module-8-advanced-skills.md) | Build + Quiz | Production-grade skill with testing + patterns | ☐ |

---

## Progress Tracking

Mark modules complete by changing `☐` to `✓` in the table above.

For interactive tracking, run `/learn` — it maintains `course/progress.json` with quiz scores and completion status.

---

## Source Material Map

Every concept in this course is grounded in files that already exist in this repo:

| Topic | Primary File |
|-------|-------------|
| Commands | [best-practice/claude-commands.md](../best-practice/claude-commands.md) |
| Skills | [best-practice/claude-skills.md](../best-practice/claude-skills.md) |
| Subagents | [best-practice/claude-subagents.md](../best-practice/claude-subagents.md) |
| Memory/CLAUDE.md | [best-practice/claude-memory.md](../best-practice/claude-memory.md) |
| Settings | [best-practice/claude-settings.md](../best-practice/claude-settings.md) |
| CLI Flags | [best-practice/claude-cli-startup-flags.md](../best-practice/claude-cli-startup-flags.md) |
| MCP | [best-practice/claude-mcp.md](../best-practice/claude-mcp.md) |
| Agent Memory | [reports/claude-agent-memory.md](../reports/claude-agent-memory.md) |
| Advanced Tool Use | [reports/claude-advanced-tool-use.md](../reports/claude-advanced-tool-use.md) |
| Global vs Project Settings | [reports/claude-global-vs-project-settings.md](../reports/claude-global-vs-project-settings.md) |
| Skills in Monorepos | [reports/claude-skills-for-larger-mono-repos.md](../reports/claude-skills-for-larger-mono-repos.md) |
| Browser MCPs | [reports/claude-in-chrome-v-chrome-devtools-mcp.md](../reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| Boris's Tips | [tips/claude-boris-tips-feb-26.md](../tips/claude-boris-tips-feb-26.md) |
| Advanced Skills (Anthropic Guide) | [The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) |
| Orchestration Demo | [orchestration-workflow/orchestration-workflow.md](../orchestration-workflow/orchestration-workflow.md) |
| Live Examples | [.claude/agents/](../.claude/agents/), [.claude/skills/](../.claude/skills/), [.claude/commands/](../.claude/commands/) |
