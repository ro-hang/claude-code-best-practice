# Module 2: Project Memory

**Journey position:** 20% → 40%
**Learning mode:** Build + Quiz
**Time estimate:** 60–90 minutes

---

## Objectives

By the end of this module you'll be able to:
- Write an effective CLAUDE.md that survives the 150-line limit
- Understand exactly which CLAUDE.md files load in a monorepo (and which don't)
- Create Rules in `.claude/rules/` that multiply across every future interaction
- Distinguish between CLAUDE.md, auto-memory, Rules, and agent memory

---

## Concept 1: CLAUDE.md — Persistent Project Context

Without CLAUDE.md, Claude starts fresh every session. With it, Claude knows your project before you type anything.

**What to put in it:**

| Category | Examples |
|----------|----------|
| Architecture overview | "This is a Next.js 14 monorepo with frontend, backend, and shared packages" |
| Key file paths | "API routes are in `backend/src/routes/`, types in `shared/types.ts`" |
| Conventions | "Always use `Result<T>` for error handling, never throw" |
| Anti-patterns | "Never use `any` in TypeScript. Never commit `.env` files." |
| Testing setup | "Tests use Vitest. Run with `npm run test:watch`" |
| Workflow notes | "PRs must pass `npm run lint && npm run typecheck`" |

**The 200-line rule:** Keep CLAUDE.md under 200 lines per file. Longer files lose adherence — Claude starts ignoring parts of it. If you need more detail, use **Rules** (see Concept 3) or link to other files.

**How to generate a first draft:**
```
/init
```
This analyzes your project and generates a starting CLAUDE.md. Then edit it to be accurate.

---

## Concept 2: Monorepo Loading — Which Files Load When

This is the most misunderstood part of CLAUDE.md. There are **two distinct mechanisms:**

### Ancestor Loading (UP the tree) — Always at startup

When you run `claude` from any directory, it walks **upward** and loads every CLAUDE.md it finds, immediately.

```
Running from: /myapp/frontend/src/
Loads at startup:
  ✓ /CLAUDE.md           (root, ancestor)
  ✓ /myapp/CLAUDE.md     (ancestor)
  ✓ /myapp/frontend/CLAUDE.md  (current dir ancestor)
  ✗ /myapp/frontend/src/components/CLAUDE.md  (descendant, not yet)
```

### Descendant Loading (DOWN the tree) — Lazy, when you touch files

CLAUDE.md files in **subdirectories** only load when Claude reads files in those subdirectories.

```
Still in /myapp/frontend/src/:
  → Claude reads /myapp/frontend/src/components/Button.tsx
  → Now loads /myapp/frontend/src/components/CLAUDE.md  ✓
```

### Sibling directories — NEVER load

```
Running from /myapp/frontend/:
  ✗ /myapp/backend/CLAUDE.md   (sibling — never loads)
  ✗ /myapp/api/CLAUDE.md       (sibling — never loads)
```

**Practical implication for monorepos:**
- Root CLAUDE.md = shared conventions for the whole repo
- Component CLAUDE.md = specific rules for that component (loads lazily when needed)
- Never expect sibling package instructions to load

### Global CLAUDE.md

`~/.claude/CLAUDE.md` applies to **all** your Claude Code sessions, across all projects. Good for personal preferences that aren't project-specific.

---

## Concept 3: Rules — The Multiplier

Rules live in `.claude/rules/` and are **glob-scoped** markdown files. The first line `# Glob: <pattern>` determines which file paths trigger the rule.

**Why Rules get double weight in the curriculum:**

A single Rule encodes a convention that applies to **every future interaction** in that scope. You write it once, Claude follows it forever. CLAUDE.md is general guidance — Rules are specific enforcement.

**Real examples from this repo:**

**`.claude/rules/markdown-docs.md`** — scoped to `**/*.md` (all markdown files):
```markdown
# Glob: **/*.md

## Documentation Standards
- Keep files focused and concise — one topic per file
- Use relative links between docs, not absolute GitHub URLs
- Include back-navigation link at top of best-practice and report docs
- When adding a new concept or report, update the corresponding table in README.md
```

**`.claude/rules/presentation.md`** — scoped to `presentation/**`:
```markdown
# Glob: presentation/**

## Delegation Rule
Any request to update, modify, or fix the presentation MUST be handled by the
`presentation-curator` agent. Always delegate via the Task tool — never edit directly.
```

Every time Claude works on a markdown file, the `markdown-docs` rule fires. Every time Claude touches presentation files, it delegates to the curator agent. You never have to re-explain these conventions.

**Structure:** Rules are markdown files with a `# Glob: <pattern>` header on line 1 for path scoping. They're separate from CLAUDE.md to keep both concise and targeted.

---

## Concept 4: The Four Memory Systems (And How They Differ)

Claude Code has four memory systems. They're complementary, not duplicates:

| System | Who Writes | Who Reads | Persists | When to Use |
|--------|-----------|-----------|----------|-------------|
| **CLAUDE.md** | You (manually) | All of Claude | Always | Project conventions, architecture, key paths |
| **Rules** | You (manually) | Claude when in-scope | Always | Specific, path-scoped behavioral rules |
| **Auto-memory** | Claude (automatically) | Main Claude only | Per project | Claude's own learnings — you can read via `/memory` |
| **Agent memory** | Specific agent | That agent only | Configurable | Agents accumulating knowledge across sessions (Module 5+) |

**Auto-memory:** Claude automatically writes notes to `~/.claude/projects/<hash>/CLAUDE.md` as it learns things about your project. You can view and edit these via `/memory`. This is personal — not shared with teammates.

---

## Concept 5: CLAUDE.local.md — Personal Overrides

Create `CLAUDE.local.md` (add to `.gitignore`) for personal preferences that shouldn't be shared:

```markdown
# My personal preferences (not shared)
- I prefer explicit types over inference
- Add `// @ts-nocheck` temporarily when prototyping, I'll remove it
- Use British English in comments (I know the codebase uses American, just for my edits)
```

This sits alongside the shared CLAUDE.md and only loads for you.

---

## Quiz

**Q1:** You run `claude` from `/myapp/backend/`. Which CLAUDE.md files load immediately at startup?

```
/CLAUDE.md
/myapp/CLAUDE.md
/myapp/backend/CLAUDE.md
/myapp/backend/src/CLAUDE.md
/myapp/frontend/CLAUDE.md
/myapp/shared/CLAUDE.md
```

**A1:** Only the first three load immediately (ancestors + current directory):
- ✓ `/CLAUDE.md` — ancestor
- ✓ `/myapp/CLAUDE.md` — ancestor
- ✓ `/myapp/backend/CLAUDE.md` — current directory
- ✗ `/myapp/backend/src/CLAUDE.md` — descendant (loads lazily when Claude reads files in `src/`)
- ✗ `/myapp/frontend/CLAUDE.md` — sibling, never loads
- ✗ `/myapp/shared/CLAUDE.md` — sibling, never loads

---

**Q2:** Your CLAUDE.md is 280 lines long. What should you do?

**A2:** Trim to under 200 lines per file. Move detailed, path-specific content into Rules (`.claude/rules/`) or separate documentation files. Keep CLAUDE.md as a high-level overview with key facts. Longer files see reduced adherence — Claude starts treating later sections as lower priority.

---

**Q3:** What's the difference between CLAUDE.md and a Rule?

**A3:** CLAUDE.md is always-loaded general project context. A Rule is a specific, path-scoped behavioral instruction that fires when Claude is working in matching directories. Rules are better for precise, actionable constraints (like testing conventions) because they're separate from the general context and don't bloat CLAUDE.md.

---

**Q4:** A teammate asks "why does Claude know our API structure in my frontend sessions but doesn't know the backend conventions?" What's the explanation?

**A4:** Because she's running Claude from the `frontend/` directory. The `backend/CLAUDE.md` is a sibling directory — it never loads. The `api/CLAUDE.md` might be in `shared/` (loads lazily when files there are touched). She needs to either run from the root or add backend conventions to the root CLAUDE.md.

---

## Build Exercise

**Goal:** Give Claude persistent context for a real project.

1. Pick a real project you work on (or create a sample one)
2. Run `/init` in that project to generate a draft CLAUDE.md
3. Edit it to be accurate — add the things `/init` missed, remove the generic filler
4. Verify it's under 150 lines
5. Create one Rule file in `.claude/rules/` for a convention you repeatedly have to explain (e.g., error handling, testing, naming)
6. Start a new Claude session and verify Claude knows the convention without you explaining it

**Completion check:**
- [ ] CLAUDE.md written and under 150 lines
- [ ] Can explain monorepo loading behavior (ancestor vs descendant vs sibling)
- [ ] At least one Rule created and working
- [ ] Know the four memory systems and when to use each

---

## Further Reading

- [best-practice/claude-memory.md](../best-practice/claude-memory.md) — Full CLAUDE.md guide with monorepo diagrams
- [Humanlayer - Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md) — The definitive external guide
- [reports/claude-global-vs-project-settings.md](../reports/claude-global-vs-project-settings.md) — Auto-memory, tasks, and what's global-only

**Previous:** [Module 1 →](module-1-prompting.md) | **Next:** [Module 3 — Configuration Mastery →](module-3-config.md)
