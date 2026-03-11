---
name: course-sync
description: Knowledge for syncing upstream changes into course modules
user-invocable: false
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
