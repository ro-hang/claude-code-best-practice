# Module 4: Skills

**Journey position:** 50% → 65%
**Learning mode:** Build + Quiz
**Time estimate:** 60–90 minutes

---

## Objectives

By the end of this module you'll be able to:
- Explain the two skill patterns and when to use each
- Write a complete skill with frontmatter covering all fields
- Understand how `context: fork` isolates skill execution
- Use string substitutions (`$ARGUMENTS`, `!`command``) for dynamic skills
- Explain how skills load in monorepos (very different from CLAUDE.md)

---

## Concept 1: The Two Skill Patterns

This is the most important distinction in the skills system. Get this right.

### Pattern A: Invocable Skill
A standalone, reusable workflow. Users can call it with `/skill-name` or a command/agent can call it with the `Skill()` tool.

**Example in this repo:** `weather-svg-creator` — called by the `weather-orchestrator` command via `Skill(skill: "weather-svg-creator")`

**When to use:** Any task you want to reuse across different contexts — code review, commit message generation, changelog formatting, screenshot capture.

### Pattern B: Agent Skill (Preloaded)
Domain knowledge injected directly into an agent's context at startup. The agent doesn't invoke it — the full content is just *there* when the agent starts.

**Example in this repo:** `weather-fetcher` — preloaded into `weather-agent` via the agent's `skills:` frontmatter field. The agent doesn't call the skill; it reads the instructions and knows how to fetch weather.

**When to use:** Specialized knowledge that a specific agent always needs — API documentation, deployment checklists, domain-specific conventions.

### The Key Difference

| | Invocable Skill | Agent Skill (Preloaded) |
|--|-----------------|------------------------|
| How accessed | `/skill-name` or `Skill()` tool | Injected at agent startup |
| `user-invocable` | `true` (default) | `false` (hidden from menu) |
| Who uses it | Anyone, any context | One specific agent |
| Content loaded | On invocation | Always, at startup |

Look at the weather system: `weather-fetcher` (preloaded) gives the agent API knowledge, while `weather-svg-creator` (invocable) is called by the orchestrating command after the agent returns temperature data. Clean separation.

---

## Concept 2: Skill Frontmatter — All Fields

Skills live in `.claude/skills/<name>/SKILL.md`. The YAML frontmatter controls behavior:

```yaml
---
name: code-review                    # Display name + /slash-command identifier
description: Review code for quality  # Shows in autocomplete, used for discovery
argument-hint: [file-path]           # Autocomplete hint when using the skill
disable-model-invocation: false      # true = prevent automatic invocation
user-invocable: true                 # false = hidden from / menu (agent use only)
allowed-tools: Read, Grep, Glob      # Tools allowed without permission prompts
model: sonnet                        # Model to use when this skill runs
context: fork                        # "fork" = runs in isolated subagent context
agent: general-purpose               # Which agent type when context: fork
hooks:                               # Lifecycle hooks scoped to this skill
  Stop:
    - hooks:
        - type: command
          command: "./scripts/log-complete.sh"
---
```

**`user-invocable: false` vs `disable-model-invocation: true`:**
- `user-invocable: false` — hides the skill from the `/` menu. Claude can still invoke it; users can't.
- `disable-model-invocation: true` — prevents Claude from automatically invoking this skill. Only works if explicitly called.

---

## Concept 3: `context: fork` — Isolated Execution

By default, skills run in the current conversation context. With `context: fork`, the skill runs in an isolated subagent — its own context, its own tools.

**When to use `context: fork`:**
- The skill does a lot of work that would bloat your conversation context
- The skill needs different permissions than the current session
- You want the skill to complete independently without affecting your conversation

**The `agent` field** (only relevant with `context: fork`) specifies which agent type runs the skill:
- `general-purpose` (default)
- `Explore` (read-only, fast)
- Any custom agent you've defined

---

## Concept 4: String Substitutions

Skills can use dynamic values:

| Variable | What It Is |
|----------|-----------|
| `$ARGUMENTS` | Everything the user typed after the skill name |
| `$ARGUMENTS[N]` | A specific argument by 0-based index |
| `$N` | Shorthand — `$0` is first argument, `$1` is second |
| `${CLAUDE_SESSION_ID}` | The current session's unique identifier |
| `` !`command` `` | Dynamic injection — runs a shell command and inserts the output |

**Dynamic injection example** — automatically includes git context:
```markdown
---
description: Review changes against main branch
---

Review the staged changes in context of the overall diff:

Current branch diff: !`git diff main...HEAD`
Staged changes: !`git diff --cached`

Focus on: security, correctness, test coverage.
```

The `` !`git diff main...HEAD` `` runs before Claude sees the prompt. The output is inserted inline.

---

## Concept 5: Skills in Monorepos — NOT Like CLAUDE.md

This is critical and frequently misunderstood:

**CLAUDE.md** uses ancestor/descendant loading based on the directory you're in.

**Skills** use a **fixed discovery model:**

1. Project-level: `.claude/skills/` (highest priority)
2. User-level: `~/.claude/skills/`
3. Plugin-level: `<plugin>/skills/`

**Nested discovery:** Skills in nested `.claude/skills/` directories are discovered when you edit files in those packages. If you have:
```
packages/
  frontend/
    .claude/skills/
      react-patterns/SKILL.md    ← discovered when editing frontend files
  backend/
    .claude/skills/
      api-conventions/SKILL.md   ← discovered when editing backend files
```

**Description loading:** Skill descriptions are always loaded (for discovery). Full skill content loads on-demand when the skill is actually invoked. This prevents skills from bloating context unnecessarily.

**Priority:** When two skills share the same name, the higher-priority location wins (project > user > plugin).

---

## Quiz

**Q1:** You want to build a "generate commit message" skill that users can run with `/commit-msg`. It should read the staged diff and suggest a message. Should it be an invocable skill or an agent skill?

**A1:** Invocable skill (`user-invocable: true`, which is the default). It's a standalone utility anyone can call from any context, not specialized knowledge for a specific agent.

**Bonus design:** Use `` !`git diff --cached` `` to inject the staged diff automatically, so users don't have to specify it.

---

**Q2:** Your `documentation-agent` always needs to know your company's API documentation style guide. Should you put this in a preloaded agent skill or an invocable skill?

**A2:** Preloaded agent skill. Add it to the agent's `skills:` field in frontmatter with `user-invocable: false`. The agent always needs this knowledge; it shouldn't be invoked on-demand. Other agents don't need it.

---

**Q3:** What does `context: fork` do when set on a skill, and when would you use it?

**A3:** It runs the skill in an isolated subagent with its own context window. Use it when the skill does extensive work (lots of file reading, web fetching) that would bloat your main conversation context, or when the skill needs different tool permissions than the current session.

---

**Q4:** You run Claude from `packages/frontend/`. You have skills in:
- `.claude/skills/` (project root)
- `packages/frontend/.claude/skills/`
- `packages/backend/.claude/skills/`

Which skills are available?

**A4:**
- ✓ `.claude/skills/` — always available (project root)
- ✓ `packages/frontend/.claude/skills/` — discovered because you're editing frontend files
- ✗ `packages/backend/.claude/skills/` — not yet discovered (you'd need to touch backend files first)

---

## Build Exercise

**Goal:** Build two skills using both patterns.

**Part 1 — Invocable Skill:**
Create a skill in `.claude/skills/commit-msg/SKILL.md` that:
- Uses `` !`git diff --cached` `` to auto-inject staged changes
- Formats commit messages in your preferred style (e.g., conventional commits)
- Is user-invocable (shows in / menu)

**Part 2 — Agent Skill (Preloaded):**
Create a preloaded skill in `.claude/skills/my-domain/SKILL.md` for a domain you work in (e.g., your API conventions, testing patterns, deployment procedures). Then:
- Create a simple agent in `.claude/agents/domain-expert.md`
- Preload the skill via the `skills:` frontmatter field
- Set `user-invocable: false` on the skill

**Part 3 — Study the weather example:**
Read both `.claude/skills/weather-fetcher/SKILL.md` and `.claude/skills/weather-svg-creator/SKILL.md` side by side. Notice: one has `user-invocable: false` (preloaded into agent), the other is standalone. Then read how the `weather-orchestrator` command uses the standalone one via `Skill()`.

**Completion check:**
- [ ] Invocable skill created and working (`/skill-name` triggers it)
- [ ] Agent skill created and preloaded into an agent
- [ ] Can explain why `weather-fetcher` is a preloaded skill and `weather-svg-creator` is invocable
- [ ] Understand `context: fork` and when to use it

---

## Further Reading

- [best-practice/claude-skills.md](../best-practice/claude-skills.md) — Full skills reference with all frontmatter fields and examples
- [reports/claude-skills-for-larger-mono-repos.md](../reports/claude-skills-for-larger-mono-repos.md) — Deep dive on discovery behavior in monorepos
- [.claude/skills/](../.claude/skills/) — All skills in this repo (live examples of both patterns)
- [implementation/claude-skills-implementation.md](../implementation/claude-skills-implementation.md) — Implementation checklist
- [Module 8: Advanced Skills](module-8-advanced-skills.md) — Production-grade skills: progressive disclosure, 5 workflow patterns, testing, troubleshooting, distribution (based on Anthropic's official guide)

**Previous:** [Module 3 →](module-3-config.md) | **Next:** [Module 5 — Subagents & Commands →](module-5-agents-commands.md)
