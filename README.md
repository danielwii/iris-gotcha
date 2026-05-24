# iris-gotcha

A Claude Code plugin that acts as a **signal-to-noise optimizer for AI context** — captures only the engineering knowledge that's worth a permanent slot in every future session's context. Tool gotchas, project structure, personal preferences, recurring mistakes, workflow recipes — encoded compactly (one-line index, lazy bodies), gated against training-redundancy and against low-signal noise.

**Core purpose (also the evaluation criterion for future changes):**

> Does this raise the ratio of "behavior-changing knowledge" to "context tokens consumed"?

Every mechanism in this plugin — strict 6-category typing, disambiguation gate, severity ladder, strengthening protocol, lazy body loading, `## Related` cross-references, `action=overview`, `action=audit` — exists to push signal density up or to keep noise from accumulating. New features should be evaluated against the same test. If a change can't articulate a signal-density answer, it probably shouldn't ship.

## Why

Engineering knowledge accumulates one mistake at a time. Most tools either:

- **Just store**, requiring you to remember to search later (claude-mem, GaZmagik memory).
- **Or distill freely**, letting categories collapse into each other (the author's prior attempt: `gotcha` ate `recipe`).

iris-gotcha takes a different approach:

1. **Six mutually-exclusive categories**, each with strict definitions and a mandatory `disambiguation` field on every entry.
2. **Index-via-CLAUDE.md** — the entire entry index is `@-imported` into your CLAUDE.md, so Claude always sees what you've learned without needing to actively search.
3. **Rule strengthening over duplication** — when you correct Claude for the same mistake twice, the existing entry's severity gets bumped (and its language gets stronger), rather than a near-duplicate getting created.

## What goes in (and what doesn't)

The AI already handles plenty without help: standard security ("don't log secrets"), generic best practices (input validation, error handling, REST conventions), well-known tool usage. **iris-gotcha doesn't capture those** — they pollute the index without changing AI behavior.

What goes in: tool gotchas the AI hasn't seen, your project structure, personal preferences that diverge from defaults, workflow recipes specific to your environment, quality lessons from your specific bugs, recurring mistakes the AI keeps making.

Single test before capturing: **would the AI, in a fresh session with no notebook, refuse / handle this correctly anyway?** If yes, don't capture.

## The six categories

All entries use **English category identifiers** (`type:` field). Chinese names are kept as cultural glosses.

| `type` | Chinese | Shape | What it captures |
|---|---|---|---|
| `lesson` | 教训 | Behavioral, incident-derived | A specific past mistake **plus** the corrective rule (the classic "gotcha") |
| `rule` | 规则 | Behavioral, MUST | Project-specific MUST, or user override of AI default behavior |
| `best-practice` | 最佳实践 | Behavioral, SHOULD | Class-level recommendation the AI wouldn't follow by default |
| `habit` | 习惯 | Behavioral, soft preference | Personal/team preference |
| `architecture` | 架构 | Reference, design intent | How a system is designed and why |
| `topology` | 拓扑 | Reference, location | Where services / files / endpoints live |

Full strict definitions with examples and disambiguation tests: see [`skills/iris-gotcha/definitions.md`](skills/iris-gotcha/definitions.md).

## Install

```
/plugin marketplace add danielwii/iris-gotcha
/plugin install iris-gotcha
```

That's it. The first capture in any directory automatically:
- Bootstraps `~/.claude/iris-gotcha/` or `<pwd>/.claude/iris-gotcha/` (depending on scope) with the 6 category subdirectories and an `index.md`.
- Wires the `@-import` line into the relevant `CLAUDE.md` (user-scope → `~/.claude/CLAUDE.md`; project-scope → `<pwd>/CLAUDE.md` or, if absent, creates `<pwd>/.claude/CLAUDE.md`).

No manual configuration steps required.

## How it triggers

Claude invokes this skill autonomously when any of these happen:

- You say "记一下", "这是个坑", "remember this", "as a gotcha"
- Claude notices itself retrying the same sub-problem 3+ times → captures as `lesson`
- Claude finishes a non-trivial task and identifies reusable knowledge
- A multi-turn debugging or planning arc concludes → Claude reflects on what was learned and proposes capture (debug: v0.8.0; planning **T1**: v0.11.0)
- **You correct Claude** → if the reason matches an existing entry, the entry gets strengthened; if the reason is new to the index, captured as a new entry (**T2**: v0.11.0)
- **Before writing non-trivial code** → Claude scans the index for applicable rules and announces them (**T3**: v0.11.0)
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
├── lesson/, rule/, architecture/, topology/, habit/, best-practice/
└── (entries: YYYY-MM-DD-<slug>.md)

<project>/.claude/iris-gotcha/        # project scope
└── (same structure)
```

To version-control your knowledge, `git init` inside `~/.claude/iris-gotcha/`. The skill provides an `action=push` that commits and pushes. Cross-machine sync is out of scope — manage that with git or any sync mechanism you prefer.

## Design notes

- **No embeddings.** Keyword + LLM judgment is sufficient given the index is always in context.
- **No hooks.** Everything is triggered by the Claude session, guided by the skill description.
- **No replacement for `claude-mem`.** `claude-mem` records *what happened*; iris-gotcha records *what we learned should happen next time*. They coexist.

## Versioning

Latest: **v0.11.0** — three new triggers (T1 end-of-planning retrospective, T2 novel-reason correction, T3 pre-implementation rule check).

Full release notes and design rationale: [CHANGELOG.md](CHANGELOG.md).

## License

MIT
