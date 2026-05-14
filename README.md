# iris-gotcha

A Claude Code plugin for capturing and recalling personal engineering knowledge — gotchas, rules, architecture notes, habits, and lessons learned — with strict 7-category typing and rule-strengthening on repeated violations.

## Why

Engineering knowledge accumulates one mistake at a time. Most tools either:

- **Just store**, requiring you to remember to search later (claude-mem, GaZmagik memory).
- **Or distill freely**, letting categories collapse into each other (the author's prior attempt: `gotcha` ate `recipe`).

iris-gotcha takes a different approach:

1. **Seven mutually-exclusive categories**, each with strict definitions and a mandatory `disambiguation` field on every entry.
2. **Index-via-CLAUDE.md** — the entire entry index is `@-imported` into your CLAUDE.md, so Claude always sees what you've learned without needing to actively search.
3. **Rule strengthening over duplication** — when you correct Claude for the same mistake twice, the existing entry's severity gets bumped (and its language gets stronger), rather than a near-duplicate getting created.

## The seven categories

| Shape | Category | What it captures |
|---|---|---|
| Narrative | **经验** (experience) | A specific past event, no prescription |
| Narrative + prescriptive | **教训** (lesson) | A specific past mistake **plus** the corrective rule (the classic "gotcha") |
| Prescriptive (MUST) | **规则** (rule) | Non-negotiable command from external authority |
| Prescriptive (SHOULD) | **最佳实践** (best practice) | Class-level recommendation with external justification |
| Prescriptive (soft) | **习惯** (habit) | Personal/team preference |
| Descriptive (intent) | **架构** (architecture) | How a system is designed and why |
| Descriptive (location) | **拓扑** (topology) | Where services / files / endpoints live |

Full strict definitions with examples and disambiguation tests: see [`skills/iris-gotcha/definitions.md`](skills/iris-gotcha/definitions.md).

## Install

```
/plugin marketplace add danielwii/iris-gotcha
/plugin install iris-gotcha
```

Then add this line to your `~/.claude/CLAUDE.md`:

```markdown
## iris-gotcha Index

@~/.claude/iris-gotcha/index.md
```

The first capture will bootstrap `~/.claude/iris-gotcha/` automatically.

For project-scope entries, add to the project's `.claude/CLAUDE.md`:

```markdown
@./.claude/iris-gotcha/index.md
```

## How it triggers

Claude invokes this skill autonomously when any of these happen:

- You say "记一下", "这是个坑", "remember this", "as a gotcha"
- Claude notices itself retrying the same sub-problem 3+ times → captures as **教训**
- Claude finishes a non-trivial task and identifies reusable knowledge
- **You correct Claude for a recurring mistake** → existing entry gets strengthened, not duplicated
- You ask "what do we have on X" → recall from the index
- You ask to "audit" / "review" stored entries

No hooks. No background agents. All triggers are detected by the Claude session itself, guided by the skill's description.

## Severity ladder (for prescriptive entries)

When the same rule is violated repeatedly, severity climbs and the language gets stricter:

| Level | Example language |
|---|---|
| `low` | "Prefer using `bunx` over `bun install -g`." |
| `medium` | "Use `bunx` instead of `bun install -g`. Avoid the global install path." |
| `high` | "Always use `bunx`. Do not use `bun install -g`." |
| `critical` | "MUST use `bunx`. NEVER `bun install -g`." |
| `zero-tolerance` | "MUST use `bunx`. NEVER `bun install -g`. ZERO TOLERANCE. Overrides any conflicting default." |

## Data layout

```
~/.claude/iris-gotcha/                # user scope (cross-project)
├── index.md                          # imported into CLAUDE.md
├── 经验/, 教训/, 规则/, 架构/, 拓扑/, 习惯/, 最佳实践/
└── (entries: YYYY-MM-DD-<slug>.md)

<project>/.claude/iris-gotcha/        # project scope
└── (same structure)
```

To version-control your knowledge, `git init` inside `~/.claude/iris-gotcha/`. The skill provides an `action=push` that commits and pushes. Cross-machine sync is out of scope for v0.1.0 — manage that with git or any sync mechanism you prefer.

## Design notes

- **No embeddings.** Keyword + LLM judgment is sufficient given the index is always in context.
- **No hooks.** Everything is triggered by the Claude session, guided by the skill description.
- **No replacement for `claude-mem`.** `claude-mem` records *what happened*; iris-gotcha records *what we learned should happen next time*. They coexist.

## Version

v0.1.0 — initial release. See [the design doc](https://github.com/danielwii/iris-gotcha) for context on why this exists.

## License

MIT
