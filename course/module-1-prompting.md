# Module 1: Better Prompting

**Journey position:** 0% → 20%
**Learning mode:** Tutorial + Quiz
**Time estimate:** 45–60 minutes

---

## Objectives

By the end of this module you'll be able to:
- Write prompts that give Claude the right context upfront
- Use @ references, /plan, and /compact effectively
- Choose the right model/effort for different task sizes
- Understand the 40+ built-in slash commands by category

---

## Concept 1: Good vs Bad Prompts

The single biggest lever you have before any configuration is prompt quality.

**Bad prompt:**
> "Fix the bug"

**Why it fails:** Claude doesn't know which bug, which file, what the expected behavior is, or what constraints to follow.

**Good prompt:**
> "The `calculateTotal()` function in @src/cart.ts is returning NaN when any item has a null price. It should treat null prices as 0. Don't change the function signature."

**The pattern:** `[What file/function] + [What's wrong] + [What correct behavior looks like] + [Constraints]`

**@ References — inject files into context:**
- `@filename.ts` — loads a specific file
- `@src/components/` — loads a directory
- `@git:diff` — loads the current git diff
- `@url` — fetches and loads a URL

Use these instead of saying "look at the auth file" — give Claude exactly what it needs.

---

## Concept 2: Context Window Management

The context window is finite. Every file read, every tool output, every back-and-forth exchange consumes tokens. When it fills up, Claude's output quality degrades.

**Run `/context` now.** You'll see a color grid showing how full your context is.

**When to `/compact`:**
- At ~50% context usage (the CLAUDE.md in this repo sets autocompact at 80%)
- Before starting a new, unrelated task
- After a long debugging session where the history isn't relevant anymore

**`/compact [instructions]`** — The optional instructions focus what Claude remembers. Example:
```
/compact Keep the context about the authentication bug, forget the styling conversation
```

**When to start a new session (`/clear`):**
- After you've fully finished a task and the history serves no purpose
- When you're switching to a completely different project area

**Tip from Boris:** The `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` env variable (in this repo's settings.json set to `"80"`) controls when autocompact triggers. You can set it lower (e.g., `50`) to compact earlier and preserve more quality throughout long sessions.

---

## Concept 3: Plan Mode (`/plan`)

Plan mode is **read-only** — Claude explores your codebase and proposes an approach, but cannot write or edit any files until you exit plan mode.

**When to use it:**
- Before implementing any non-trivial feature
- When you're unsure how Claude will approach a problem
- When the task touches multiple files

**How it works:**
1. Run `/plan` (or ask Claude to enter plan mode)
2. Claude explores, researches, and writes a plan file (saved to `./reports/` in this repo)
3. You review and approve (or ask for changes)
4. Claude exits plan mode and implements

**The discipline:** If you skip planning on complex tasks, Claude often goes down the wrong path and you waste turns fixing it. The 5-minute planning step saves 30 minutes of correction.

---

## Concept 4: Built-in Commands by Category

There are 50+ built-in slash commands (growing with each release). Here's the map:

**Session management:**
`/clear`, `/compact`, `/fork`, `/rename`, `/resume`, `/rewind`, `/exit`

**Context & cost:**
`/context` (visual grid), `/cost` (spend), `/usage` (plan limits), `/extra-usage` (overflow billing), `/stats` (history), `/status` (session summary)

**Model:**
`/model` (switch + effort level), `/plan` (read-only mode), `/fast` (fast mode toggle), `/passes` (review pass count)

**Project:**
`/init` (generate CLAUDE.md), `/memory` (view/edit all memory files), `/add-dir` (add working dirs), `/diff` (git diff review), `/review` (code review), `/pr-comments` (PR comments), `/security-review` (security audit)

**Config:**
`/config`, `/keybindings`, `/permissions`, `/sandbox`, `/statusline`, `/terminal-setup`, `/theme`, `/vim`, `/output-style`, `/privacy-settings`

**Extensions:**
`/agents`, `/hooks`, `/mcp`, `/plugin`, `/skills`, `/ide`, `/chrome`, `/reload-plugins`

**Remote & companion apps:**
`/desktop`, `/mobile`, `/remote-control`, `/install-github-app`, `/install-slack-app`

**Debug:**
`/doctor`, `/feedback` (alias: `/bug`), `/help`, `/tasks`, `/release-notes`

**Export:**
`/copy`, `/export`

**Auth:** `/login`, `/logout`

**Scheduled tasks:**
`/loop` — Schedule recurring tasks on a cron interval (e.g., `/loop 1m "check server status"`). Auto-expires after 3 days. Cancel with `cron cancel <job-id>`.

**Side conversations:**
`/btw` — Start a side chain conversation while Claude is actively working. Use it to ask a quick question or give a clarifying note without interrupting the main task flow.

---

## Concept 5: Effort Levels (Boris's Tip #2)

Run `/model` to access effort level controls (Opus 4.6 only):
- **Low** — fast responses, fewer tokens. Use for simple lookups, explanations.
- **Medium** — balanced. Good for everyday coding tasks.
- **High** (Boris's preference) — full reasoning depth. Use for complex bugs, architecture decisions.

The right effort level for the task size saves money and time. Don't use opus + high effort to rename a variable.

---

## Quiz

**Q1:** You ask Claude to "add error handling to the API." Claude asks clarifying questions and the output is mediocre. What's wrong with your prompt and how would you rewrite it?

**A1:** The prompt is too vague. A better version: "Add error handling to the `POST /api/users` endpoint in @src/routes/users.ts. Currently it crashes with 500 if email is missing. It should return 400 with `{ error: 'email required' }`. Don't modify the response format for success cases."

---

**Q2:** Your session is at 75% context. You're mid-way through debugging a tricky auth bug. Should you `/compact`, `/clear`, or keep going?

**A2:** `/compact` — with instructions like "keep the auth bug context, forget the earlier test discussion." This preserves relevance while freeing context. `/clear` would lose your debugging progress.

---

**Q3:** What does `/plan` prevent Claude from doing, and why is that useful?

**A3:** `/plan` prevents Claude from writing, editing, or creating any files (read-only mode). This is useful because you can let Claude explore freely without risk of premature changes, review the proposed approach, and course-correct before any code is written.

---

**Q4:** What's the difference between `@src/auth.ts` and just saying "look at the auth file" in your prompt?

**A4:** `@src/auth.ts` explicitly loads the file content into context at the start of the prompt. "Look at the auth file" relies on Claude to find and read the file mid-task, which costs extra turns and may read the wrong file.

---

## Build Exercise

**Goal:** Develop a personal prompting habit.

1. Find 3 prompts you've actually sent Claude recently that got mediocre results
2. Rewrite each using the `[File/Function] + [What's wrong] + [Expected behavior] + [Constraints]` pattern
3. Test the rewritten versions and note the difference in output quality
4. Practice `/plan` on one non-trivial task in any real project you have
5. Run `/model` and intentionally use haiku for a simple task (note the speed/quality tradeoff)

**Completion check:**
- [ ] Can articulate specifically why a bad prompt fails
- [ ] Used @ references at least once in a real prompt
- [ ] Successfully used `/plan` and approved/modified a plan before implementation
- [ ] Know when to `/compact` vs `/clear` vs keep going

---

## Further Reading

- [best-practice/claude-commands.md](../best-practice/claude-commands.md) — Full commands reference with all 40+ built-in commands
- [tips/claude-boris-tips-feb-26.md](../tips/claude-boris-tips-feb-26.md) — Boris's #1 (terminal setup), #2 (effort), #11 (output styles)

**Previous:** [Module 0 →](module-0-orientation.md) | **Next:** [Module 2 — Project Memory →](module-2-memory.md)
