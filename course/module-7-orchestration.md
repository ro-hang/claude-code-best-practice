# Module 7: Advanced Orchestration (Capstone)

**Journey position:** 95% → 100%
**Learning mode:** Deep dive + Capstone Project
**Time estimate:** 3–6 hours (this is the big one)

---

## Objectives

By the end of this module you'll be able to:
- Explain Programmatic Tool Calling (PTC) and why it reduces tokens by ~37%
- Use dynamic web filtering and the Tool Search Tool concepts
- Design multi-agent systems with parallel execution
- Use git worktrees for agent isolation
- Understand the RPI (Research → Plan → Implement) workflow pattern
- Know what the CLI sends vs the SDK (and why there's no determinism guarantee)
- Set up a cross-model workflow (Claude Code + Codex CLI) for plan/QA/implement/verify
- Use `/loop` for scheduled recurring tasks
- Build a complete multi-agent system for a real use case you care about

---

## Concept 1: Programmatic Tool Calling (PTC)

This is an API-level feature that changes how Claude uses tools. Instead of a round-trip per tool call, Claude **writes Python that calls all the tools it needs** in a single inference pass.

**Traditional (N tools = N round trips):**
```
User prompt → Claude → tool_call_1 → result_1 → Claude → tool_call_2 → result_2 → Claude → answer
```

**PTC (N tools = 1 round trip):**
```
User prompt → Claude → writes Python script → script calls tool_1, tool_2, tool_3 → stdout → Claude → answer
```

**Why this matters:** Intermediate tool results go into the code's memory, not Claude's context. Only `stdout` at the end enters the context window. For 10 tools: ~90% fewer tokens consumed on intermediate results.

**Example — batch processing across 5 regions in 1 inference:**
```python
regions = ["West", "East", "Central", "North", "South"]
results = {}
for region in regions:
    data = await query_database(f"SELECT SUM(revenue) FROM sales WHERE region='{region}'")
    results[region] = data[0]["revenue"]

top = max(results.items(), key=lambda x: x[1])
print(f"Top region: {top[0]} with ${top[1]:,}")
```
Without PTC: 5 round trips (5 model inferences). With PTC: 1 round trip (1 model inference, 5 tool calls from inside the script).

**For Claude Code users:** PTC is API-only — not available in the CLI. But it's directly relevant if you build agents with the Anthropic SDK (e.g., `@anthropic-ai/claude-agent-sdk`).

**Config (API level):**
```json
{
  "tools": [
    { "type": "code_execution_20250825", "name": "code_execution" },
    {
      "name": "query_database",
      "allowed_callers": ["code_execution_20250825"]  // callable from Python only
    }
  ]
}
```

---

## Concept 2: Dynamic Web Filtering + Tool Search Tool

Two more API-level patterns for token efficiency:

**Dynamic Web Filtering (~24% fewer tokens):**
Instead of dumping full web pages into context, Claude writes Python to filter/extract relevant content before it enters the context window.

```
Before: Search → Full HTML × N pages → All content in context
After:  Search → Claude writes filter code → Only relevant content in context
```

**BrowseComp benchmark results:**
- Sonnet 4.6: 33.3% → 46.6% accuracy with filtering
- Opus 4.6: 45.3% → 61.6% accuracy with filtering

**Tool Search Tool (~85% fewer definition tokens):**
With 50+ MCP tools at ~1.5K tokens each = 75K tokens before a user even asks a question. Mark infrequently-used tools with `defer_loading: true`. Claude discovers them on-demand.

**Claude Code equivalent:** MCPSearch auto mode (enabled by default since v2.1.7). When MCP tool descriptions exceed 10% of context, they're deferred. Tune with `ENABLE_TOOL_SEARCH=auto:N`.

**Tool Use Examples (72% → 90% accuracy):**
Add `input_examples` to tool definitions — concrete usage patterns that help Claude understand parameter combinations:
```json
{
  "name": "create_ticket",
  "input_examples": [
    { "title": "Login page 500 error", "priority": "critical", "assignee": "oncall" },
    { "title": "Add dark mode", "priority": "low", "labels": ["feature"] }
  ]
}
```

---

## Concept 3: Parallel Agent Execution

The workflow commands in this repo demonstrate parallel execution. Look at `workflow-concepts.md`:

```markdown
## Phase 0: Launch 2 agents in parallel

Task(subagent_type="workflow-concepts-agent", prompt="Research drift...")
Task(subagent_type="claude-code-guide", prompt="Research latest features...")
```

Both Task() calls fire simultaneously. Results arrive async. The command then merges findings.

**When to parallelize:**
- Independent research tasks (searching different sources)
- Processing different components of a codebase simultaneously
- Running checks that don't depend on each other

**When NOT to parallelize:**
- Agent B needs results from Agent A
- Agents would conflict writing to the same files
- Sequential dependency exists

**Git worktrees for parallel safety:**
```yaml
---
isolation: worktree
---
```
Each agent gets a temporary git branch. Agents can write code without conflicting. Auto-cleaned if no net changes.

---

## Concept 4: The RPI Workflow (Research → Plan → Implement)

Look at `development-workflows/rpi/`. This is a multi-agent workflow for complex implementation tasks:

1. **Research phase:** `Task(Explore)` agents gather information about the codebase and requirements
2. **Plan phase:** `Task(Plan)` agent designs the implementation approach, writes a plan file
3. **Implement phase:** Implementation agents work from the approved plan

**Why this structure works:**
- Research is read-only (safe, fast)
- Planning is read-only (human-approved before code changes)
- Implementation has a clear spec to follow

**The plan file as contract:** The plan file written in Phase 2 is the interface between planning and implementation. Implementation agents receive the plan as input — they don't re-research, they execute.

---

## Concept 5: SDK vs CLI — What Claude Actually Receives

This matters if you're building tools or trying to understand Claude's behavior:

**CLI (Claude Code) sends:**
- ~269 base tokens in system prompt
- 110+ conditionally-loaded components (tools, memory files, project context)
- CLAUDE.md files (all loaded ancestors + lazy descendants)
- Tool definitions for all configured tools

**SDK (minimal):**
- Just what you pass in `system` parameter
- No automatic CLAUDE.md loading
- No built-in tools unless you configure them

**Why there's no determinism guarantee even at temperature=0:**
- Floating-point arithmetic differences across hardware
- Mixture-of-Experts (MoE) routing — models don't always route the same tokens to the same experts
- Infrastructure variations (different GPUs, different batching)

**Practical implication:** Don't rely on identical outputs for identical inputs in production systems. Design for variance.

---

## Concept 6: Workflow Audit Commands — The Meta-Pattern

The `workflow-concepts` and `workflow-claude-subagents` commands in this repo demonstrate a production pattern: **automated documentation drift detection.**

**The pattern:**
1. Two agents run in parallel — one researches external sources, one checks local docs
2. Merge findings and cross-reference
3. Mandatory append to changelog (never overwrite)
4. Update "Last Updated" badge
5. Validate all hyperlinks
6. Present findings with NEW/RECURRING/RESOLVED status

**Why mandatory appending matters:** If you overwrite, you lose historical drift patterns. By appending, you can see: "This missing concept was flagged NEW in February, RECURRING in March — we need to actually fix it."

**Verification checklists:** The workflow commands maintain verification checklists (if they exist). When discovering new drift types, they append new rules to the checklist. The system becomes more thorough over time.

---

## Concept 7: Cross-Model Workflows (Claude Code + Codex CLI)

An advanced orchestration pattern that uses **two different AI models** across terminals for complementary strengths:

**The 4-step workflow:**
1. **Plan (Claude Code, Opus 4.6):** Open in plan mode. Claude interviews you via AskUserQuestion. Produces a phased plan with test gates → `plans/{feature-name}.md`
2. **QA Review (Codex CLI, GPT-5.4):** Second terminal. Codex reviews the plan against the actual codebase. Inserts intermediate phases ("Phase 2.5") with "Codex Finding" headings. Adds to the plan — never rewrites original phases.
3. **Implement (Claude Code, Opus 4.6):** New session. You implement phase-by-phase with test gates at each boundary.
4. **Verify (Codex CLI, GPT-5.4):** New Codex session. Verifies implementation against the plan.

**Why cross-model?** Different models have different strengths — Claude for planning and implementation, GPT for adversarial review. The plan file is the contract between them. Each model can only *add to* the plan, never rewrite. This creates a layered review system.

See: `development-workflows/cross-model-workflow/cross-model-workflow.md`

---

## Concept 8: Scheduled Tasks with `/loop`

For recurring automated work during a session:

```
/loop 1m "tell current time"
/loop 5m /simplify
/loop 10m "check deploy status"
```

**How it works:**
- Cron-based: `1m` → `*/1 * * * *`. Minimum granularity is 1 minute.
- Auto-expire after **3 days**
- Session-scoped — jobs stop when Claude exits
- Cancel with `cron cancel <job-id>`
- Each iteration triggers async hooks (UserPromptSubmit, Stop)

**Use cases:** Periodic monitoring (deploy status), recurring code review sweeps, regular data fetching. Combine with agents for powerful automation: `/loop 30m "run the test suite and report failures"`.

See: `implementation/claude-scheduled-tasks-implementation.md`

---

## Capstone Project

Build a complete multi-agent system for something you actually care about. This is the test of Module 5-7 combined.

**Requirements:**

**Architecture (required):**
- [ ] A command as the entry point and orchestrator
- [ ] At least 2 agents (at least one with restricted tools and a preloaded skill)
- [ ] At least 1 parallel Task() execution
- [ ] Agent memory that persists across sessions (choose appropriate scope)
- [ ] Hooks for at least 1 lifecycle event

**Documentation (required):**
- [ ] A flow diagram (text is fine) showing Command → Agents → Skills
- [ ] A CLAUDE.md for the system explaining what it does
- [ ] A README explaining how to invoke it and what it produces

**Verification (required):**
- [ ] Run the system end-to-end at least 3 times, note any inconsistencies
- [ ] Write a short audit report on what worked and what didn't

**Example systems that would satisfy requirements:**
- A PR review system: Command → Research agent (reads PR diff) + Test agent (runs tests in parallel) → Summary skill
- A codebase audit system: Command → 3 parallel Explore agents (different areas) → Findings skill
- A content pipeline: Command → Research agent → Writer agent → Format skill

**Bonus challenge (optional):**
Make the system self-auditing — add a verification command (like `workflow-concepts`) that checks whether the system's documentation is still accurate compared to its implementation.

---

## Advanced Topics Reference

These topics are beyond the core curriculum but worth knowing about:

**Agent Teams (experimental, Feb 2026):**
Multiple agents coordinating on shared work. Currently global-only (configured in `~/.claude/`). Experimental — APIs may change.

**Background agents (`background: true`):**
Agent always runs as a background task. Check on it with `/tasks`. Useful for long-running operations you don't want to wait for.

**Claude Code vs Agent SDK:**
- Claude Code CLI: Everything in this course. Best for interactive development.
- Agent SDK (`@anthropic-ai/claude-agent-sdk`): For building applications that use Claude agents. PTC, dynamic filtering available here.

**Usage commands:**
- `/usage` — plan limits and rate limit status (subscription plans)
- `/extra-usage` — enable pay-as-you-go overflow at API rates ($2K/day cap)
- `/cost` — session spending (API users)
- `--max-budget-usd` CLI flag — hard cap on session spending

---

## Further Reading

- [reports/claude-advanced-tool-use.md](../reports/claude-advanced-tool-use.md) — PTC, dynamic filtering, Tool Search, Tool Use Examples
- [reports/claude-agent-memory.md](../reports/claude-agent-memory.md) — Agent memory deep dive
- [reports/claude-agent-sdk-vs-cli-system-prompts.md](../reports/claude-agent-sdk-vs-cli-system-prompts.md) — What CLI vs SDK sends
- [development-workflows/rpi/](../development-workflows/rpi/) — RPI workflow implementation
- [.claude/commands/workflows/](../.claude/commands/workflows/) — Audit workflow commands (meta-pattern)
- [reports/claude-usage-and-rate-limits.md](../reports/claude-usage-and-rate-limits.md) — Usage, billing, rate limits

**Previous:** [Module 6 →](module-6-hooks-mcp.md) | **Next:** [Module 8 — Advanced Skills →](module-8-advanced-skills.md)

---

**Congratulations — you've reached 100% Agentic Engineering.**

The journey from "type prompts and hope" to "structured multi-agent systems with persistent memory, automated hooks, and production-grade orchestration" is complete. The capstone project is your proof.

**Want to go deeper on skills?** Module 8 covers production-grade skill engineering — progressive disclosure, workflow patterns, testing, troubleshooting, and distribution — based on Anthropic's official skills guide.
