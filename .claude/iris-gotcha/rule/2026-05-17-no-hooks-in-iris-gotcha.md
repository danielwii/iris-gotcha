---
type: rule
title: "Don't add hooks to iris-gotcha plugin"
keywords: [hooks, settings.json, no-runtime, PreToolUse, PostToolUse, agent-agnostic]
scope: project
severity: medium
created: 2026-05-17
violation_count: 0
disambiguation: "why not architecture: this is an imperative MUST restricting future contributors (don't do X); the descriptive rationale (why markdown-only is the right design) is split into a sibling architecture entry that this one cross-references"
---

## Background

The plugin's value proposition rests on doctrine propagating uniformly across all consumer agents (Claude Code, ein, future Claude-derived agents) via markdown. Hooks tie behavior to harness APIs that vary between agents — adding any hook breaks the cross-agent consistency property.

## Prescription

Do not register Claude Code hooks (`PreToolUse`, `PostToolUse`, `SessionStart`, `SessionEnd`, `PreCompact`, etc.) in this plugin. If a behavior seems to need a hook, the doctrinally-correct alternative is one of:

1. **Document the trigger in SKILL.md's "When to invoke" table** — let the AI invoke the skill autonomously when conditions match. This is how v0.3.0 onward handles "retried 3+ times in this turn → capture as lesson".
2. **Add a new action with explicit invocation** — like `action=audit` or `action=overview`. The user (or AI on the user's behalf) calls it.
3. **Update an existing procedure step** to handle the case inline.

If after considering these a hook still seems necessary, the answer is "drop the feature instead". Convenience hooks are a permanent architectural cost: every consumer agent must be revisited when the harness API changes, and doctrine can drift silently between agents.

## Related

- See `architecture/2026-05-17-markdown-only-doctrine.md` — the design intent this rule enforces: why markdown-only is non-negotiable.
