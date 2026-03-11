# Plan: Claude Code Mastery Course — Structured Learning Curriculum

## Context

This repository is a reference implementation of Claude Code best practices, containing 50+ unique concepts across commands, agents, skills, hooks, MCP, settings, memory systems, and advanced tool use patterns. The goal is to transform this into a structured, progressive learning course for a user who has basic Claude Code usage (no config setup yet) and wants **full mastery of every concept, pattern, and edge case**.

The presentation already has the right conceptual backbone (0%→100% journey), but the repo's deeper content — 7 best-practice guides, 7 research reports, 3 implementation guides, hooks system, orchestration demos, and the tips from Boris Cherny — hasn't been organized into a learnable sequence. This plan creates that structure.

**Format**: Both markdown docs (reference, lessons, quizzes) AND an interactive `/learn` command that acts as Claude-as-instructor for each module. Module 0-1 will be compressed (user already has basic familiarity), all other modules weighted equally for depth.

---

## Course Overview: 8 Modules

The course follows the **Vibe → Agentic** arc from the presentation, extended with the deeper report content and hands-on building exercises.

| Module | Title | Learning Mode | Source Material |
|--------|-------|---------------|-----------------|
| 0 | Setup & Orientation | Tutorial | README, CLAUDE.md, presentation slides 1-9 |
| 1 | Better Prompting | Tutorial + Quiz | best-practice/claude-commands.md, slides 10-17 |
| 2 | Project Memory | Build + Quiz | best-practice/claude-memory.md, slides 18-24 |
| 3 | Configuration Mastery | Reference + Quiz | best-practice/claude-settings.md, reports/claude-global-vs-project-settings.md, claude-cli-startup-flags.md |
| 4 | Skills | Build + Quiz | best-practice/claude-skills.md, reports/claude-skills-for-larger-mono-repos.md |
| 5 | Subagents & Commands | Build + Quiz | best-practice/claude-subagents.md, best-practice/claude-commands.md, implementation/ |
| 6 | Hooks & MCP | Build + Lab | .claude/hooks/, best-practice/claude-mcp.md |
| 7 | Advanced Orchestration | Capstone Project | reports/claude-advanced-tool-use.md, orchestration-workflow/, reports/claude-agent-memory.md |

---

## Module 0: Setup & Orientation
**Goal:** Understand the Claude Code environment and this repo's purpose.

### Lessons
1. **What is Claude Code?** — REPL-based AI coding tool. Difference from ChatGPT. The Vibe→Agentic spectrum.
2. **Installation & Auth** — Homebrew install, subscription vs API key, first session
3. **The Interface** — Keybindings (Shift+Enter, /help, /clear), output styles, status line
4. **Reading This Repo** — How the .claude/ directory works, what each subdirectory means, CLAUDE.md purpose

### Deliverable
- Claude Code installed, logged in
- Clone this repo, run `/doctor`, understand the directory structure

---

## Module 1: Better Prompting
**Goal:** Write prompts that actually work. Understand context injection and planning.

### Lessons
1. **Good vs Bad Prompts** — Specific vs vague, @ file references, including relevant context
2. **Context Window Management** — What fills context, when to `/compact`, CLAUDE_AUTOCOMPACT_PCT_OVERRIDE setting
3. **Plan Mode (`/plan`)** — Why plan before code, how to use plan mode, when to exit
4. **Built-in Commands Reference** — The 40+ built-in slash commands by category (Session, Context, Model, Project, Memory, Config, Debug)
5. **Boris's Tip: Effort Levels** — Using `/model` to switch between haiku/sonnet/opus for different task sizes

### Quiz Topics
- Which @ reference loads a file vs a folder?
- When should you `/compact` vs start a new session?
- What does `/plan` prevent Claude from doing?
- Name 3 differences between a good and bad prompt

### Deliverable
- Write 3 before/after prompt pairs for a real task you have
- Practice using `/plan` on a non-trivial coding task

---

## Module 2: Project Memory
**Goal:** Give Claude persistent, project-aware context that survives session resets.

### Lessons
1. **CLAUDE.md Fundamentals** — What it is, how `/init` generates it, the 150-line rule
2. **What to Put In CLAUDE.md** — Architecture overview, conventions, "never do X", key file paths
3. **Monorepo Loading Behavior** — Ancestor loading (startup), descendant loading (lazy), sibling files never load
4. **Rules System (`.claude/rules/`)** — Modular, path-scoped conventions. Why Rules get double weight (they're multipliers — applied to every future interaction)
5. **Auto Memory** — Claude's self-written notes per project (~/.claude/projects/), personal only, never shared

### Quiz Topics
- If you're editing `frontend/src/App.tsx`, which CLAUDE.md files are loaded?
- What's the difference between CLAUDE.md and CLAUDE.local.md?
- Why should CLAUDE.md stay under 150 lines?
- What do Rules in `.claude/rules/` do differently than CLAUDE.md?

### Deliverable
- Write a CLAUDE.md for a real project you work on (or the TodoApp example)
- Create one Rule file that encodes a coding convention

---

## Module 3: Configuration Mastery
**Goal:** Understand the full settings system — permissions, hooks config, CLI flags, and global vs project scope.

### Lessons
1. **Settings Hierarchy (5 Levels)** — From CLI args (highest) → managed settings (lowest). Knowing what overrides what.
2. **Permissions Syntax** — `Bash(pattern)`, `Read(path)`, `WebFetch(domain:*)`, `Task(agent)`, `Skill(name)`. Allow vs Ask vs Deny.
3. **Global vs Project Settings** — What lives in `~/.claude/` vs `.claude/`. Tasks, Agent Teams, Auto Memory are global-only.
4. **CLI Startup Flags** — Key flags: `--continue`, `--resume`, `--model`, `--permission-mode`, `--max-budget-usd`, `--max-turns`, `--print` (non-interactive)
5. **Attribution & Custom Spinner** — Commit co-author attribution, custom spinner verbs, custom tips
6. **MCP Configuration** — `.mcp.json` at project root, stdio vs http, the top 4 MCPs (Context7, Playwright, Claude in Chrome, DeepWiki)

### Quiz Topics
- A setting in `.claude/settings.local.json` vs `.claude/settings.json` — which wins?
- What permission do you need to allow Claude to run any npm command?
- What's the difference between `--permission-mode bypassPermissions` and `--permission-mode acceptEdits`?
- Which settings are global-only (only in `~/.claude/`)?

### Deliverable
- Set up your own `.claude/settings.json` with custom permissions for a project
- Configure at least one MCP server and verify it works

---

## Module 4: Skills
**Goal:** Build reusable knowledge units that can be injected into agents or invoked on demand.

### Lessons
1. **Skill Patterns — Two Types**
   - **Invocable Skill**: User calls `/skill-name` or command uses `Skill()` tool. Standalone utility.
   - **Agent Skill (Preloaded)**: Baked into agent via `skills:` field. Domain knowledge, not invoked separately.
2. **Skill Frontmatter** — All fields: `name`, `description`, `user-invocable`, `disable-model-invocation`, `allowed-tools`, `model`, `context`, `agent`, `hooks`
3. **When to Use `context: fork`** — Isolated subagent context for skills that need to run independently
4. **String Substitutions** — `$ARGUMENTS`, `$N`, `${CLAUDE_SESSION_ID}`, `` !`command` `` for dynamic injection
5. **Skills in Monorepos** — Discovery is NOT like CLAUDE.md. Fixed locations, lazy full-content loading, how nested `.claude/skills/` directories work
6. **The `agent-browser` Skill** — Real example: navigation, snapshots, interactive refs, parallel sessions, mobile

### Quiz Topics
- What's the difference between `user-invocable: false` and `disable-model-invocation: true`?
- How do you preload a skill into a specific agent?
- In a monorepo, if you're editing `packages/frontend/src/App.tsx`, which skills are available?
- When should you use `context: fork`?

### Deliverable
- Create an invocable skill for a task you repeat often (e.g., code review, commit message generation)
- Create an agent skill (preloaded knowledge) for a domain you work in
- Study the `weather-fetcher` vs `weather-svg-creator` split and understand why they're separated

---

## Module 5: Subagents & Commands
**Goal:** Build specialized agents that work on focused tasks, and commands that orchestrate them.

### Lessons
1. **Subagent Frontmatter (20+ fields)** — `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `isolation`, `color`
2. **Tool Allowlists** — Be explicit. Restricting tools keeps agents fast and safe. `Task(agent_type)` syntax.
3. **The Task Tool** — How to invoke agents programmatically. Never use bash to launch agents.
4. **Agent Memory Scopes** — `user` (`~/.claude/agent-memory/`), `project` (`.claude/agent-memory/`), `local` (`.claude/agent-memory-local/`). First 200 lines injected at startup.
5. **Custom Commands** — Entry points for workflows. Frontmatter: `description`, `argument-hint`, `allowed-tools`, `model`. Commands vs Agents: commands orchestrate, agents execute.
6. **Built-in Agents** — `general-purpose`, `Explore`, `Plan`, `claude-code-guide`, `statusline-setup`
7. **The Self-Evolving Agent Pattern** — `presentation-curator` example: after executing, updates its own preloaded skills to prevent knowledge drift

### Quiz Topics
- Why can't subagents invoke other subagents via bash?
- What's the difference between `tools` and `disallowedTools` in agent frontmatter?
- When should you use `permissionMode: bypassPermissions`?
- What are the three agent memory scopes and where do files live?
- What's the difference between a Command and an Agent?

### Deliverable
- Build the **Command → Agent → Skill** pattern from scratch using the weather system as a template
- Create a custom agent with restricted tools and a preloaded skill
- Write a command that orchestrates 2 agents in parallel using the Task tool

---

## Module 6: Hooks & MCP
**Goal:** Automate responses to Claude's lifecycle events and connect to external tools.

### Lessons
1. **The 18 Hook Events** — Full list with when each fires. Key ones: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `SubagentStart/Stop`, `SessionStart/End`, `PreCompact`, `ConfigChange`, `WorktreeCreate/Remove`
2. **Hook Options** — `async`, `once`, `timeout`, `statusMessage`, `matcher` (filter by tool/event type)
3. **Decision Control** — How `PreToolUse` hooks can `allow`/`deny`/`ask` tool calls
4. **Agent-Scoped Hooks** — 6 hooks supported in agent frontmatter. Agent Stop → SubagentStop conversion. Used in `weather-agent`.
5. **The Hooks Sound System** — Walking through `hooks.py`: cross-platform audio, `HOOK_SOUND_MAP`, `AGENT_HOOK_SOUND_MAP`, git commit detection, config fallback pattern
6. **MCP Servers Deep Dive** — Scopes (project > user > subagent), stdio vs http, permission naming (`mcp__<server>__<tool>`), report: Playwright vs Chrome DevTools vs Claude in Chrome comparison
7. **`disableAllHooks` and Individual Hook Disabling** — `hooks-config.json` pattern for team-shared with personal override

### Lab Exercise
- Add a new hook event to `hooks-config.json` and wire it to a custom sound
- Write a `PreToolUse` hook that blocks `rm -rf` with a decision control response
- Set up the Playwright MCP and use it to screenshot a webpage

### Deliverable
- Extend the hooks system with a custom event handler
- Configure a local MCP server and write a skill that uses it

---

## Module 7: Advanced Orchestration (Capstone)
**Goal:** Master the deepest patterns in Claude Code — programmatic tool calling, parallel agent teams, agent memory, and production-grade workflow design.

### Lessons
1. **Programmatic Tool Calling (PTC)** — Claude writes Python to orchestrate tools in a single inference. ~37% token reduction. When to use vs sequential tool calls.
2. **Dynamic Web Filtering** — Claude writes code to filter search results before loading into context (~24% fewer tokens). Tool Search Tool for deferred loading (~85% reduction).
3. **Tool Use Examples** — Input examples in tool schemas (72%→90% accuracy improvement). How to write them.
4. **Agent Teams (Experimental)** — Multiple agents coordinating on shared work. Global-only feature.
5. **Git Worktrees for Agent Isolation** — `isolation: worktree` in agent frontmatter. Agents work in temp branches, auto-cleaned.
6. **The RPI Workflow** — Research → Plan → Implement. Multi-agent workflow in `development-workflows/rpi/`.
7. **SDK vs CLI System Prompts** — What the CLI sends (~269 base tokens + 110+ conditional components) vs SDK minimal default. Why there's no determinism guarantee even at temperature=0.
8. **Workflow Audit Commands** — The `workflow-concepts` and `workflow-claude-subagents` commands: how they use 2 parallel agents to detect documentation drift, mandatory changelog appending, verification checklists.

### Capstone Project
Build a complete multi-agent system for a real use case:
- A **command** as orchestrator
- At least **2 agents** (one with preloaded skills, one parallel)
- **Agent memory** that persists across sessions
- **Hooks** for lifecycle events
- A **verification workflow** that audits your system's accuracy over time

---

## Learning Modalities Per Module

| Module | Read | Build | Quiz | Lab |
|--------|------|-------|------|-----|
| 0 | ✓ | ✓ | | |
| 1 | ✓ | ✓ | ✓ | |
| 2 | ✓ | ✓ | ✓ | |
| 3 | ✓ | ✓ | ✓ | |
| 4 | ✓ | ✓ | ✓ | |
| 5 | ✓ | ✓ | ✓ | |
| 6 | ✓ | ✓ | | ✓ |
| 7 | ✓ | ✓ | | ✓ (Capstone) |

---

## Critical Source Files Per Module

| Module | Primary Files |
|--------|---------------|
| 0 | `README.md`, `CLAUDE.md`, `presentation/index.html` (slides 1-9) |
| 1 | `best-practice/claude-commands.md`, `tips/claude-boris-tips-feb-26.md`, slides 10-17 |
| 2 | `best-practice/claude-memory.md`, slides 18-24 |
| 3 | `best-practice/claude-settings.md`, `best-practice/claude-cli-startup-flags.md`, `reports/claude-global-vs-project-settings.md`, `best-practice/claude-mcp.md` |
| 4 | `best-practice/claude-skills.md`, `reports/claude-skills-for-larger-mono-repos.md`, `.claude/skills/` |
| 5 | `best-practice/claude-subagents.md`, `best-practice/claude-commands.md`, `implementation/`, `.claude/agents/`, `.claude/commands/` |
| 6 | `.claude/hooks/`, `best-practice/claude-mcp.md`, `reports/claude-in-chrome-v-chrome-devtools-mcp.md` |
| 7 | `reports/claude-advanced-tool-use.md`, `reports/claude-agent-memory.md`, `orchestration-workflow/`, `reports/claude-agent-sdk-vs-cli-system-prompts.md`, `development-workflows/rpi/` |

---

## Verification: How to Measure Progress

- **Module 0**: `/doctor` passes, can navigate .claude/ structure
- **Module 1**: Can articulate why a prompt failed, uses /plan on complex tasks
- **Module 2**: CLAUDE.md for real project written, understands monorepo loading rules
- **Module 3**: Custom settings.json committed, at least 1 MCP configured
- **Module 4**: Built invocable + preloaded skill, understands the two patterns
- **Module 5**: Command→Agent→Skill working end-to-end from scratch
- **Module 6**: At least 1 custom hook and 1 MCP integration working
- **Module 7**: Full multi-agent system built for real use case

---

## Implementation Plan

### What to Create

**1. Course Directory (`course/`)**
```
course/
├── README.md             ← Course index, progress tracker, how to use /learn
├── module-0-orientation.md
├── module-1-prompting.md
├── module-2-memory.md
├── module-3-config.md
├── module-4-skills.md
├── module-5-agents-commands.md
├── module-6-hooks-mcp.md
└── module-7-orchestration.md
```
Each module file contains: Learning objectives → Concepts → Quiz questions (with answers) → Build exercise → Completion checklist.

**2. Interactive `/learn` Command (`.claude/commands/learn.md`)**
A command that accepts a module number or topic as argument. When invoked:
- Presents the module's concepts interactively
- Asks quiz questions one at a time, evaluates answers
- Guides the user through the build exercise step by step
- Tracks progress via a `course/progress.json` file (completed modules, quiz scores)
- Can resume where the user left off

**3. Course Skill (`.claude/skills/course-instructor/SKILL.md`)**
Preloaded knowledge for the `/learn` command — the full curriculum map, quiz bank, build exercise instructions. Allows the /learn command to be model-powered without re-reading all source files each time.

### Files to Create (in order)
1. `course/README.md` — course index
2. `course/module-0-orientation.md` through `course/module-7-orchestration.md` — 8 module files
3. `.claude/skills/course-instructor/SKILL.md` — curriculum knowledge skill
4. `.claude/commands/learn.md` — interactive learning command

### Files to Read (no changes needed)
All source material in `best-practice/`, `reports/`, `implementation/`, `tips/`, `orchestration-workflow/`, `.claude/agents/`, `.claude/skills/`, `.claude/hooks/`

### Notes
- Module 0 will be a quick-start (user has basic familiarity, skip install basics)
- Boris's tips from `tips/claude-boris-tips-feb-26.md` woven into relevant modules
- The presentation (`presentation/index.html`) referenced as visual companion
- The weather orchestration system is the hands-on template for Module 5's build exercise
- `course/progress.json` tracks completed modules + quiz scores across sessions
