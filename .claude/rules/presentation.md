# Glob: presentation/**

## Delegation Rule

Any request to update, modify, or fix the presentation (`presentation/index.html`) MUST be handled by the `presentation-curator` agent. Always delegate presentation work to this agent via the Agent tool — never edit the presentation directly.

```
Agent(subagent_type="presentation-curator", description="...", prompt="...")
```

## Why

The presentation-curator agent has three preloaded skills that keep it in sync with the presentation's structure, styling, and conceptual framework. It also self-evolves after every execution, updating its own skills to prevent knowledge drift. Bypassing the agent risks breaking slide numbering, level transitions, or style consistency.
