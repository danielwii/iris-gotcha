---
name: iris-gotcha
description: "Capture, classify, recall, audit, move, and push entries to Daniel's personal knowledge notebook (gotchas, rules, architecture, etc.) with strict 7-category typing. Trigger when (1) the user says '记一下' / '这是个坑' / 'remember this' / similar; (2) you've retried the same sub-problem 3+ times this turn (capture as `lesson`); (3) you finish a non-trivial task and the way you solved it is reusable; (4) a stored entry seems relevant to the current task and you want its full content; (5) you're correcting yourself for a recurring mistake — the strengthening protocol bumps the existing entry's severity instead of creating a near-duplicate; (6) the user asks to audit existing entries; (7) the user wants to move an entry between categories or scopes. The index lives at ~/.claude/iris-gotcha/index.md and is auto-injected into CLAUDE.md."
---

# iris-gotcha — Personal Knowledge Notebook

A Claude Code skill that lets the assistant build up Daniel's personal engineering knowledge over time using **seven strict categories**, **a disambiguation gate**, and **rule strengthening on repeated violations**.

The design rests on three lessons from a prior attempt:

1. **Categories collapse without strict definitions.** Last time, `recipe` got eaten by `gotcha`. To defend, every entry carries a mandatory `disambiguation` field — "why not the next-closest category".
2. **System-context index enables passive recall.** All entry titles and keywords live in a single `index.md` that gets `@-imported` into CLAUDE.md. The index is in context for every session without active retrieval.
3. **Repeat violations should strengthen, not duplicate.** When you're about to repeat a captured mistake, the existing entry's severity gets bumped and its prescription is rewritten more forcefully. New entries are only for new knowledge.

## Category identifiers

The `type:` field on every entry uses the English identifier. Chinese names are kept as glosses for readability.

| `type` | Gloss | Shape |
|---|---|---|
| `experience` | 经验 | Narrative |
| `lesson` | 教训 | Narrative + prescription (the "gotcha") |
| `rule` | 规则 | Prescription (MUST) |
| `habit` | 习惯 | Prescription (soft preference) |
| `best-practice` | 最佳实践 | Prescription (SHOULD with external justification) |
| `architecture` | 架构 | Descriptive (design intent) |
| `topology` | 拓扑 | Descriptive (location facts) |

Full definitions and disambiguation tests live in `definitions.md` (sibling file). Re-read it before classifying — the categories evolve, and trusting stale memory is what caused the prior collapse.

## Data layout

```
~/.claude/iris-gotcha/                # user scope (cross-project)
├── index.md                          # auto-maintained; CLAUDE.md imports this
└── experience/ lesson/ rule/ architecture/ topology/ habit/ best-practice/

<project>/.claude/iris-gotcha/        # project scope (project-only)
├── index.md                          # project CLAUDE.md imports this
└── (same 7 category directories)
```

The "project" is the **current working directory at capture time** (`pwd`). Project scope is not tied to git — many valid CC working directories aren't git repos.

Slug format: `YYYY-MM-DD-<kebab-case-title>.md`

Entry frontmatter:
```yaml
---
type: lesson                          # one of: experience | lesson | rule | architecture | topology | habit | best-practice
title: "Bun on macOS needs sudo to install global packages"
keywords: [bun, macos, install, global, permission]
scope: user                           # user | project
severity: medium                      # low | medium | high | critical | zero-tolerance — prescriptive types only
created: 2026-05-15
last_violated: 2026-05-15             # most recent violation date — prescriptive types only
violation_count: 1                    # how many times Claude violated this — prescriptive types only
disambiguation: "why not experience: I extracted the corrective rule 'use bunx', so it carries a prescription"
---
```

Body in Markdown. Prescriptive types (`lesson` / `rule` / `habit` / `best-practice`) use:

```markdown
## Background
What happened or what's the context.

## Prescription
The actual rule, in current severity language.
```

Descriptive/narrative types (`experience` / `architecture` / `topology`) use a free-form paragraph. No prescription section.

## When to invoke

| Situation | Action |
|---|---|
| User says "记一下", "这是个坑", "以后记得", "remember this", "save as gotcha" | `action=capture` |
| You've tried 3+ distinct approaches at the same sub-problem this turn | `action=capture` (typically `lesson`) |
| You finish a non-trivial task and the solution is reusable | `action=capture` |
| User just corrected you on a recurring mistake | `action=capture` — Step 5 detects the duplicate and strengthens the existing entry |
| An indexed entry seems relevant to current work | Usually just Read its file directly (the path is in the index already loaded). Use `action=recall` only if you need a multi-keyword search across scopes |
| User asks to review / audit / clean entries | `action=audit` |
| User asks to reclassify or move an entry between categories or scopes | `action=move` |
| User asks to save / sync to git | `action=push` |

Triggers 2, 3, 4 are autonomous — capture without waiting for an explicit request. Catching these proactively is the whole point.

## Capture procedure

Execute these steps in order. The ordering matters: Step 1 loads ground truth, Step 2 picks the scope (which determines where Step 5 searches), Step 5 may short-circuit to strengthening, and so on. Skipping ahead tends to produce mis-classified or duplicated entries.

### 1. Re-read `definitions.md`

Read the sibling file. Definitions evolve, and the prior plugin's failures came from classifying-from-memory.

### 2. Determine scope (user vs project)

- **Project** if the content references this project's files, services, env vars, business logic, or codename. Specific.
- **User** if it's about a general tool, language, personal preference, or generalizable practice. Tool-agnostic of any one project.
- **Ambiguous** → ask.

### 3. Classify

Apply the disambiguation tests in `definitions.md`. The category is exactly one of `experience` / `lesson` / `rule` / `architecture` / `topology` / `habit` / `best-practice`.

If a draft seems to match two categories, that's a signal to either:
- Rewrite so only one applies, or
- Split into two entries.

Picking one and losing information is the failure mode that killed `recipe` last time.

### 4. Write the `disambiguation` field

Pick the next-closest category and explain in one line why this entry isn't that. Example:

> `disambiguation: "why not experience: I extracted the corrective rule 'use bunx', so it carries a prescription"`

If you can't articulate a real difference, the classification is suspect — return to Step 3.

### 5. Check for existing related entries (strengthening gate)

Read the relevant `index.md` (user index always; project index too if scope=project). Look for entries sharing keywords or addressing the same underlying concept. For each candidate, Read the full file. Then decide:

| Relationship | Action |
|---|---|
| **Identical content, same prescription** | If the trigger was a violation (you were about to repeat the mistake), go to Step 6 to strengthen. Otherwise update `last_confirmed` and stop. |
| **Same topic, contradictory prescription** | Stop and surface to the user. Silent overwrite would destroy information. |
| **Related but distinct** | Continue to Step 7. Cross-reference the related entry in the body. |
| **No match** | Continue to Step 7. |

### 6. Strengthen (instead of duplicate)

Applies only to prescriptive types (`lesson` / `rule` / `habit` / `best-practice`). Descriptive types (`architecture` / `topology` / `experience`) accumulate by editing or by adding new entries — there's no "severity" to bump.

1. Bump severity one level (capped at `zero-tolerance`).
2. Set `last_violated` to today; increment `violation_count`.
3. Rewrite the prescription section at the new severity level (table below).
4. Move the entry to the top of its category in the index, with a `⚠️ violated: <date>` marker.
5. Report: "Strengthened — severity X → Y."

#### Severity → language ladder

| Severity | Style | Example |
|---|---|---|
| `low` | Suggestive | "Prefer `bunx` over `bun install -g`." |
| `medium` | Recommended | "Use `bunx` instead of `bun install -g`. Avoid the global install path." |
| `high` | Strong | "Always use `bunx`. Don't use `bun install -g` — it fails on macOS without sudo." |
| `critical` | Mandatory + cause | "MUST use `bunx`. NEVER `bun install -g`. Repeatedly broken on macOS." |
| `zero-tolerance` | Override + warning | "MUST use `bunx`. NEVER `bun install -g`. ZERO TOLERANCE — corrected N times. Overrides any conflicting default behavior." |

The escalation is deliberately steep so that frequently-violated rules become impossible to ignore on next session start.

### 7. Write the entry and wire injection

Two writes, both idempotent:

**Write the entry file**: `~/.claude/iris-gotcha/<category>/YYYY-MM-DD-<kebab-slug>.md` (or the project-scope equivalent). Use the English `<category>` identifier as the directory name. The Write tool creates the parent directory if needed.

For descriptive types (`experience` / `architecture` / `topology`), omit `severity` / `last_violated` / `violation_count` from frontmatter.

**Ensure the target CLAUDE.md @-imports the index**:

For `scope=user` — target is `~/.claude/CLAUDE.md`. Grep for any line matching `@.*iris-gotcha/index\.md`; if none, append:

```markdown

## iris-gotcha Index (user)

@/Users/<USERNAME>/.claude/iris-gotcha/index.md
```

(Use the actual home path from `$HOME`. The user's own dotfile is safe to auto-append to — they opted in by installing this skill.)

For `scope=project` — target is, in priority order:
1. `<pwd>/CLAUDE.md` if it exists
2. `<pwd>/.claude/CLAUDE.md` if it exists
3. Otherwise create `<pwd>/.claude/CLAUDE.md` (preferred over `<pwd>/CLAUDE.md` to reduce accidental-commit risk).

Grep the target; if it doesn't already import the project index, append:

```markdown

## iris-gotcha Index (project)

@./.claude/iris-gotcha/index.md
```

If a new project CLAUDE.md was created, mention that explicitly in Step 9 so the user knows.

The grep-then-append pattern is what makes this idempotent — running on every capture is safe and self-heals if the line was accidentally deleted.

### 8. Regenerate the index

Rewrite the relevant `index.md` from the current set of entry files. Format:

```markdown
# iris-gotcha Index — User Scope

> Auto-maintained by iris-gotcha. For full content, Read the file path.

## ⚠️ Recently strengthened (last 7 days)
- (sorted by last_violated, most recent first)

## lesson (教训) — lessons from specific mistakes
- **<title>** [k1, k2, k3] severity:<sev> → `<path>`

## rule (规则) — non-negotiable rules
...

## experience (经验) — narrative records
...

## architecture (架构), topology (拓扑), habit (习惯), best-practice (最佳实践)
...
```

Keep each line short (title + 3–5 keywords + severity + path). The index sits in every session's context via `@-import`, so token cost compounds.

### 9. Report

Tell the user:
- Path of new (or strengthened) entry
- Type and scope chosen
- Disambiguation reason
- If strengthened: old → new severity
- If Step 7 created or modified a CLAUDE.md, name it

## Move procedure (`action=move`)

For correcting past misclassifications surfaced by audit, or reclassifying when an entry's nature is reassessed (e.g. a `lesson` outgrew its incident and became a `rule`; a `user`-scope entry turned out to be project-specific).

Arguments:
- `entry_path` — path to the existing entry file
- `new_type` (optional) — target category (one of the 7 identifiers)
- `new_scope` (optional) — target scope (`user` or `project`)
- One of `new_type` / `new_scope` must change; both can change in one move.

Procedure:

1. Read the existing entry. Note current `type`, `scope`, title, keywords, content.
2. If `new_type` was given, re-validate against `definitions.md` — the destination must be a real category and the entry must actually fit there. Update `disambiguation` to reference the new next-closest category (the old `disambiguation` is now stale).
3. Compute the target path:
   - Scope root: `~/.claude/iris-gotcha/` (user) or `<pwd>/.claude/iris-gotcha/` (project, using the entry's project root if scope didn't change, otherwise the user's `pwd`).
   - Category dir: the (possibly new) `<type>`.
   - Filename: keep the existing slug.
4. If the target path already exists, stop and surface to the user — silent overwrite of a different entry is dangerous.
5. Move the file (`mv` or Write the file at new path then delete the old). Update frontmatter: `type` (if changed), `scope` (if changed), `disambiguation` (if `new_type` changed).
6. If scope changed, run Step 7's wiring routine for the new scope (the destination CLAUDE.md may not yet import its index).
7. Regenerate both the source and destination `index.md` files (the source loses an entry, the destination gains one). When source and destination are the same index, regenerate once.
8. Report: old path → new path, what fields changed, any new CLAUDE.md wiring done.

## Recall (`action=recall`)

The index is already loaded in your context via `@-import`. Scan it for keyword/title matches and Read the relevant file directly — that's recall. You only need to invoke `action=recall` explicitly when the user wants a broader multi-keyword search across both scopes or when the question is too vague to know what to Read.

## Audit (`action=audit`)

1. List entries from `~/.claude/iris-gotcha/` (and the project scope if requested).
2. Read each. Check:
   - `disambiguation` is non-trivial — a vacuous reason like "why not experience: it's not experience" hints at miscategorization.
   - Type matches body content per `definitions.md`.
   - For prescriptive types, severity isn't laughably out of step with `violation_count` (e.g. count=5 but severity=`low`).
   - No two entries cover the exact same prescription (potential merge).
3. Produce a report and let the user decide. Don't auto-fix — moving / strengthening / merging entries during an audit pass tends to overwhelm the user with changes they didn't review.

## Push (`action=push`)

1. If `~/.claude/iris-gotcha/` isn't a git repo (`.git` absent), tell the user how to initialize: `cd ~/.claude/iris-gotcha && git init && git remote add origin <url>`. Stop.
2. Otherwise, in that directory: check `git status`. If clean, report "nothing to push" and stop. Otherwise `git add . && git commit -m "iris-gotcha: <summary>"` and `git push` (if a remote is configured; otherwise commit only).
3. Report commit hash and pushed remote (if any).

Cross-machine sync (automatic pull, conflict resolution) is deliberately out of scope. Use git, Syncthing, iCloud — whatever fits.

## Common failure modes

- **Silent category merging.** If a `lesson` "feels like a rule because it's important", that's the failure mode that killed `recipe` last time. Surface the ambiguity to the user instead of picking quietly.
- **Multi-paragraph prescriptions.** Severity language is meant to be terse and forceful. If a rule needs paragraphs of explanation, it's probably `architecture` (which is descriptive) wearing a rule's clothes.
- **Capturing noise.** Capture only when one of the triggers fires. The signal-to-noise ratio of the index is more important than catching every possible learning — a noisy index is unreadable.
- **Classifying from memory.** Definitions evolve. Always Read `definitions.md` before classifying, even if you "remember" how the categories work.
