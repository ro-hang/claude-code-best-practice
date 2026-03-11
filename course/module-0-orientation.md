# Module 0: Setup & Orientation

**Journey position:** Before 0% — laying the foundation
**Learning mode:** Quick-start (you already have basic Claude Code usage)
**Time estimate:** 30–45 minutes

---

## Objectives

By the end of this module you'll be able to:
- Explain the Vibe → Agentic spectrum and where you currently sit
- Navigate the `.claude/` directory structure and know what each part does
- Use `/doctor` to verify your installation health
- Understand how this repo is structured as a reference implementation

---

## Concept 1: The Vibe → Agentic Spectrum

Most people start as "Vibe Coders" — they type prompts, Claude responds, and it works... sometimes. The quality is inconsistent because Claude has no context about your project.

**Vibe Coding (0%):**
No project context, no conventions, no persistent knowledge. Each session Claude starts fresh. You get different results for the same prompt because Claude is guessing your intent every time.

**Agentic Engineering (100%):**
Claude knows your project architecture, coding conventions, and workflows. It has specialized agents for specific tasks, reusable skills, and hooks that automate lifecycle events. Output is consistent and production-quality.

**The insight:** You don't get to 100% in one step. This course is a progression — each module adds one layer that builds on the last.

---

## Concept 2: The `.claude/` Directory

This is the entire configuration system for Claude Code. Open the one in this repo and explore it:

```
.claude/
├── agents/          ← Custom subagent definitions (.md files)
├── commands/        ← Custom slash commands (.md files)
├── skills/          ← Reusable knowledge/workflow units (.md files)
├── hooks/           ← Event-triggered scripts
│   ├── scripts/     ← The handler code (hooks.py)
│   ├── config/      ← Hook configuration (hooks-config.json)
│   └── sounds/      ← Audio files for notifications
├── settings.json    ← Team-shared configuration (version controlled)
└── settings.local.json  ← Personal overrides (git-ignored, not in this repo)
```

**Key rule:** Everything in `.claude/` is project-scoped. It only affects Claude Code when you run it from this project directory (or a subdirectory).

**Global config** lives in `~/.claude/` — applies to all your projects.

---

## Concept 3: This Repo is a Reference Implementation

This isn't an application — it's a working demonstration of Claude Code patterns. Every file in `.claude/` is a real, working example of a concept:

| Example | What It Teaches |
|---------|----------------|
| `.claude/commands/weather-orchestrator.md` | Command → Agent → Skill pattern |
| `.claude/agents/weather-agent.md` | Agent with preloaded skill + memory |
| `.claude/skills/weather-fetcher/SKILL.md` | Preloaded (agent) skill pattern |
| `.claude/skills/weather-svg-creator/SKILL.md` | Invocable skill pattern |
| `.claude/agents/presentation-curator.md` | Self-evolving agent pattern |
| `.claude/hooks/scripts/hooks.py` | Full 18-event hook system |
| `.claude/settings.json` | Team-shared settings with all features |

When you're confused about how something works, read the corresponding file. It's live, working code.

---

## Concept 4: Built-in Commands You Should Know Now

Before diving into configuration, know these commands — they'll save you constantly:

| Command | What It Does |
|---------|-------------|
| `/doctor` | Health check — run this if anything seems broken |
| `/help` | List all available slash commands |
| `/context` | Visual grid showing current context window usage |
| `/cost` | Token usage and spend for this session |
| `/compact` | Compress conversation to free context (with optional focus instructions) |
| `/plan` | Enter read-only planning mode (no edits allowed until you exit) |
| `/model` | Switch between haiku/sonnet/opus, adjust effort level |
| `/config` | Interactive settings UI with search |
| `/memory` | View/edit all CLAUDE.md memory files |

---

## Build Exercise

**Goal:** Get oriented. Don't write anything yet — just explore.

1. Run `/doctor` and make sure everything is green
2. Open this repo in Claude Code (`cd` to the repo root, then `claude`)
3. Ask Claude: "Explain the .claude/ directory structure in this repo and what each component does"
4. Run `/context` — note how much context you're using just from asking that question
5. Read the first 20 lines of each agent file in `.claude/agents/` (just skim the frontmatter)
6. Open `orchestration-workflow/orchestration-workflow.md` and understand the weather system flow

**Completion check:**
- [ ] `/doctor` passes with no errors
- [ ] You can explain what `.claude/agents/`, `.claude/commands/`, and `.claude/skills/` are for
- [ ] You understand the Command → Agent → Skill pattern at a high level

---

## Further Reading

- [CLAUDE.md](../CLAUDE.md) — This repo's own project instructions (meta: it uses the thing it teaches)
- [orchestration-workflow/orchestration-workflow.md](../orchestration-workflow/orchestration-workflow.md) — Flow diagram of the weather system
- [best-practice/claude-commands.md](../best-practice/claude-commands.md) — Full built-in commands reference

**Next:** [Module 1 — Better Prompting →](module-1-prompting.md)
