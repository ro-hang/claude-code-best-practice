---
name: course-sync
description: Knowledge for syncing upstream changes into course modules
user-invocable: false
disable-model-invocation: true
---

# Course Sync Knowledge

This skill provides the course-sync-agent with knowledge about the course structure, which source files map to which modules, and how to analyze upstream diffs for course-relevant changes.

## Source File → Module Mapping

| Source File/Directory | Affects Module(s) |
|----------------------|-------------------|
| `best-practice/claude-commands.md` | 1 (Prompting) |
| `best-practice/claude-memory.md` | 2 (Memory) |
| `best-practice/claude-settings.md` | 3 (Config) |
| `best-practice/claude-cli-startup-flags.md` | 3 (Config) |
| `best-practice/claude-mcp.md` | 3 (Config), 6 (Hooks & MCP) |
| `best-practice/claude-skills.md` | 4 (Skills) |
| `best-practice/claude-subagents.md` | 5 (Agents & Commands) |
| `reports/claude-advanced-tool-use.md` | 7 (Orchestration) |
| `reports/claude-agent-memory.md` | 5 (Agents), 7 (Orchestration) |
| `reports/claude-global-vs-project-settings.md` | 3 (Config) |
| `reports/claude-skills-for-larger-mono-repos.md` | 4 (Skills) |
| `reports/claude-in-chrome-v-chrome-devtools-mcp.md` | 6 (Hooks & MCP) |
| `reports/claude-agent-sdk-vs-cli-system-prompts.md` | 7 (Orchestration) |
| `.claude/hooks/` | 6 (Hooks & MCP) |
| `.claude/agents/`, `.claude/commands/` | 5 (Agents & Commands) |
| `.claude/skills/` | 4 (Skills) |
| `implementation/` | 5, 7 |
| `development-workflows/` | 7 (Orchestration) |
| `tips/` | 1 (Prompting) |
| `CLAUDE.md` | 0 (Orientation), all modules |

## Sync Process

1. **Pull upstream:** `git fetch upstream && git merge upstream/main`
2. **Analyze diff:** `git diff HEAD~1..HEAD --name-only` (or `git diff <before-sha>..HEAD --name-only`)
3. **Categorize changes:** Map changed files to affected modules using the table above
4. **For each affected module:**
   - Read the changed source file(s)
   - Read the current module file
   - Identify what's new, changed, or removed
   - Update the module with accurate content
5. **Update quiz bank:** If module content changed, update corresponding quiz entries in `.claude/skills/course-instructor/SKILL.md`
6. **Write sync report:** Append to `course/sync-log.md` with date, changes found, modules updated

## Change Categories to Watch For

- **New concepts/features** → Add to relevant module section
- **Renamed tools/APIs** → Find-and-replace across all modules + quiz bank
- **New hook events** → Update Module 6 event table and count
- **New CLI flags** → Update Module 3 flags table
- **New settings** → Update Module 3 settings section
- **New agents/skills/commands** → Update Module 5 or 4 as appropriate
- **Changed frontmatter fields** → Update the relevant module's frontmatter reference
- **Entirely new Claude Code layer** → May need a new module (see Module Growth Rules below)

## Module Growth Rules

Modules should stay focused and digestible. Apply these rules when deciding whether to add content to an existing module or create a new one.

### When to add to an existing module
- The new content belongs to the same **Claude Code layer** (e.g., a new CLI flag → Module 3, a new hook event → Module 6)
- The feature is a natural extension of an existing concept (e.g., `/loop` is a built-in command → Module 1)
- A feature can appear in **two modules** at different depths: introduction in the natural home module, advanced usage in Module 7. Example: `/loop` is introduced in Module 1 (built-in commands list) and explored in Module 7 (combining with agents for orchestration)

### When to split a module
- A module exceeds **~350 lines** — it's getting too long for a single learning session
- A module covers **two genuinely distinct topics** that don't share prerequisites (e.g., if Module 6 grew to cover hooks, MCP, AND browser automation in depth, browser automation could split into Module 6b)
- Quiz questions exceed **6 per module** — learner fatigue

**How to split:**
1. Identify the natural seam (usually a concept boundary)
2. Create `module-Na-topic.md` and `module-Nb-topic.md` (or renumber if cleaner)
3. Update `course/README.md` module table
4. Update `.claude/skills/course-instructor/SKILL.md` module map and quiz bank
5. Update navigation links (Previous/Next) in adjacent modules
6. Log the split in `course/sync-log.md`

### When to create a new module
Only when **all three** are true:
1. The topic represents a genuinely new **layer** of Claude Code (not just a new feature in an existing layer)
2. It has enough depth for **3+ concepts, a quiz, and a build exercise**
3. It doesn't fit naturally as a section in any existing module

**Examples that would warrant a new module:**
- Agent Teams (if it graduates from experimental) — multi-agent coordination is a new paradigm
- Claude Agent SDK (building applications) — fundamentally different from using the CLI
- CI/CD & Production Deployment — deploying Claude-powered workflows

**Examples that would NOT warrant a new module:**
- A new MCP server → Module 3 or 6
- A new agent frontmatter field → Module 5
- A new built-in command → Module 1

**How to create:**
1. Create `course/module-N-topic.md` with standard structure (Objectives → Concepts → Quiz → Build Exercise → Completion Check → Further Reading)
2. Add a row to the mapping table above for the new source files
3. Update `course/README.md` module table
4. Update `.claude/skills/course-instructor/SKILL.md` — add module to map, add quiz entries
5. Update navigation links in the previous last module
6. Log the addition in `course/sync-log.md`

### Updating dependent files after any structural change
Whenever modules are added, split, or renumbered, these files MUST be updated:
- `course/README.md` — module table
- `.claude/skills/course-instructor/SKILL.md` — module map + quiz bank
- `course/HOW-TO-LEARN.md` — module table
- Navigation links (Previous/Next) in affected module files
- `course/progress.json` schema (if module numbering changes)
