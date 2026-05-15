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

All entries use **English category identifiers** (`type:` field). Chinese names are kept as cultural glosses.

| `type` | Chinese | Shape | What it captures |
|---|---|---|---|
| `experience` | 经验 | Narrative | A specific past event, no prescription |
| `lesson` | 教训 | Narrative + prescriptive | A specific past mistake **plus** the corrective rule (the classic "gotcha") |
| `rule` | 规则 | Prescriptive (MUST) | Non-negotiable command from external authority |
| `best-practice` | 最佳实践 | Prescriptive (SHOULD) | Class-level recommendation with external justification |
| `habit` | 习惯 | Prescriptive (soft) | Personal/team preference |
| `architecture` | 架构 | Descriptive (intent) | How a system is designed and why |
| `topology` | 拓扑 | Descriptive (location) | Where services / files / endpoints live |

Full strict definitions with examples and disambiguation tests: see [`skills/iris-gotcha/definitions.md`](skills/iris-gotcha/definitions.md).

## Install

```
/plugin marketplace add danielwii/iris-gotcha
/plugin install iris-gotcha
```

That's it. The first capture in any directory automatically:
- Bootstraps `~/.claude/iris-gotcha/` or `<pwd>/.claude/iris-gotcha/` (depending on scope) with the 7 category subdirectories and an `index.md`.
- Wires the `@-import` line into the relevant `CLAUDE.md` (user-scope → `~/.claude/CLAUDE.md`; project-scope → `<pwd>/CLAUDE.md` or, if absent, creates `<pwd>/.claude/CLAUDE.md`).

No manual configuration steps required.

## How it triggers

Claude invokes this skill autonomously when any of these happen:

- You say "记一下", "这是个坑", "remember this", "as a gotcha"
- Claude notices itself retrying the same sub-problem 3+ times → captures as `lesson`
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
├── experience/, lesson/, rule/, architecture/, topology/, habit/, best-practice/
└── (entries: YYYY-MM-DD-<slug>.md)

<project>/.claude/iris-gotcha/        # project scope
└── (same structure)
```

To version-control your knowledge, `git init` inside `~/.claude/iris-gotcha/`. The skill provides an `action=push` that commits and pushes. Cross-machine sync is out of scope for v0.2.x — manage that with git or any sync mechanism you prefer.

## Design notes

- **No embeddings.** Keyword + LLM judgment is sufficient given the index is always in context.
- **No hooks.** Everything is triggered by the Claude session, guided by the skill description.
- **No replacement for `claude-mem`.** `claude-mem` records *what happened*; iris-gotcha records *what we learned should happen next time*. They coexist.

## Versioning

- `0.1.0` — initial release, Chinese category names in directories and frontmatter.
- `0.2.0` — switched to English category identifiers (`type:` field), with Chinese kept as glosses. **Breaking change** for anyone who installed `0.1.0`: re-classify entries by moving from `教训/` etc. to `lesson/` etc. and update each frontmatter `type:` field to the English identifier.
- `0.3.0` — automatic, idempotent CLAUDE.md `@-import` wiring on every capture. The skill now ensures the relevant CLAUDE.md (user-scope: `~/.claude/CLAUDE.md`; project-scope: `<pwd>/CLAUDE.md` or `<pwd>/.claude/CLAUDE.md`) imports the right index file, creating the project-level CLAUDE.md if absent. No manual setup needed after install.
- `0.4.0` — adds `action=move` for reclassifying entries between categories or scopes; tightens SKILL.md (removed redundant Bootstrapping / Project-scope sections; merged the inline severity-list duplicate; compressed the Recall section to one paragraph); softens tone (explains *why* instead of leaning on `MUST` / `NEVER` where reasoning is more reliable than imperatives).
- `0.4.1` — rewrites the SKILL.md frontmatter `description` per the writing-skills CSO doctrine: pure "Use when..." trigger conditions with no workflow summary. Prior versions began the description with "Capture, classify, recall, audit, move, and push..." which (per writing-skills testing) can cause Claude to act on the description's process summary rather than read the full SKILL body.

## License

MIT
