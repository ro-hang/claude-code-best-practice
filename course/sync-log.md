# Course Sync Log

## Sync: 2026-03-11

**Upstream changes:** 8 files (1 text change, 4 binary audio assets, 3 operational/demo files)
**Modules updated:** Module 1
**Key changes:**
- Module 1: Added /btw command under new "Side conversations" category — start a side chain conversation while Claude is actively working
- Hook audio assets updated (pretooluse/posttooluse .mp3 and .wav files) — no conceptual change to hooks
- Operational files updated (weather demo output, weather-agent memory log) — no course content impact

## Sync: 2026-03-10

**Upstream changes:** ~60 files (initial upstream merge)
**Modules updated:** 1, 2, 3, 5, 6, 7
**Key changes:**
- Module 1: Expanded commands list from 40+ to 50+, added /loop scheduled tasks
- Module 2: Updated 150-line rule to 200-line rule, replaced hypothetical Rule examples with real ones from .claude/rules/
- Module 3: Reordered settings hierarchy to put managed-settings.json (MDM/plist/Registry) at top
- Module 5: Updated Task tool → Agent tool rename (v2.1.63, Task still works as alias)
- Module 6: Updated from 18 to 19 hook events (added InstructionsLoaded), all agent hook events now supported
- Module 7: Added cross-model workflow (Claude Code + Codex CLI) and /loop scheduled tasks concepts
- Quiz bank updated in course-instructor skill
