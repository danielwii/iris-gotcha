---
type: architecture
title: "iris-gotcha is markdown-only — no hooks, no runtime, no per-agent customization"
keywords: [markdown, hooks, runtime, agent-agnostic, doctrine, single-source-of-truth]
scope: project
created: 2026-05-17
disambiguation: "why not rule: describes the design property and its rationale (descriptive intent); the imperative consequence ('don't add hooks') is split into a sibling rule entry that this one cross-references"
---

iris-gotcha encodes all its doctrine — capture procedure, category definitions, severity ladder, split test, wiring rules — exclusively in two markdown files: `skills/iris-gotcha/SKILL.md` and `skills/iris-gotcha/definitions.md`. There is no executable code, no JavaScript/TypeScript runtime, no hooks registered in `settings.json`, no `PreToolUse` / `PostToolUse` / `SessionStart` handlers, no per-agent customization layer.

This is deliberate. The single source of truth means:

1. **Doctrine propagates uniformly across all agents.** Daniel uses Claude Code; ein runs a separate Claude-derived agent; future contributors may use other agents. All of them install the same plugin from `danielwii/iris-gotcha` and read the same markdown. When the doctrine evolves (v0.6.0 → v0.7.0 etc.), a single `git tag` + `/plugin install` updates every consumer atomically.

2. **No per-agent integration work.** When ein discovered a v0.7.0 doctrine gap in its first capture run, the fix was entirely on the SKILL side — write better markdown, push v0.8.0, ein installs. No code change in ein, no special API, no hook contract.

3. **Behavior is the markdown.** The skill never relies on harness behavior beyond `Read` / `Write` / `Bash` — tools every Claude-derived agent has. There is no "iris-gotcha runtime" that could drift between agents or fail to install in one but not another.

4. **Reviewability and forkability stay high.** A new contributor inspecting the plugin can read SKILL.md top to bottom in 5 minutes and have the full mental model. A JavaScript hook would introduce a parallel logic path that diverges from the markdown — a future Claude would have to read both to know what the plugin actually does.

The alternative — adding hooks for "convenience" (auto-capture on every retry, auto-audit at session end, auto-strengthen on certain patterns) — seems appealing locally, but each hook adds per-agent runtime that must be ported to every consumer agent and re-tested when any harness API changes. The convenience benefit is small; the architecture cost is permanent.

## Related

- See `rule/2026-05-17-no-hooks-in-iris-gotcha.md` — the imperative consequence: don't add hooks even if they seem locally convenient.
