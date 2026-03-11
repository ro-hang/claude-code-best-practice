# Course Sync Agent Memory

## Key File Paths
- Course modules: `/Users/rohanghiya/claude-code-best-practice/course/module-N-*.md`
- Sync log: `/Users/rohanghiya/claude-code-best-practice/course/sync-log.md`
- Source→module mapping: `.claude/skills/course-sync/SKILL.md`
- Quiz bank: `.claude/skills/course-instructor/SKILL.md`
- Upstream remote: `upstream` → `https://github.com/shanraisshan/claude-code-best-practice.git`

## Confirmed Patterns

### Files that never require module updates
- `orchestration-workflow/output.md` and `orchestration-workflow/weather.svg` — weather demo outputs, operational only
- `.claude/agent-memory/weather-agent/MEMORY.md` — temperature log, operational only
- `.claude/hooks/sounds/` (binary audio files) — asset updates, no conceptual content change

### README.md changes to watch
- New entries in the CONCEPTS table (Features section) often signal new built-in commands or features
- Each new feature row maps to the module for its category (commands → Module 1, hooks → Module 6, etc.)
- New entries in the Tips/Changelog section at bottom of README are informational only — no module update needed

### Module 1 built-in commands section structure
Commands in Concept 4 are grouped by category. New commands should be added under the most fitting category, or a new named category if genuinely new (e.g., "Scheduled tasks" for /loop, "Side conversations" for /btw).

## Sync History Summary
- 2026-03-10: Initial sync, ~60 files, modules 1/2/3/5/6/7 updated
- 2026-03-11: 8 files, Module 1 only — added /btw command
