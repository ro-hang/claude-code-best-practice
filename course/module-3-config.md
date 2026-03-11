# Module 3: Configuration Mastery

**Journey position:** Foundational layer (enables everything else)
**Learning mode:** Reference + Quiz
**Time estimate:** 60–90 minutes

---

## Objectives

By the end of this module you'll be able to:
- Navigate the 5-level settings hierarchy and know what overrides what
- Write permission rules using the full syntax (Bash, Read, Write, WebFetch, Task, MCP)
- Know which features are global-only vs project-scoped
- Configure at least one MCP server
- Use key CLI startup flags for automation and budget control

---

## Concept 1: The Settings Hierarchy (5 Levels)

Claude Code settings are layered. Higher priority always wins. From highest to lowest:

| Priority | File | Scope | Version Controlled |
|----------|------|-------|--------------------|
| — | `managed-settings.json` (MDM plist / Registry) | **Org-enforced** | Cannot be overridden |
| 1 | CLI arguments (`--model`, `--permission-mode`) | Session only | N/A |
| 2 | `.claude/settings.local.json` | This project, personal | No (git-ignored) |
| 3 | `.claude/settings.json` | This project, team-shared | **Yes** ← commit this |
| 4 | `~/.claude/settings.local.json` | All projects, personal | No |
| 5 | `~/.claude/settings.json` | All projects, personal defaults | No |

**The critical one:** `.claude/settings.json` — check this into git. It shares your permissions, hooks, spinner customizations, and attribution settings with your team. This is Boris's Tip #12.

**Special rule on `deny`:** Deny rules have the highest safety precedence — even a lower-priority deny cannot be overridden by a higher-priority allow. Deny is truly final.

---

## Concept 2: Permissions Syntax

The permission system controls what Claude can do without asking. You define allow/ask/deny rules in `.claude/settings.json`.

**Permission modes (per session or per agent):**

| Mode | Behavior |
|------|----------|
| `default` | Standard — prompts for anything not pre-approved |
| `acceptEdits` | Auto-accepts file edits, prompts for Bash |
| `bypassPermissions` | Skip all permission checks (use for trusted scripts only) |
| `plan` | Read-only — Claude can explore but not change files |

**Tool permission syntax:**

| Tool | Pattern | Example |
|------|---------|---------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`, `Bash(git *)` |
| `Read` | `Read(path)` | `Read(.env)` to deny, `Read(src/**)` to allow |
| `Edit` | `Edit(path)` | `Edit(*.ts)`, `Edit(src/**)` |
| `Write` | `Write(path)` | `Write(*.md)` |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:*)` = all domains |
| `Task` | `Task(agent-name)` | `Task(Explore)`, `Task(my-agent)` |
| `Skill` | `Skill(skill-name)` | `Skill(weather-fetcher)` |
| `MCP` | `mcp__server__tool` | `mcp__memory__*`, `mcp__*` = all |

**Look at this repo's settings.json** (`.claude/settings.json`) — it has a real, working permission configuration:
- `allow`: Edit(*), Write(*), WebFetch, specific MCP tools, specific domains
- `ask`: Destructive bash commands (rm, rmdir), git/gh commands, package managers
- `deny`: Empty (nothing blocked entirely)

**Bash wildcard positions:**
- `Bash(npm run *)` — matches `npm run dev`, `npm run test`, etc.
- `Bash(* install)` — matches `npm install`, `pip install`, etc.
- `Bash(git * main)` — matches `git push origin main`
- `Bash(*)` — equivalent to `Bash` (all bash commands)

---

## Concept 3: Global-Only vs Project-Scoped Features

Some features only exist in `~/.claude/` — they can't be configured per-project:

**Global-only (in `~/.claude/` only):**
- **Tasks system** — cross-session task lists with dependencies
- **Agent Teams** — experimental multi-agent coordination
- **Auto-memory** — Claude's self-written project notes
- **Credentials** — API keys, OAuth tokens
- **Keybindings** — custom keyboard shortcuts

**Both global and project-scoped:**
- CLAUDE.md, settings.json, settings.local.json
- Agents (`.claude/agents/`)
- Skills (`.claude/skills/`)
- Commands (`.claude/commands/`)
- Hooks (configured in settings.json)
- MCP servers (`.mcp.json` or `settings.json`)

**Why this matters:** If you expect a feature to work project-by-project but it's global-only, you'll be confused when project-specific config doesn't affect it.

---

## Concept 4: Key Settings to Know

**Model configuration:**
```json
{
  "model": "opus",            // Override default model
  "agent": "code-reviewer",  // Set default main-session agent
  "alwaysThinkingEnabled": true  // Extended thinking by default
}
```

**Model aliases:** `"sonnet"`, `"opus"`, `"haiku"`, `"default"`, `"opusplan"` (Opus for planning, Sonnet for execution)

**Useful display/UX settings:**
```json
{
  "plansDirectory": "./reports",     // Where /plan saves plan files
  "outputStyle": "Explanatory",      // Response format (this repo uses this!)
  "spinnerVerbs": { "mode": "replace", "verbs": ["Thinking", "Brewing"] },
  "spinnerTipsOverride": { "tips": ["Run /compact at 50%"], "excludeDefault": true },
  "statusLine": { "type": "command", "command": "git branch --show-current" }
}
```

**Autocompact threshold:**
```json
{
  "env": { "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "80" }
}
```
This repo sets it to 80%. Default is ~95%. Lower values = earlier compaction = better quality over long sessions.

**Attribution:**
```json
{
  "attribution": {
    "commit": "Co-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with [Claude Code](https://claude.ai/code)"
  }
}
```

---

## Concept 5: MCP Servers

MCP (Model Context Protocol) extends Claude Code with external tools. Configure in `.mcp.json` at the project root.

**The top 4 MCPs for daily use:**

| MCP | Best For | Why |
|-----|----------|-----|
| **Context7** | Library docs | Prevents hallucinated APIs — fetches up-to-date docs |
| **Playwright** | Browser automation | E2E testing, screenshots, form filling |
| **Claude in Chrome** | Logged-in testing | Works with authenticated sessions, console/network inspection |
| **DeepWiki** | GitHub repo exploration | Structured wiki-style docs for any GitHub repo |

**Basic `.mcp.json` structure:**
```json
{
  "mcpServers": {
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

**Enable all project MCP servers in settings.json:**
```json
{ "enableAllProjectMcpServers": true }
```

**Manage MCPs interactively:** `/mcp`

**Permission naming:** MCP tools use `mcp__<server>__<tool>` — e.g., `mcp__context7__resolve-library-id`. Use `mcp__*` to allow all MCP tools.

---

## Concept 6: Key CLI Startup Flags

For automation and scripting:

| Flag | What It Does |
|------|-------------|
| `--continue` | Resume the most recent session |
| `--resume <id>` | Resume a specific session by ID |
| `--model <alias>` | Override model for this session |
| `--permission-mode <mode>` | Set permission mode (bypassPermissions, acceptEdits, etc.) |
| `--print "query"` | Non-interactive mode — runs query, prints output, exits |
| `--max-budget-usd <n>` | Cap spending for this session |
| `--max-turns <n>` | Limit number of agentic turns |
| `--agent <name>` | Set default agent for the session |
| `--debug` | Enable debug logging (see hook execution, tool details) |
| `--init` | Initialize CLAUDE.md for current project |

**Non-interactive example (CI/CD):**
```bash
claude --print "Run the test suite and report any failures" --max-budget-usd 0.50
```

---

## Quiz

**Q1:** You have `"deny": ["Bash(rm -rf *)"]` in `.claude/settings.json` and `"allow": ["Bash(rm -rf *)"]` in `.claude/settings.local.json`. Which wins?

**A1:** The `deny` rule wins — regardless of priority. Deny rules have absolute safety precedence and cannot be overridden by any allow rule at any priority level.

---

**Q2:** Which of these belongs in `.claude/settings.json` (committed) vs `.claude/settings.local.json` (personal, git-ignored)?

- Custom spinner verbs with team's branding
- Your personal API key
- Permissions to allow `Bash(npm run *)`
- Your preference to disable sound hooks

**A2:**
- ✓ Settings.json (committed): Custom spinner verbs with team branding, `Bash(npm run *)` permission
- ✓ Settings.local.json (personal): Personal API key (never commit), preference to disable sound hooks

---

**Q3:** You're at a company that wants to prevent engineers from using `bypassPermissions` mode. Where should this be configured, and which setting?

**A3:** In `managed-settings.json` (org-enforced layer, highest authority). Specifically: `"permissions": { "disableBypassPermissionsMode": "disable" }`.

---

**Q4:** What does `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50` do vs `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=95`?

**A4:** 50 triggers compaction at 50% context usage (earlier, more aggressively), preserving more quality throughout long sessions at the cost of occasional context loss. 95 waits until nearly full — you get more conversation history but risk quality degradation as context fills.

---

## Build Exercise

**Goal:** Set up your own settings.json for a real project.

1. Create `.claude/settings.json` in a project you work on
2. Configure permissions:
   - Allow your common commands without prompting (e.g., `Bash(npm run *)`, `Edit(src/**)`)
   - Set `ask` for destructive operations (e.g., `Bash(rm *)`, `Bash(git push *)`)
3. Add attribution with your team name
4. Set `plansDirectory` somewhere meaningful
5. Configure at least one MCP server (Context7 is easiest — just `npx @upstash/context7-mcp`)
6. Verify it works: run `/mcp` and confirm the server appears, run a task that uses your allowed permissions

**Completion check:**
- [ ] settings.json committed with permissions, attribution, and at least one custom setting
- [ ] At least one MCP server configured and verified working
- [ ] Can explain the 5-level hierarchy and which file wins for a given setting
- [ ] Know which features are global-only

---

## Further Reading

- [best-practice/claude-settings.md](../best-practice/claude-settings.md) — All 38 settings with examples
- [best-practice/claude-cli-startup-flags.md](../best-practice/claude-cli-startup-flags.md) — All CLI flags reference
- [best-practice/claude-mcp.md](../best-practice/claude-mcp.md) — MCP setup guide with the top 4 recommendations
- [reports/claude-global-vs-project-settings.md](../reports/claude-global-vs-project-settings.md) — Deep dive on scope and what's global-only
- [.claude/settings.json](../.claude/settings.json) — This repo's live, working settings file

**Previous:** [Module 2 →](module-2-memory.md) | **Next:** [Module 4 — Skills →](module-4-skills.md)
