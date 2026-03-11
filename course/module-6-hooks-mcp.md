# Module 6: Hooks & MCP

**Journey position:** 85% → 95%
**Learning mode:** Build + Lab
**Time estimate:** 90–120 minutes

---

## Objectives

By the end of this module you'll be able to:
- Name all 18 hook events and when each fires
- Use hook options (`async`, `once`, `timeout`, `matcher`) correctly
- Write a `PreToolUse` hook that blocks operations using decision control
- Configure agent-scoped hooks in agent frontmatter
- Walk through the hooks.py architecture in this repo
- Choose the right browser automation MCP (Playwright vs Chrome DevTools vs Claude in Chrome)

---

## Concept 1: The 19 Hook Events

Hooks are shell commands that fire at lifecycle events — outside the agentic loop, so they don't consume Claude's context. The `hooks.py` in this repo handles all 19.

**Grouped by when they fire:**

**Session lifecycle:**
| Event | When |
|-------|------|
| `SessionStart` | New or resumed session begins |
| `SessionEnd` | Session terminates |
| `Setup` | When `--init` or `--maintenance` runs (one-time setup) |
| `InstructionsLoaded` | After CLAUDE.md and rules files are loaded into context |

**Per-turn:**
| Event | When |
|-------|------|
| `UserPromptSubmit` | User submits a message |
| `Stop` | Claude finishes responding |
| `Notification` | Claude sends a notification |

**Tool lifecycle:**
| Event | When |
|-------|------|
| `PreToolUse` | Before any tool executes |
| `PostToolUse` | After a tool succeeds |
| `PostToolUseFailure` | After a tool fails |
| `PermissionRequest` | Permission dialog appears |

**Subagent lifecycle:**
| Event | When |
|-------|------|
| `SubagentStart` | A subagent is spawned |
| `SubagentStop` | A subagent completes |

**Context management:**
| Event | When |
|-------|------|
| `PreCompact` | Before context compaction |

**Experimental / newer:**
| Event | When |
|-------|------|
| `TeammateIdle` | Agent Teams teammate goes idle |
| `TaskCompleted` | A tracked task completes |
| `ConfigChange` | A configuration file changes during session |
| `WorktreeCreate` | A git worktree is created |
| `WorktreeRemove` | A git worktree is removed |

---

## Concept 2: Hook Options

Each hook entry in settings.json supports:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",           // Regex pattern for tool name filtering
        "hooks": [
          {
            "type": "command",       // "command" (shell) or "prompt" (LLM evaluation)
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/validate.py",
            "timeout": 5000,         // Max execution time (ms)
            "async": true,           // Don't block Claude's work
            "once": true,            // Run only once per session
            "statusMessage": "Validating..."  // Spinner message while running
          }
        ]
      }
    ]
  }
}
```

**Key options:**

`async: true` — Hook runs without blocking Claude. Use for notifications, logging, sounds. Don't use when you need the hook to make a decision before Claude proceeds.

`once: true` — Hook fires only once per session. Useful for `SessionStart` (load context once, not on every resume).

`matcher` — Regex filtering for `PreToolUse`, `PostToolUse`, `PermissionRequest`:
- `"Bash"` — exact match
- `"Edit|Write"` — multiple tools (regex OR)
- `"mcp__.*"` — all MCP tools
- `"mcp__memory__.*"` — specific MCP server

**Environment variables available in hooks:**
- `${CLAUDE_PROJECT_DIR}` — project root
- `CLAUDE_TOOL_NAME` — current tool name
- `CLAUDE_TOOL_INPUT` — tool input as JSON
- `CLAUDE_TOOL_OUTPUT` — tool output (PostToolUse only)

---

## Concept 3: Decision Control — PreToolUse Can Block

`PreToolUse` hooks can actively control whether Claude proceeds:

**Exit codes:**
- `0` — success, continue
- `1` — error (logged but continues)
- `2` — **block the operation**

**For structured decision responses** (JSON to stdout):
```python
import json, sys

data = json.loads(sys.stdin.read())
tool_name = data.get("tool_name", "")
tool_input = data.get("tool_input", {})

if tool_name == "Bash":
    command = tool_input.get("command", "")
    if "rm -rf" in command:
        print(json.dumps({
            "decision": "block",
            "reason": "Refusing destructive rm -rf command"
        }))
        sys.exit(0)  # Exit 0 even when blocking — the JSON carries the decision

print(json.dumps({"decision": "allow"}))
```

**Decision values:**
- `"allow"` — proceed
- `"block"` — stop with reason (shown to Claude)
- `"ask"` — ask user for confirmation

---

## Concept 4: Agent-Scoped Hooks

Agents can have hooks in their frontmatter (6 events supported):

```yaml
---
name: weather-agent
hooks:
  PreToolUse:
    - hooks:
        - type: command
          command: "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=voice-hook-agent"
          async: true
  PostToolUse:
    - hooks:
        - type: command
          command: "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=voice-hook-agent"
          async: true
  PostToolUseFailure:
    - hooks:
        - type: command
          command: "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=voice-hook-agent"
          async: true
---
```

**All hook events are supported in agent frontmatter.** The most commonly used are:
`PreToolUse`, `PostToolUse`, `PermissionRequest`, `PostToolUseFailure`, `Stop`

**Important:** The agent's `Stop` hook fires as `SubagentStop` at runtime — a known quirk to be aware of.

**The `--agent` flag** in this repo's hooks.py: When called with `--agent=name`, the script plays from a different sound folder (`agent_pretooluse`, `agent_posttooluse`, etc.) — distinguishing main session sounds from subagent sounds.

---

## Concept 5: Walking Through hooks.py

Read `.claude/hooks/scripts/hooks.py`. It's 470 lines and covers:

**Architecture:**
- Cross-platform audio detection: macOS (`afplay`), Linux (`paplay`, `aplay`, etc.), Windows (`winsound`)
- Two sound maps: `HOOK_SOUND_MAP` (18 standard events → sound folders) and `AGENT_HOOK_SOUND_MAP` (6 agent events → agent sound folders)
- Config layering: `hooks-config.local.json` → `hooks-config.json` (personal override pattern)
- Security: Directory traversal prevention (blocks `/` and `..` in sound paths)

**Disabling hooks:**
- `hooks-config.json` has boolean flags like `disablePreToolUseHook`, `disablePostToolUseHook`
- Set to `true` to disable individual hooks without removing them from settings.json
- Create `hooks-config.local.json` for personal overrides (git-ignored)

**Git commit detection:**
```python
if "git" in tool_name.lower() and "commit" in command.lower():
    sound_name = "pretooluse-git-committing"
```
Special sound plays when committing — a nice touch that shows hooks can contain logic, not just trigger blindly.

**Logging:**
```json
{ "enableHookLogging": true }
```
Writes JSONL to `.claude/hooks/logs/hooks-log.jsonl` when enabled.

---

## Concept 6: Choosing the Right Browser MCP

All three automate browsers, but they're specialized:

| MCP | Best For | Strength | Weakness |
|-----|----------|----------|---------|
| **Playwright** | E2E testing, cross-browser | 21 tools, full automation, headless | Can't use logged-in sessions |
| **Chrome DevTools** | Performance analysis, debugging | 26 tools, network/perf/console | Requires specific Chrome setup |
| **Claude in Chrome** | Testing authenticated flows | Works with your real logged-in Chrome | 16 tools, macOS only |

**Recommendation from this repo's research:**
- **Primary:** Playwright — best coverage, cross-browser, reliable automation
- **Secondary:** Chrome DevTools — when you need performance analysis or network inspection
- **Tertiary:** Claude in Chrome — when you specifically need logged-in session testing

**Token efficiency:** Chrome DevTools MCP is most token-efficient. Claude in Chrome is least efficient (requires more DOM inspection).

---

## Lab Exercise

**Lab 1 — Add a custom hook:**
1. Open `.claude/hooks/config/hooks-config.json`
2. Note that all hooks are enabled (all `false` flags)
3. Create `.claude/hooks/config/hooks-config.local.json` and set `disablePostToolUseHook: true`
4. Verify that PostToolUse sounds stop playing while PreToolUse sounds continue

**Lab 2 — Decision control hook:**
Write a simple `PreToolUse` hook that blocks any bash command containing `force-push`:
```python
#!/usr/bin/env python3
import json, sys

data = json.loads(sys.stdin.read())
if data.get("tool_name") == "Bash":
    if "force-push" in data.get("tool_input", {}).get("command", ""):
        print(json.dumps({"decision": "block", "reason": "Force push blocked by safety hook"}))
        sys.exit(0)
print(json.dumps({"decision": "allow"}))
```
Wire it into `settings.json` under `PreToolUse` with `matcher: "Bash"`.

**Lab 3 — MCP setup:**
Install Playwright MCP and verify:
1. Add to `.mcp.json`: `{ "mcpServers": { "playwright": { "type": "stdio", "command": "npx", "args": ["@playwright/mcp@latest"] } } }`
2. Run `/mcp` and confirm it appears
3. Ask Claude to take a screenshot of a webpage using the Playwright MCP

**Completion check:**
- [ ] Can name all 18 hook events from memory (or a quick glance)
- [ ] Know when to use `async: true` vs not
- [ ] Written a PreToolUse hook with decision control
- [ ] Have hooks-config.local.json for personal overrides
- [ ] At least one browser MCP configured and tested

---

## Further Reading

- [.claude/hooks/HOOKS-README.md](../.claude/hooks/HOOKS-README.md) — Full hooks documentation for this repo
- [.claude/hooks/scripts/hooks.py](../.claude/hooks/scripts/hooks.py) — The full 470-line hook handler
- [best-practice/claude-settings.md](../best-practice/claude-settings.md) — Hooks configuration section
- [reports/claude-in-chrome-v-chrome-devtools-mcp.md](../reports/claude-in-chrome-v-chrome-devtools-mcp.md) — Browser MCP comparison with token counts
- [best-practice/claude-mcp.md](../best-practice/claude-mcp.md) — Top MCPs and setup guide

**Previous:** [Module 5 →](module-5-agents-commands.md) | **Next:** [Module 7 — Advanced Orchestration (Capstone) →](module-7-orchestration.md)
