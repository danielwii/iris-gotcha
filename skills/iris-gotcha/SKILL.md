---
name: iris-gotcha
description: "Capture, classify, recall, audit, and push entries to Daniel's personal knowledge notebook (gotchas, rules, architecture, etc.) with strict 7-category typing. Use when (1) the user says '记一下' / '这是个坑' / 'remember this' / similar; (2) you notice yourself retrying the same sub-problem 3+ times in the current turn (capture as lesson); (3) you finish a non-trivial task and the solution is reusable; (4) you suspect a stored entry is relevant to the current task and want its full content; (5) you are correcting yourself after a user pointed out a recurring mistake — STRENGTHEN the existing entry rather than create a new one; (6) the user asks to audit existing entries for miscategorization. Read ~/.claude/iris-gotcha/index.md for recall and write new entries with strict typing and disambiguation."
---

# iris-gotcha — Personal Knowledge Notebook

A Claude Code skill that lets the assistant build up Daniel's personal knowledge over time using **seven strict categories**, **a disambiguation gate**, and **rule strengthening on repeated violations**.

This skill is designed around three observations from a previous attempt:
1. **Categories collapse without strict definitions.** Last time `recipe` got eaten by `gotcha`. This skill defends against that with a mandatory disambiguation field on every entry.
2. **Index in system context is what enables recall.** All entry titles + keywords live in a single index file that gets `@-imported` into the user's CLAUDE.md, so the index appears in every session's context without any active retrieval mechanism.
3. **Repeated violations should strengthen rules, not add new entries.** When the assistant catches itself about to break an existing rule, the existing entry's severity gets bumped instead of a duplicate being created.

## Category identifiers

All entries use English category identifiers in the `type:` frontmatter field. Chinese names are kept as glosses for readability.

| `type` value | Chinese gloss | Shape |
|---|---|---|
| `experience` | 经验 | Narrative |
| `lesson` | 教训 | Narrative + prescription (the "gotcha") |
| `rule` | 规则 | Prescription (MUST) |
| `habit` | 习惯 | Prescription (soft preference) |
| `best-practice` | 最佳实践 | Prescription (SHOULD with external justification) |
| `architecture` | 架构 | Descriptive (design intent) |
| `topology` | 拓扑 | Descriptive (location facts) |

Full strict definitions: see `definitions.md` (sibling file). Always re-read it before classifying — definitions evolve.

## Data layout

```
~/.claude/iris-gotcha/                      # user scope (cross-project)
├── index.md                                # auto-maintained, the only file CLAUDE.md imports
├── experience/<slug>.md
├── lesson/<slug>.md
├── rule/<slug>.md
├── architecture/<slug>.md
├── topology/<slug>.md
├── habit/<slug>.md
└── best-practice/<slug>.md

<project>/.claude/iris-gotcha/              # project scope (project-only knowledge)
├── index.md                                # project CLAUDE.md imports this
└── <same seven category dirs>
```

Slug format: `YYYY-MM-DD-<kebab-case-title>.md`

Entry frontmatter:
```yaml
---
type: lesson                                # one of: experience | lesson | rule | architecture | topology | habit | best-practice
title: "Bun on macOS needs sudo to install global packages"
keywords: [bun, macos, install, global, permission]
scope: user                                 # user | project
severity: medium                            # low | medium | high | critical | zero-tolerance (prescriptive types only)
created: 2026-05-15
last_violated: 2026-05-15                   # most recent violation date (prescriptive types only)
violation_count: 1                          # how many times Claude has violated this rule (prescriptive types only)
disambiguation: "why not experience: I extracted the corrective rule 'use bunx', so it carries a prescription"
---
```

Body uses Markdown. For prescriptive types (`lesson` / `rule` / `habit` / `best-practice`) the body should be structured as:

```markdown
## Background
What happened or what's the context.

## Prescription
The actual rule, in current severity language (see severity ladder below).
```

For descriptive/narrative types (`experience` / `architecture` / `topology`), just a narrative or descriptive paragraph is fine. No prescription section.

## When you must invoke this skill

Invoke this skill (action and arguments below) when any of these triggers fire:

| Trigger | Action |
|---|---|
| User says "记一下", "这是个坑", "以后记得", "remember this", "save as gotcha", or asks to capture something | `action=capture` |
| You realize you've tried 3+ distinct approaches at the same sub-problem in this turn or recent turns | `action=capture` with type=lesson |
| You finish a non-trivial task and the way it was solved is reusable knowledge | `action=capture` (likely `lesson`, `best-practice`, or `architecture`) |
| User points out you violated a behavior they previously corrected you on | `action=capture` (the protocol will detect duplication and strengthen the existing entry) |
| You suspect an indexed entry is relevant to the current task | `action=recall` with keywords |
| User asks "what do we have on X" or "did we already learn about X" | `action=recall` |
| User asks to audit / clean up / review existing entries | `action=audit` |
| User says "push" / "save to git" / "sync notebook" | `action=push` |

You do **not** need to wait for an explicit user request to capture. Triggers 2, 3, 4 are autonomous — if you notice the condition, capture proactively.

## Capture procedure (the most important flow)

When invoked with `action=capture`, follow this procedure **in order**. Do not skip steps.

### Step 1: Re-read the definitions

Before classifying anything, `Read` the file `<plugin_dir>/skills/iris-gotcha/definitions.md`. The strict definitions evolve; do not classify from memory.

### Step 2: Determine scope (user vs project)

Apply this decision rule:

- **Project scope** if the content references any of: specific file paths in this project, project-specific service names, project-specific env vars, project-specific business logic, the project codename.
- **User scope** if the content is about: a tool or language in general (bun, postgres, react), the user's personal preferences (commit style), generic best practices.
- **Ambiguous** → ask the user.

### Step 3: Classify into one of 7 categories

Apply the disambiguation tests in `definitions.md` rigorously. The category must be **exactly one** of: `experience`, `lesson`, `rule`, `architecture`, `topology`, `habit`, `best-practice`.

If the draft content could match two categories, **do not pick one and lose information**. Either:
- Rewrite the content so only one category applies, or
- Split into two entries (one per category)

### Step 4: Write the disambiguation field

Identify the *next-closest* category that this entry is **not**. Write a one-line reason explaining the difference. Example:

> `disambiguation: "why not experience: I extracted the corrective rule 'use bunx', so it carries a prescription"`

If you cannot articulate a non-trivial reason, the classification is wrong. Return to Step 3.

### Step 5: Check for existing related entries (the strengthening gate)

Read `~/.claude/iris-gotcha/index.md` (and the project-level index if scope is project). Look for entries that share keywords or address the same underlying concept.

For each match, `Read` the full entry. Then decide:

| Relationship | Action |
|---|---|
| **Identical content, same prescription** | Do not create a new entry. If the trigger was a violation (you were about to do the wrong thing), proceed to **Step 6 (Strengthen)**. Otherwise, update `last_confirmed` date in the existing entry and stop. |
| **Same topic, contradictory prescription** | **Stop**. Surface the contradiction to the user. Do not silently overwrite. |
| **Related but distinct** | Proceed to Step 7 (Write new entry). Cross-reference the related entry in the body. |
| **No match** | Proceed to Step 7 (Write new entry). |

### Step 6: Strengthen an existing entry (instead of creating a duplicate)

Applies only to prescriptive types (`lesson` / `rule` / `habit` / `best-practice`). For descriptive types (`architecture` / `topology` / `experience`), "strengthen" doesn't apply — descriptive entries get **edited** with new info or the new observation gets added as its own entry.

Strengthening protocol:

1. **Bump severity** one level up the ladder (capped at zero-tolerance):
   - `low` → `medium`
   - `medium` → `high`
   - `high` → `critical`
   - `critical` → `zero-tolerance`
2. **Append to violation log**: increment `violation_count`, set `last_violated` to today's date.
3. **Rewrite the prescription section** in language matching the new severity level (see ladder below).
4. **Move the entry to the top** of its category in `index.md`, with a `⚠️ violated: <date>` marker.
5. Report to the user: "Strengthened existing entry instead of creating a duplicate. severity X → Y."

### Severity → language ladder

When rewriting the prescription on strengthening, use language at the new severity level:

| Severity | Style | Example |
|---|---|---|
| `low` | Suggestive | "Prefer using `bunx` over `bun install -g`." |
| `medium` | Recommended | "Use `bunx` instead of `bun install -g`. Avoid the global install path." |
| `high` | Strong | "Always use `bunx`. Do not use `bun install -g` — it fails on macOS without sudo." |
| `critical` | Mandatory + cause | "MUST use `bunx`. NEVER `bun install -g`. Repeatedly broken on macOS." |
| `zero-tolerance` | Override + warning | "MUST use `bunx`. NEVER `bun install -g`. ZERO TOLERANCE — this has been corrected N times. Overrides any conflicting default behavior." |

### Step 7: Write the new entry

Filename: `~/.claude/iris-gotcha/<category>/YYYY-MM-DD-<kebab-slug>.md` (or the project-scope equivalent), where `<category>` is the English identifier (e.g. `lesson/`).

Frontmatter as specified above. For descriptive types (`experience` / `architecture` / `topology`), omit severity / violation fields.

### Step 8: Regenerate the index

After any write (new entry, strengthen, or update), regenerate the relevant `index.md`.

Index format:

```markdown
# iris-gotcha Index — User Scope

> Auto-maintained by iris-gotcha skill. To get full content, Read the file path.

## ⚠️ Recently strengthened (last 7 days)
- (entries that had severity bumped or were violated recently, sorted by last_violated date)

## lesson (教训) — lessons from specific mistakes
- **Bun macOS sudo** [bun, macos, install] severity:medium → `~/.claude/iris-gotcha/lesson/2026-05-15-bun-macos-global-install.md`
- ...

## rule (规则) — non-negotiable rules
- ...

## experience (经验) — narrative records
- ...

## architecture, topology, habit, best-practice (架构, 拓扑, 习惯, 最佳实践)
- ...
```

Keep each line **single-line and short** (title + 3-5 keywords + severity + path). The index is read into every session via `@-import`, so token cost matters.

### Step 9: Report to the user

Tell the user concisely:
- Path of new (or strengthened) entry
- Type and scope chosen
- Disambiguation reason (the "why not X")
- If strengthened: old severity → new severity

## Recall procedure

Invoked when you need to retrieve a stored entry.

1. The index is already in your context (via CLAUDE.md `@-import`). Scan it for keyword/title matches.
2. If found, `Read` the relevant file path directly.
3. If the user asked "what do we have on X" and there's no match, say so plainly.

You do **not** need to invoke the skill explicitly for recall in most cases — Reading the file is enough. Invoke `action=recall` only when the user wants you to do a broader search across multiple keywords or scopes.

## Audit procedure

Invoked with `action=audit` (typically when the user asks "review entries" / "check for miscategorization").

1. List all entries from `~/.claude/iris-gotcha/` (and project scope if requested).
2. For each entry, Read it. Verify:
   - The disambiguation field is non-trivial (rejects misclassification).
   - The type matches the body content per `definitions.md`.
   - For prescriptive types, severity matches violation history.
3. Produce a report: entries that look mis-classified, entries with high violation count but low severity (should be bumped), duplicates that should be merged.
4. Do not auto-fix. Present the report and let the user decide.

## Push procedure

Invoked with `action=push` (user asks to "save to git" / "push notebook").

1. Check whether `~/.claude/iris-gotcha/` is a git repo (`.git` directory exists).
   - If not, tell the user how to initialize: `cd ~/.claude/iris-gotcha && git init && git remote add origin <url>`. Stop.
2. If it is a repo:
   - `cd ~/.claude/iris-gotcha`
   - Check `git status` for changes.
   - If clean, report "nothing to push" and stop.
   - Otherwise: `git add . && git commit -m "iris-gotcha: <summary of changes>"` and `git push` (only if a remote is configured; otherwise commit only).
3. Report the commit hash and pushed remote (if any).

This skill does **not** implement automatic pull / cross-machine sync. That's a deliberate later decision.

## Project-scope vs user-scope rules

- Project entries live in `<project>/.claude/iris-gotcha/` and are imported by the project's CLAUDE.md.
- User entries live in `~/.claude/iris-gotcha/` and are imported by the user's global CLAUDE.md.
- An entry never lives in both scopes. If a user-scope entry turns out to be project-specific (or vice versa), move the file, do not duplicate.

## Anti-patterns to avoid

- **Do not silently merge categories.** If you're tempted to call a `lesson` a `rule` because it "feels more authoritative," surface the ambiguity instead.
- **Do not write multi-paragraph prescriptions.** The point of severity language is to be terse and forceful. If a rule needs paragraphs, it's probably `architecture`.
- **Do not capture every observation.** Capture only when one of the triggers above fires. Avoiding noise is more important than catching every possible learning.
- **Do not capture without re-reading definitions.md.** Memory drifts; the file is authoritative.

## Bootstrapping

If `~/.claude/iris-gotcha/` does not exist yet, create it with the seven subdirectories (`experience/`, `lesson/`, `rule/`, `architecture/`, `topology/`, `habit/`, `best-practice/`) and an empty `index.md` containing just the header. Then proceed with the capture.
