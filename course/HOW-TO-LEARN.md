# How to Learn Claude Code — Quick Start Guide

This repo contains a structured 8-module course that takes you from basic Claude Code usage to full agentic engineering mastery. Here's how to use it.

---

## Prerequisites

1. **Claude Code installed:** `brew install claude-code` (or `npm install -g @anthropic-ai/claude-code`)
2. **Logged in:** Run `claude` and authenticate with your Anthropic account
3. **This repo cloned:** `git clone https://github.com/ro-hang/claude-code-best-practice.git`

---

## Two Ways to Learn

### Option 1: Interactive (recommended)

Open Claude Code in this repo and run:

```
/learn
```

This launches an interactive instructor that:
- Walks you through concepts one at a time
- Quizzes you and evaluates your answers
- Guides you through hands-on build exercises
- Tracks your progress across sessions

**Navigate modules:**
- `/learn 0` — Start from Module 0 (orientation)
- `/learn 3` — Jump to Module 3 (configuration)
- `/learn status` — See your progress
- `/learn quiz 2` — Run Module 2's quiz
- `/learn build 5` — Start Module 5's build exercise

### Option 2: Self-paced (read the docs)

Read the module files directly in `course/`:

| Module | File | What You'll Learn |
|--------|------|-------------------|
| 0 | `module-0-orientation.md` | Repo structure, .claude/ directory, getting started |
| 1 | `module-1-prompting.md` | Writing effective prompts, @ references, /plan, /compact |
| 2 | `module-2-memory.md` | CLAUDE.md, monorepo loading, Rules, memory systems |
| 3 | `module-3-config.md` | Settings hierarchy, permissions, MCP servers, CLI flags |
| 4 | `module-4-skills.md` | Invocable vs preloaded skills, frontmatter, monorepo discovery |
| 5 | `module-5-agents-commands.md` | Subagents, commands, Command→Agent→Skill pattern |
| 6 | `module-6-hooks-mcp.md` | 19 hook events, decision control, browser automation MCPs |
| 7 | `module-7-orchestration.md` | PTC, parallel agents, cross-model workflows, capstone |

Each module has: **Concepts** → **Quiz (with answers)** → **Build Exercise** → **Completion Checklist**

---

## Recommended Path

1. **Modules 0-1** (1-2 hours): Get oriented, learn to prompt well
2. **Module 2** (1 hour): Set up CLAUDE.md for your projects — this is the highest-ROI thing you can do
3. **Module 3** (1 hour): Configure permissions and at least one MCP server
4. **Modules 4-5** (2-3 hours): Build your first skill and agent — this is where it gets powerful
5. **Module 6** (1-2 hours): Add hooks for lifecycle events
6. **Module 7** (3-6 hours): The capstone — build a complete multi-agent system

**Total time:** ~10-15 hours for the full course

---

## Quick Wins (if you're short on time)

If you only have 30 minutes, do these three things:

1. **Run `/init` in your project** — generates a CLAUDE.md that gives Claude persistent context
2. **Add `Bash(npm run *)` to your allow list** in `.claude/settings.json` — stops Claude from asking permission for every npm command
3. **Learn `/plan`** — use it before any complex task to prevent Claude from jumping straight to code

---

## Keeping the Course Updated

This repo tracks the upstream Claude Code best practices. When new features ship:

```
/sync-course
```

This pulls upstream changes and automatically updates the course modules. Run it periodically to stay current.

---

## Course Structure at a Glance

```
course/
├── HOW-TO-LEARN.md          ← You are here
├── README.md                 ← Course index with source material map
├── module-0-orientation.md
├── module-1-prompting.md
├── module-2-memory.md
├── module-3-config.md
├── module-4-skills.md
├── module-5-agents-commands.md
├── module-6-hooks-mcp.md
└── module-7-orchestration.md

.claude/
├── commands/learn.md          ← /learn command (interactive instructor)
├── commands/sync-course.md    ← /sync-course command (upstream sync)
├── agents/learn-agent.md      ← Course instructor agent
├── agents/course-sync-agent.md ← Upstream sync agent
└── skills/
    ├── course-instructor/     ← Curriculum knowledge (preloaded into learn-agent)
    └── course-sync/           ← Sync mapping knowledge (preloaded into course-sync-agent)
```

---

## Tips

- **Use `/compact` often** — especially during long learning sessions. Compact at ~50% context.
- **Build as you learn** — each module has a hands-on exercise. Don't skip them.
- **Start a fresh session per module** — prevents context bleed between topics.
- **Read the source material** — the `Further Reading` section at the bottom of each module links to the detailed reference docs.
