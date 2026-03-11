# Module 5: Subagents & Commands

**Journey position:** 65% → 85%
**Learning mode:** Build + Quiz
**Time estimate:** 90–120 minutes

---

## Objectives

By the end of this module you'll be able to:
- Write a complete agent definition with all 15 frontmatter fields
- Invoke agents via the Task tool (never bash)
- Understand agent memory scopes and when to use each
- Write commands that orchestrate agents and skills
- Explain the self-evolving agent pattern (presentation-curator)
- Build the Command → Agent → Skill pattern from scratch

---

## Concept 1: What Subagents Are (and Aren't)

A subagent is a **specialized Claude instance** with its own:
- Name and visual color in the CLI
- Restricted tool set (only what it needs)
- Model selection
- Preloaded skills (domain knowledge at startup)
- Permission mode
- Memory (optional, persistent across sessions)
- Lifecycle hooks

**What they're NOT:** They're not just Claude with a different system prompt. They're properly isolated instances with controlled tool access, their own memory, and lifecycle management.

**The key rule:** Subagents **cannot** invoke other subagents via bash commands. Use the `Agent` tool (renamed from `Task` in v2.1.63 — `Task(...)` still works as an alias):
```
Agent(subagent_type="agent-name", description="...", prompt="...")
```
Never `Bash("claude --agent my-agent ...")`.

---

## Concept 2: Agent Frontmatter — All Fields

Agents live in `.claude/agents/<name>.md`:

```yaml
---
name: deploy-manager                     # Unique identifier (lowercase, hyphens)
description: Use PROACTIVELY for deployment pipelines  # When to invoke
tools: Read, Write, Edit, Bash, Task(monitor)  # Tool allowlist (omit = inherit all)
disallowedTools: NotebookEdit             # Remove from allowed tools
model: sonnet                            # haiku, sonnet, opus, or inherit (default)
permissionMode: acceptEdits              # default, acceptEdits, bypassPermissions, plan
maxTurns: 25                             # Max agentic turns before stopping
skills:                                  # Preloaded skills (full content at startup)
  - deploy-checklist
  - rollback-procedures
mcpServers:                              # Agent-specific MCP servers
  - slack
  - name: pagerduty
    command: npx
    args: ["-y", "@pagerduty/mcp-server"]
hooks:                                   # Agent-scoped lifecycle hooks
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-deploy-command.sh"
  Stop:
    - hooks:
        - type: command
          command: "./scripts/notify-deploy-complete.sh"
memory: project                          # user, project, or local
background: false                        # true = always run as background task
isolation: worktree                      # "worktree" = isolated git branch
color: blue                              # CLI output color
---

You are a deployment manager. Follow the deploy-checklist skill for
pre-flight checks and use rollback-procedures if any step fails.
```

**Important fields to know deeply:**

`tools` — Be explicit. Restricting tools keeps agents fast and safe. `Task(agent_type)` syntax restricts which subagents this agent can spawn.

`description` with `"PROACTIVELY"` — This makes Claude automatically invoke the agent when relevant, without the user having to ask. Use carefully.

`isolation: worktree` — Agent runs in a temporary git worktree (isolated branch). Auto-cleaned if no changes. Essential for risky or parallel work.

---

## Concept 3: Agent Memory Scopes

Introduced in Claude Code v2.1.33. Agents can have persistent memory that survives across sessions.

| Scope | Storage Location | Shared | Use When |
|-------|-----------------|--------|----------|
| `user` | `~/.claude/agent-memory/<name>/` | No | Agent should learn across all your projects |
| `project` | `.claude/agent-memory/<name>/` | **Yes** (committed) | Agent knowledge the whole team should have |
| `local` | `.claude/agent-memory-local/<name>/` | No | Project-specific personal knowledge |

**How it works:**
1. At startup: First 200 lines of the agent's `MEMORY.md` are injected into its system prompt
2. The agent reads/writes its memory directory during execution
3. When `MEMORY.md` exceeds 200 lines, the agent moves details into topic files

**Best practice prompting for agents with memory:**
```
"Before starting, review your memory for relevant patterns. After completing,
update your memory with what you learned about this codebase's conventions."
```

**Example: weather-agent in this repo** uses `memory: project` to track weather readings across sessions.

---

## Concept 4: Built-in Agents (Official Claude Agents)

These are always available, no configuration needed:

| Agent | Model | Tools | Use For |
|-------|-------|-------|---------|
| `general-purpose` | inherit | All | Complex multi-step autonomous work |
| `Explore` | haiku | Read-only | Fast codebase searches, no write risk |
| `Plan` | inherit | Read-only | Architecture planning before code |
| `claude-code-guide` | inherit | Glob, Grep, Read, WebFetch | Questions about Claude Code itself |
| `statusline-setup` | inherit | Read, Edit | Configure status line |

Use these via Agent tool: `Agent(subagent_type="Explore", prompt="Find all API endpoints")`

---

## Concept 5: Commands — Entry Points for Workflows

Commands live in `.claude/commands/<name>.md`. They're the top-level entry points that orchestrate agents, skills, and tools.

**Frontmatter (simpler than agents):**
```yaml
---
description: Fix a GitHub issue by number     # Shows in autocomplete
argument-hint: [issue-number]                  # Autocomplete hint
allowed-tools: Read, Edit, Bash(gh *)          # Pre-approved tools for this command
model: sonnet                                  # Model for this command
---
```

**Commands vs Agents:**
- **Commands** = orchestrators. They sequence steps, ask user questions, call agents and skills.
- **Agents** = executors. They do focused work within well-defined tool boundaries.

A command is what users invoke. A command then delegates to agents which do the actual work.

**Invocation:**
- `/command-name` — from the slash menu
- `/command-name [args]` — with arguments (accessible via `$ARGUMENTS`, `$0`, `$1`)
- Subdirectory commands: `/subdir:command-name`

---

## Concept 6: The Command → Agent → Skill Pattern

This is the core orchestration pattern. Study it in the weather system:

**1. Command (`weather-orchestrator`):**
- Entry point, runs at `haiku` model (cheap)
- Asks user: C° or F°?
- Uses `Agent()` to invoke `weather-agent` with the unit preference
- Uses `Skill()` to invoke `weather-svg-creator` with the temperature result

**2. Agent (`weather-agent`):**
- Specialized, restricted tools (`WebFetch, Read, Write, Edit`)
- Has `weather-fetcher` preloaded (knows how to fetch from API)
- Returns temperature + unit to the command

**3. Skill (`weather-svg-creator`):**
- Independent, invocable
- Receives temperature from command context
- Creates SVG + writes output files

**The flow:**
```
User: /weather-orchestrator
  → Command asks: C or F?
  → Agent() → weather-agent (reads preloaded weather-fetcher skill, fetches from Open-Meteo)
  → Agent returns: 26°C
  → Skill() → weather-svg-creator (creates SVG with 26°C)
  → Command summarizes output
```

**Why this structure?**
- Command handles user interaction and orchestration
- Agent handles fetching (restricted tools, preloaded knowledge)
- Skill handles rendering (independent, reusable)
- Each layer does exactly one thing

---

## Concept 7: The Self-Evolving Agent Pattern

The `presentation-curator` agent in this repo demonstrates the most advanced pattern: an agent that updates its own knowledge after every execution.

After modifying `presentation/index.html`, the agent:
1. Reads the actual current presentation state
2. Updates its preloaded `vibe-to-agentic-framework` skill with new weight data
3. Updates its `presentation-structure` skill with new slide ranges
4. Updates its `presentation-styling` skill with any new CSS patterns

**Why this works:** Skills are just markdown files. The agent has `Write` and `Edit` tools. So it can update its own knowledge base the same way it updates any file.

**The insight:** This prevents "knowledge drift" — where an agent's preloaded knowledge becomes stale as the thing it manages evolves. The agent stays synchronized automatically.

---

## Quiz

**Q1:** Why can't subagents invoke other subagents via bash commands? What should you use instead?

**A1:** Bash commands in a subagent context run outside the Claude Code agentic loop — they don't have access to the Task/Skill tool infrastructure. The agent would be trying to spawn a new Claude process via shell, which creates a new, disconnected session. Use the `Agent()` tool (or its `Task()` alias) instead, which properly integrates with Claude Code's orchestration system.

---

**Q2:** What's the difference between `tools` and `disallowedTools` in agent frontmatter, and when would you use each?

**A2:**
- `tools` is an explicit allowlist — the agent can ONLY use the listed tools (starts from scratch)
- `disallowedTools` removes tools from the inherited set — the agent inherits all tools then removes specific ones

Use `tools` when you want strict control (security-sensitive agents). Use `disallowedTools` when you want to inherit most tools but block a few specific ones (e.g., block `NotebookEdit` if the agent should never touch notebooks).

---

**Q3:** Which memory scope should you choose for an agent that:
- Learns which code patterns your team prefers
- Should share those learnings with all teammates

**A3:** `memory: project` — stored in `.claude/agent-memory/<name>/` which is version-controlled and shared. All team members get the same accumulated knowledge.

---

**Q4:** You have a complex workflow: research a GitHub issue, plan a fix, implement it, write tests, open a PR. Should this be one command or multiple agents?

**A4:** One command orchestrating multiple agents. The command sequences the workflow. Each major phase (research, planning, implementation, testing, PR) can be a separate Agent() call to an appropriate agent. For example: `Agent(subagent_type="Explore")` for research, `Agent(subagent_type="Plan")` for planning, a custom `Agent(subagent_type="code-implementer")` for the code, a custom `Agent(subagent_type="test-writer")` for tests. The command handles sequencing and passes results between phases.

---

## Build Exercise

**Goal:** Build the Command → Agent → Skill pattern from scratch for something useful to you.

**Step 1 — Define your workflow:**
Pick something you do repeatedly (e.g., "Research a library and generate a usage example", "Review a PR and create a summary", "Fetch my calendar and summarize today").

**Step 2 — Create the skill:**
In `.claude/skills/<your-skill>/SKILL.md` — this is the reusable output step (creates a file, formats output, etc.)

**Step 3 — Create the agent:**
In `.claude/agents/<your-agent>.md` — specialized fetcher/worker. Give it:
- Restricted tools (only what it needs)
- A preloaded skill if it needs domain knowledge
- A color and description

**Step 4 — Create the command:**
In `.claude/commands/<your-command>.md` — this orchestrates:
1. Asks user for any input needed
2. `Agent()` → your agent
3. `Skill()` → your output skill
4. Summarizes results

**Step 5 — Test it:**
Run `/your-command` and verify the full flow works end-to-end.

**Completion check:**
- [ ] Command → Agent → Skill pattern working end-to-end
- [ ] Agent has restricted tools (not inheriting everything)
- [ ] At least one preloaded skill in the agent
- [ ] Command uses Agent() for agent, Skill() for the output step
- [ ] Can explain why bash-launching agents doesn't work

---

## Further Reading

- [best-practice/claude-subagents.md](../best-practice/claude-subagents.md) — Full agent frontmatter reference
- [best-practice/claude-commands.md](../best-practice/claude-commands.md) — Full commands reference
- [reports/claude-agent-memory.md](../reports/claude-agent-memory.md) — Agent memory deep dive
- [orchestration-workflow/orchestration-workflow.md](../orchestration-workflow/orchestration-workflow.md) — Weather system flow diagram
- [implementation/claude-subagents-implementation.md](../implementation/claude-subagents-implementation.md) — Implementation checklist
- [.claude/agents/presentation-curator.md](../.claude/agents/presentation-curator.md) — Self-evolving agent example

**Previous:** [Module 4 →](module-4-skills.md) | **Next:** [Module 6 — Hooks & MCP →](module-6-hooks-mcp.md)
