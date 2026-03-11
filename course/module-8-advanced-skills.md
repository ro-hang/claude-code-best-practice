# Module 8: Advanced Skills

**Journey position:** Beyond 100% — mastery content
**Learning mode:** Build + Quiz
**Time estimate:** 90–120 minutes
**Prerequisite:** Module 4 (Skills fundamentals)

---

## Objectives

By the end of this module you'll be able to:
- Explain the 3-level progressive disclosure system and why it matters for token efficiency
- Structure a skill directory with `scripts/`, `references/`, and `assets/`
- Write descriptions that trigger reliably using the Anthropic formula
- Identify which of the 3 use case categories your skill falls into
- Apply the 5 workflow patterns to real skill designs
- Test skills systematically (triggering, functional, performance)
- Diagnose and fix the 7 most common skill failure modes
- Understand distribution paths (Claude.ai upload, org-wide deployment, Skills API)

---

## Concept 1: Progressive Disclosure — The 3-Level System

This is the foundational architecture of how skills load. Module 4 covered what skills *are*; this explains how Claude *decides* what to load and when.

### The Three Levels

**Level 1 — YAML Frontmatter (always loaded):**
Your frontmatter is injected into Claude's system prompt at all times. This is how Claude decides whether your skill is relevant to the current task. It must be concise and descriptive — it's always consuming tokens.

**Level 2 — SKILL.md Body (loaded on relevance):**
When Claude determines your skill is relevant (based on the frontmatter description matching the user's request), it loads the full SKILL.md body. This is where your actual instructions live.

**Level 3 — Linked Files (loaded on demand):**
Files in `references/`, `scripts/`, and `assets/` are only loaded when Claude actively needs them during execution. Claude discovers and navigates to them as needed.

### Why This Matters

Without progressive disclosure, every skill's full content would be in context at all times. With 20+ skills enabled, that would consume massive token budget before the user even asks a question. The three levels ensure:
- Level 1: Minimal overhead (~50 tokens per skill for discovery)
- Level 2: Medium load (only when relevant)
- Level 3: On-demand (only specific files needed for execution)

### Implication for Skill Design

Your **frontmatter description** is the gatekeeper. If it's vague, Claude either loads your skill when it shouldn't (wasting tokens) or doesn't load it when it should (broken experience). Getting Level 1 right is the single highest-leverage thing you can do.

---

## Concept 2: Skill Directory Structure

Module 4 focused on `SKILL.md`. Production skills use a richer directory structure:

```
your-skill-name/
├── SKILL.md              # Required — instructions with YAML frontmatter
├── scripts/              # Optional — executable code
│   ├── validate.py       # Deterministic validation
│   └── process_data.sh   # Data processing
├── references/           # Optional — documentation loaded on demand
│   ├── api-guide.md      # API patterns and examples
│   └── examples/         # Example inputs/outputs
└── assets/               # Optional — templates, fonts, icons
    └── report-template.md
```

### Why Bundle Scripts?

This is a key insight from Anthropic's guide: **code is deterministic; language interpretation isn't.**

Instead of writing in SKILL.md:
```markdown
Validate that the CSV has all required fields, dates are in YYYY-MM-DD format,
and no required fields are missing.
```

Bundle a script:
```markdown
Run `python scripts/validate.py --input {filename}` to check data format.
If validation fails, common issues include:
- Missing required fields (add them to the CSV)
- Invalid date formats (use YYYY-MM-DD)
```

The script guarantees consistent validation. The SKILL.md instructions tell Claude what to do when validation fails. Each does what it's best at.

### Critical Naming Rules

- **SKILL.md must be exactly `SKILL.md`** — case-sensitive. Not `skill.md`, not `SKILL.MD`, not `Skill.md`.
- **Folder name must be kebab-case** — `notion-project-setup`, not `Notion Project Setup` or `notion_project_setup`.
- **No `README.md` inside the skill folder** — all documentation goes in `SKILL.md` or `references/`. (You can have a repo-level README for humans if distributing via GitHub.)

### References as Level 3 Content

Move detailed documentation to `references/` and link from SKILL.md:

```markdown
## API Integration
Before writing queries, consult `references/api-patterns.md` for:
- Rate limiting guidance
- Pagination patterns
- Error codes and handling
```

This keeps SKILL.md focused on workflow instructions while detailed reference material loads only when Claude actually needs it.

---

## Concept 3: Writing Effective Descriptions

The description field is the most important line in your skill. It controls Level 1 of progressive disclosure — whether Claude loads your skill at all.

### The Formula

```
[What it does] + [When to use it] + [Key capabilities]
```

### Good Descriptions

```yaml
# Good — specific, includes trigger phrases
description: Analyzes Figma design files and generates developer handoff
  documentation. Use when user uploads .fig files, asks for "design specs",
  "component documentation", or "design-to-code handoff".

# Good — includes trigger phrases and scope
description: Manages Linear project workflows including sprint planning,
  task creation, and status tracking. Use when user mentions "sprint",
  "Linear tasks", "project planning", or asks to "create tickets".

# Good — clear value proposition and triggers
description: End-to-end customer onboarding workflow for PayFlow. Handles
  account creation, payment setup, and subscription management. Use when
  user says "onboard new customer", "set up subscription", or
  "create PayFlow account".
```

### Bad Descriptions

```yaml
# Too vague — won't trigger reliably
description: Helps with projects.

# Missing triggers — Claude doesn't know WHEN to load it
description: Creates sophisticated multi-page documentation systems.

# Too technical, no user triggers — users don't talk like this
description: Implements the Project entity model with hierarchical relationships.
```

### Negative Triggers (Preventing Over-Triggering)

When your skill loads for queries it shouldn't, add explicit exclusions:

```yaml
description: Advanced data analysis for CSV files. Use for statistical
  modeling, regression, clustering. Do NOT use for simple data exploration
  (use data-viz skill instead).
```

```yaml
description: PayFlow payment processing for e-commerce. Use specifically
  for online payment workflows, not for general financial queries.
```

### The Debugging Trick

Ask Claude: **"When would you use the [skill-name] skill?"** Claude will quote the description back to you. If the answer doesn't match your intent, revise the description.

### Size Constraint

Descriptions must be under **1,024 characters**. No XML angle brackets (`<` or `>`).

---

## Concept 4: Writing Effective Instructions

The SKILL.md body (Level 2) is where your actual workflow lives. How you write it determines whether Claude follows your process or goes off-script.

### Recommended Structure

```markdown
---
name: your-skill
description: [...]
---

# Your Skill Name

## Instructions

### Step 1: [First Major Step]
Clear explanation of what happens.
Run `python scripts/fetch_data.py --project-id PROJECT_ID`
Expected output: [describe what success looks like]

### Step 2: [Next Step]
...

## Examples
Example 1: [common scenario]
User says: "Set up a new marketing campaign"
Actions:
1. Fetch existing campaigns via MCP
2. Create new campaign with provided parameters
Result: Campaign created with confirmation link

## Troubleshooting
Error: [Common error message]
Cause: [Why it happens]
Solution: [How to fix]
```

### Best Practices

**Be specific and actionable:**
```markdown
# Bad
Validate the data before proceeding.

# Good
Run `python scripts/validate.py --input {filename}` to check data format.
If validation fails, common issues include:
- Missing required fields (add them to the CSV)
- Invalid date formats (use YYYY-MM-DD)
```

**Put critical instructions at the top.** Claude pays most attention to early content. If something must not be skipped, use `## Critical` or `## Important` headers and place them before detailed steps.

**Repeat key constraints if needed.** For long skills, restate critical rules at decision points — not just at the top.

**Include error handling for every MCP call:**
```markdown
## Common Issues
### MCP Connection Failed
If you see "Connection refused":
1. Verify MCP server is running: Check Settings > Extensions
2. Confirm API key is valid
3. Try reconnecting: Settings > Extensions > [Your Service] > Reconnect
```

**Use progressive disclosure in instructions too.** Keep SKILL.md focused on the core workflow. Move detailed API docs, style guides, and reference tables to `references/` and link to them.

### Combating Model Laziness

For steps Claude tends to skip or shortcut, add explicit encouragement:
```markdown
## Performance Notes
- Take your time to do this thoroughly
- Quality is more important than speed
- Do not skip validation steps
```

Note from Anthropic: adding this to **user prompts** is more effective than embedding it in SKILL.md. But it helps in both places.

---

## Concept 5: The Three Use Case Categories

Anthropic has observed three common patterns in how skills get used. Knowing which category yours falls into helps you choose the right structure.

### Category 1: Document & Asset Creation

**Used for:** Creating consistent, high-quality output — documents, presentations, apps, designs, code.

**Key techniques:**
- Embedded style guides and brand standards
- Template structures for consistent output
- Quality checklists before finalizing
- No external tools required — uses Claude's built-in capabilities

**Example:** A `frontend-design` skill that creates production-grade UI components following your design system. Or the `weather-svg-creator` in this repo — it creates a consistent SVG card from temperature data.

### Category 2: Workflow Automation

**Used for:** Multi-step processes that benefit from consistent methodology, including coordination across multiple tools or MCP servers.

**Key techniques:**
- Step-by-step workflow with validation gates
- Templates for common structures
- Built-in review and improvement suggestions
- Iterative refinement loops

**Example:** A `skill-creator` that walks users through use case definition, frontmatter generation, instruction writing, and validation. Or the `course-sync` skill in this repo — it maps upstream changes to modules.

### Category 3: MCP Enhancement

**Used for:** Adding workflow guidance on top of MCP tool access. This is the "recipes for the kitchen" pattern.

**The kitchen analogy:** MCP provides the professional kitchen (tools, ingredients, equipment). Skills provide the recipes (step-by-step instructions for creating something valuable). Without skills, users connect the MCP but don't know what workflows are possible.

**Key techniques:**
- Coordinates multiple MCP calls in sequence
- Embeds domain expertise about the service
- Provides context users would otherwise need to specify
- Error handling for common MCP issues

**Without skills:** Users connect your MCP → don't know what to do next → inconsistent results → blame the connector.
**With skills:** Pre-built workflows activate automatically → consistent tool usage → best practices in every interaction → lower learning curve.

---

## Concept 6: The Five Workflow Patterns

These patterns emerged from skills created by early adopters and Anthropic's internal teams. Most production skills use one or more of these.

### Pattern 1: Sequential Workflow Orchestration

**Use when:** Users need multi-step processes in a specific order.

```markdown
## Workflow: Onboard New Customer
### Step 1: Create Account
Call MCP tool: `create_customer`
Parameters: name, email, company
### Step 2: Setup Payment
Call MCP tool: `setup_payment_method`
Wait for: payment method verification
### Step 3: Create Subscription
Call MCP tool: `create_subscription`
Parameters: plan_id, customer_id (from Step 1)
### Step 4: Send Welcome Email
Call MCP tool: `send_email`
Template: welcome_email_template
```

**Key techniques:** Explicit step ordering, dependencies between steps, validation at each stage, rollback instructions for failures.

### Pattern 2: Multi-MCP Coordination

**Use when:** Workflows span multiple services.

```markdown
### Phase 1: Design Export (Figma MCP)
1. Export design assets from Figma
2. Generate design specifications
3. Create asset manifest

### Phase 2: Asset Storage (Drive MCP)
1. Create project folder in Drive
2. Upload all assets
3. Generate shareable links

### Phase 3: Task Creation (Linear MCP)
1. Create development tasks
2. Attach asset links to tasks
3. Assign to engineering team

### Phase 4: Notification (Slack MCP)
1. Post handoff summary to #engineering
2. Include asset links and task references
```

**Key techniques:** Clear phase separation, data passing between MCPs, validation before moving to next phase, centralized error handling.

### Pattern 3: Iterative Refinement

**Use when:** Output quality improves with iteration.

```markdown
### Initial Draft
1. Fetch data via MCP
2. Generate first draft report
3. Save to temporary file

### Quality Check
1. Run validation script: `scripts/check_report.py`
2. Identify issues (missing sections, inconsistent formatting, data errors)

### Refinement Loop
1. Address each identified issue
2. Regenerate affected sections
3. Re-validate
4. Repeat until quality threshold met

### Finalization
1. Apply final formatting
2. Generate summary
3. Save final version
```

**Key techniques:** Explicit quality criteria, validation scripts, knowing when to stop iterating (set a max iteration count or quality threshold).

### Pattern 4: Context-Aware Tool Selection

**Use when:** Same outcome, different tools depending on context.

```markdown
### Decision Tree
1. Check file type and size
2. Determine best storage location:
   - Large files (>10MB): Use cloud storage MCP
   - Collaborative docs: Use Notion/Docs MCP
   - Code files: Use GitHub MCP
   - Temporary files: Use local storage

### Execute Storage
Based on decision:
- Call appropriate MCP tool
- Apply service-specific metadata
- Generate access link

### Provide Context to User
Explain why that storage was chosen
```

**Key techniques:** Clear decision criteria, fallback options, transparency about choices.

### Pattern 5: Domain-Specific Intelligence

**Use when:** Your skill adds specialized knowledge beyond tool access — compliance rules, industry regulations, best practices that aren't in any API.

```markdown
### Before Processing (Compliance Check)
1. Fetch transaction details via MCP
2. Apply compliance rules:
   - Check sanctions lists
   - Verify jurisdiction allowances
   - Assess risk level
3. Document compliance decision

### Processing
IF compliance passed:
- Call payment processing MCP tool
- Apply appropriate fraud checks
- Process transaction
ELSE:
- Flag for review
- Create compliance case

### Audit Trail
- Log all compliance checks
- Record processing decisions
- Generate audit report
```

**Key techniques:** Domain expertise embedded in logic, compliance/validation before action, comprehensive documentation, clear governance.

---

## Concept 7: Testing Skills Systematically

Most people write a skill, try it once, and call it done. Anthropic recommends three levels of testing.

### Level 1: Triggering Tests

**Goal:** Does your skill load at the right times?

Create a test suite of queries:

**Should trigger:**
- "Help me set up a new ProjectHub workspace"
- "I need to create a project in ProjectHub"
- "Initialize a ProjectHub project for Q4 planning"

**Should NOT trigger:**
- "What's the weather in San Francisco?"
- "Help me write Python code"
- "Create a spreadsheet" (unless your skill handles sheets)

**Target:** Skill triggers on 90%+ of relevant queries. Run 10-20 test queries and track how many times it loads automatically vs. requires explicit invocation.

### Level 2: Functional Tests

**Goal:** Does the skill produce correct outputs?

```
Test: Create project with 5 tasks
Given: Project name "Q4 Planning", 5 task descriptions
When: Skill executes workflow
Then:
- Project created in ProjectHub
- 5 tasks created with correct properties
- All tasks linked to project
- No API errors
```

### Level 3: Performance Comparison

**Goal:** Prove the skill improves results vs. baseline.

| Metric | Without Skill | With Skill |
|--------|--------------|------------|
| Back-and-forth messages | 15 | 2 clarifying questions |
| Failed API calls | 3 requiring retry | 0 |
| Tokens consumed | 12,000 | 6,000 |
| Workflow automation | Manual each time | Automatic |

### Iteration Signals

**Under-triggering (skill doesn't load when it should):**
- Users manually enabling the skill
- Support questions about when to use it
- **Fix:** Add more trigger phrases and keywords to the description

**Over-triggering (skill loads for unrelated queries):**
- Users disabling the skill
- Confusion about the skill's purpose
- **Fix:** Add negative triggers, be more specific about scope

**Instructions not followed:**
- Claude loads the skill but goes off-script
- **Fix:** Put critical instructions at the top, use explicit headers, repeat key constraints, consider bundling validation as scripts instead of language

---

## Concept 8: Troubleshooting — The 7 Common Failure Modes

### 1. Skill Won't Upload

**Error:** "Could not find SKILL.md in uploaded folder"
**Cause:** File not named exactly `SKILL.md` (case-sensitive)
**Fix:** Rename to `SKILL.md`. Verify with `ls -la`.

### 2. Invalid Frontmatter

**Error:** "Invalid frontmatter"
**Cause:** YAML formatting issue

```yaml
# Wrong — missing delimiters
name: my-skill
description: Does things

# Wrong — unclosed quotes
---
name: my-skill
description: "Does things
---

# Correct
---
name: my-skill
description: Does things
---
```

### 3. Invalid Skill Name

**Error:** "Invalid skill name"
**Cause:** Name has spaces or capitals

```yaml
# Wrong
name: My Cool Skill

# Correct
name: my-cool-skill
```

### 4. Skill Doesn't Trigger

**Symptom:** Skill never loads automatically
**Checklist:**
- Is the description too generic? ("Helps with projects" won't trigger reliably)
- Does it include trigger phrases users would actually say?
- Does it mention relevant file types if applicable?
**Debug:** Ask Claude "When would you use the [skill name] skill?" and compare the answer to your intent.

### 5. Skill Triggers Too Often

**Symptom:** Loads for unrelated queries
**Fixes:**
1. Add negative triggers ("Do NOT use for simple data exploration")
2. Be more specific ("Processes PDF legal documents for contract review" instead of "Processes documents")
3. Clarify scope ("PayFlow payment processing for e-commerce. Use specifically for online payment workflows, not for general financial queries.")

### 6. MCP Connection Issues

**Symptom:** Skill loads but MCP calls fail
**Checklist:**
1. Verify MCP server is connected (Settings > Extensions)
2. Check authentication (API keys valid, OAuth tokens refreshed, proper scopes)
3. Test MCP independently — ask Claude to call the MCP directly without the skill. If this fails, the issue is MCP, not the skill.
4. Verify tool names are correct — MCP tool names are case-sensitive

### 7. Large Context Degradation

**Symptom:** Skill seems slow or responses degraded
**Causes:** Skill content too large, too many skills enabled, all content loaded instead of progressive disclosure
**Fixes:**
1. **Keep SKILL.md under 5,000 words** — move detailed docs to `references/`
2. **Limit enabled skills** — evaluate if you have more than 20-50 skills enabled simultaneously
3. **Use progressive disclosure** — don't inline everything in SKILL.md, link to `references/` files

---

## Concept 9: Expanded Frontmatter — Fields Beyond Module 4

Module 4 covered `name`, `description`, `user-invocable`, `disable-model-invocation`, `allowed-tools`, `model`, `context`, `agent`, `argument-hint`, and `hooks`. Here are the additional fields from Anthropic's official guide:

### `license`
```yaml
license: MIT  # or Apache-2.0
```
Use when making skills open source. Common for community-distributed skills.

### `compatibility`
```yaml
compatibility: Requires Claude.ai with code execution enabled.
  Network access needed for API calls.
```
1-500 characters. Indicates environment requirements: intended product surface, required system packages, network access needs, etc.

### `metadata`
```yaml
metadata:
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [project-management, automation]
  documentation: https://example.com/docs
  support: support@example.com
```
Custom key-value pairs. Not used by Claude for discovery — this is for humans and tooling. Useful for version tracking and distribution.

### Security Restrictions

Things that are **forbidden** in frontmatter:
- XML angle brackets (`<` or `>`) — frontmatter appears in Claude's system prompt; XML could inject instructions
- Skill names containing "claude" or "anthropic" — reserved by Anthropic
- Code execution in YAML — uses safe YAML parsing

---

## Concept 10: Distribution and the Skills Ecosystem

### Distribution Paths

**Individual users (Claude.ai):**
1. Download the skill folder
2. Zip it
3. Upload via Settings > Capabilities > Skills

**Individual users (Claude Code):**
Place the skill folder in `.claude/skills/` in the project, or `~/.claude/skills/` for global access.

**Organization-wide deployment:**
Admins can deploy skills workspace-wide with automatic updates and centralized management (shipped December 2025).

**Skills API (programmatic use):**
- `/v1/skills` endpoint for listing and managing skills
- `container.skills` parameter in Messages API requests
- Version control via Claude Console
- Works with the Claude Agent SDK for custom agents
- Requires Code Execution Tool beta

### When to Use API vs Claude.ai

| Use Case | Best Surface |
|----------|-------------|
| End users interacting with skills | Claude.ai / Claude Code |
| Manual testing during development | Claude.ai / Claude Code |
| Individual, ad-hoc workflows | Claude.ai / Claude Code |
| Applications using skills programmatically | API |
| Production deployments at scale | API |
| Automated pipelines and agent systems | API |

### The Open Standard

Anthropic published Agent Skills as an open standard (like MCP). Skills are designed to be portable across tools and platforms. The `compatibility` frontmatter field lets authors note platform-specific requirements.

### The `skill-creator` Skill

Available in Claude.ai and for Claude Code. It can:
- Generate skills from natural language descriptions
- Produce properly formatted SKILL.md with frontmatter
- Suggest trigger phrases and structure
- Review existing skills and flag issues (vague descriptions, missing triggers, structural problems)

Invoke it: "Help me build a skill using skill-creator"

---

## Quiz

**Q1:** What are the three levels of progressive disclosure in skills, and what loads at each level?

**A1:**
- Level 1: YAML frontmatter — always loaded in Claude's system prompt (for discovery)
- Level 2: SKILL.md body — loaded when Claude determines the skill is relevant to the current task
- Level 3: Linked files (`references/`, `scripts/`, `assets/`) — loaded on demand during execution

---

**Q2:** Your skill triggers for the query "help me write Python code" but it's a ProjectHub project setup skill. What do you add to the description?

**A2:** Add negative triggers: "Do NOT use for general coding tasks, Python scripts, or non-ProjectHub work." Be more specific about scope: "Use specifically for ProjectHub workspace setup, not for general project management or coding."

---

**Q3:** You have a data validation step in your skill. Claude sometimes skips it or validates incorrectly. What's the best fix?

**A3:** Bundle a validation script (`scripts/validate.py`) and have the SKILL.md call it: `Run python scripts/validate.py --input {filename}`. Code is deterministic; language interpretation isn't. The script guarantees consistent validation.

---

**Q4:** Your skill's SKILL.md is 8,000 words and Claude seems slow when it loads. What do you do?

**A4:** Move detailed documentation to `references/` files and link to them from SKILL.md. Keep SKILL.md under 5,000 words — it should contain core workflow instructions, not exhaustive reference material. This leverages Level 3 progressive disclosure.

---

**Q5:** You want a skill that coordinates Figma export → Google Drive upload → Linear task creation → Slack notification. Which workflow pattern is this?

**A5:** Pattern 2: Multi-MCP Coordination. It spans multiple services with clear phase separation, data passing between MCPs, and validation before moving to each next phase.

---

**Q6:** What's the difference between Category 2 (Workflow Automation) and Category 3 (MCP Enhancement)?

**A6:** Category 2 focuses on multi-step processes that benefit from consistent methodology — the emphasis is on the *workflow*. Category 3 focuses on adding guidance *on top of* existing MCP tool access — the emphasis is on teaching Claude the best way to use a specific service. In Anthropic's analogy: MCP is the kitchen (tools), skills are the recipes (how to use them).

---

## Build Exercise

**Goal:** Take an existing skill from this repo (or Module 4's build exercise) and level it up using the patterns from this module.

**Part 1 — Restructure with Progressive Disclosure:**
Take the `commit-msg` skill from Module 4 (or any skill you've built) and restructure it:
- Move any reference material to a `references/` subdirectory
- Add a `scripts/` directory with at least one validation script
- Ensure SKILL.md stays focused on core workflow instructions
- Review and improve the description using the formula: [What it does] + [When to use it] + [Key capabilities]

**Part 2 — Add Testing:**
Write a test plan for your skill:
- List 5 queries that SHOULD trigger it
- List 5 queries that should NOT trigger it
- Define 2 functional test cases (Given/When/Then)
- Run the triggering tests and document the results

**Part 3 — Apply a Workflow Pattern:**
Build a new skill (or enhance an existing one) using one of the 5 workflow patterns:
- If you have MCP servers configured, try Pattern 2 (Multi-MCP Coordination) or Pattern 3 (Iterative Refinement)
- If not, try Pattern 5 (Domain-Specific Intelligence) — embed specialized knowledge for your domain into a skill with decision logic and validation

**Completion check:**
- [ ] Skill has `scripts/` and/or `references/` subdirectories (not everything in SKILL.md)
- [ ] Description follows the [What] + [When] + [Capabilities] formula
- [ ] At least 5 triggering tests documented with results
- [ ] At least 1 functional test case defined and passing
- [ ] Can explain which workflow pattern your skill uses and why

---

## Further Reading

- [Anthropic: The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) — The source PDF for this module
- [GitHub: anthropics/skills](https://github.com/anthropics/skills) — Anthropic's public skills repository with production examples
- [best-practice/claude-skills.md](../best-practice/claude-skills.md) — Project reference for skill fundamentals
- [reports/claude-skills-for-larger-mono-repos.md](../reports/claude-skills-for-larger-mono-repos.md) — Monorepo discovery deep dive
- [Module 4: Skills](module-4-skills.md) — Prerequisites: the two skill patterns, frontmatter basics, `context: fork`, string substitutions, monorepo discovery

**Previous:** [Module 7 — Advanced Orchestration](module-7-orchestration.md)
