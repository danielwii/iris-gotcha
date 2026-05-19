---
name: iris-gotcha
description: "iris-gotcha is a signal-to-noise optimizer for AI context — it captures only the engineering knowledge that's worth a permanent slot in every future session's context. Use when the user wants to record a learning the AI wouldn't already know from training, AND that future sessions will actually reference ('记一下', '这是个坑', 'remember this', 'save as gotcha', '以后记得'); when you've retried the same sub-problem 3+ times in a turn (signals an undocumented gotcha); when finishing a non-trivial task whose solution is reusable knowledge; when you suspect a stored entry applies to the current task and you need its full content; when the user just corrected you on a recurring mistake; when the user asks to audit, reclassify, move, push, or generate an overview of stored entries. Do NOT capture safety/security defaults the AI already enforces (training-redundant), or low-signal entries no future session will reference (pays context tax for no behavioral change)."
---

# iris-gotcha — Signal-to-Noise Optimizer for AI Context

## Why this exists (the core purpose and evaluation criterion)

iris-gotcha is **a signal-to-noise optimizer for the AI's always-on context**. Every session that starts on a machine where this plugin is installed pays a fixed token tax: the user-scope index, plus any project-scope indexes for the cwd's CLAUDE.md ancestry, are inlined via `@-import`. The plugin's job is to make that tax pay for itself by carrying **only** high-signal, training-gap engineering knowledge.

This is **the criterion against which every future design choice and every individual capture should be evaluated**:

> **Does this raise the ratio of "behavior-changing knowledge" to "context tokens consumed"?**
> Or does it bloat the always-on context with material the AI was going to handle correctly anyway, or with entries no future session will actually reference?

If a proposed feature, doctrine change, or capture entry doesn't visibly raise that ratio, it doesn't belong here. Token economy is the load-bearing engineering value. Categorization, disambiguation, strengthening, split tests, overview generation — every mechanism in this skill exists to push signal density up or to keep noise from accumulating.

### Two gates every capture must pass

| Gate | Question | Failure means |
|---|---|---|
| **1. Training-gap** | Would a fresh Claude session, with no notebook, handle this correctly anyway? | The entry is redundant with training — it pays context tax for zero behavioral change. |
| **2. Signal value** | Will future sessions actually reference this — to recall a fact, avoid a mistake, or change a decision? Or is it a one-time observation that won't pay rent in next month's contexts? | The entry occupies a permanent slot but contributes nothing on most future loads. |

Capture only when **both** gates pass. If the signal value isn't obvious, default to **not capturing** rather than capturing and hoping. Undertrigger is cheap (you can capture later when the value becomes clear). Overtrigger is expensive (entries are sticky — once in the index, removing them takes audit + delete cycles, and meanwhile they tax every session).

### Implications for every mechanism in this skill

| Mechanism | How it serves signal-to-noise |
|---|---|
| **Strict 6-category typing + disambiguation** | Prevents classification-layer noise (`recipe` collapsing into `gotcha`); each category has a distinct retrieval pattern |
| **Lazy body loading** | Bodies only enter context when the AI Reads them; the always-on tax is just the index line |
| **One-line index format** | Maximizes signal density per token in the always-on layer |
| **Step 0 training-gap gate** | First-stage signal filter — rejects redundant-with-training content before classification |
| **Step 5 strengthening over duplication** | Keeps the index from inflating when the same lesson recurs; bumps severity instead of adding a near-duplicate |
| **Severity ladder + zero-tolerance** | Attention budget allocation — high-severity entries occupy more cognitive space in the index |
| **`Recently strengthened` section** | Promotes high-signal (frequently-violated or recently-acted-on) entries to the top of the always-on layer |
| **`action=audit`** | Periodic noise removal — finds stale or miscategorized entries |
| **`action=overview`** | Synthesis layer — replaces N raw architecture/topology entry loads with one digested document |
| **Active split test (3-probe)** | Avoids hybrid entries that carry multiple weakly-related signals; each entry stays focused |
| **Markdown-only / no-runtime** | Keeps the plugin itself cheap (no per-session bootstrap cost beyond file read) |

When in doubt about any change to this skill — to a procedure, a category, a default — ask: **would this raise signal density, or lower it?** If you can't articulate the signal answer, the change probably shouldn't ship.

This principle also governs **future optimization and evaluation**: when proposing v0.9.0+, when reviewing PRs, when auditing the index, the same test applies. "Does this carry its weight in tokens?" is the load-bearing question. Everything else is downstream.

## What goes in (and what doesn't)

What clears both gates:

- **Specific tool gotchas** the AI hasn't seen (e.g. `Prisma v7.8 dropped datasourceUrl, use driver adapters`) — future sessions hit this repeatedly until the library evolves past it.
- **Your project's structure** (services, ports map, package layout, design intent) — referenced whenever the AI works on the project.
- **Personal / team preferences** (commit style, error-shape choices) — referenced whenever the AI generates code touching that surface.
- **Workflow recipes** for your specific environment (`ArgoCD token expired? kubectl patch the Application CRD`) — signal scales with how often the workflow recurs.
- **Quality lessons** from specific bugs that represent a class (e.g. `upsert needs both create + update branches mirrored — caught in PR #271`) — signal scales with how often the class shows up.
- **Recurring mistakes the AI keeps making** — these get *strengthened* (severity bumped), not duplicated. Strengthening is signal concentration, not signal addition.

What fails the gates and shouldn't go in:

- Safety / security defaults the AI already enforces (training-redundant)
- Generic best practices the AI follows by default (training-redundant)
- One-time historical observations with no behavioral implication (signal = 0; if you want a trace, use `claude-mem`)
- Hyper-niche details that will be referenced once and never again (signal too low to justify permanent context cost)
- Restatements of CLAUDE.md content (duplicative noise across two layers)

## Design principles

1. **Signal-to-noise is the load-bearing value.** Every other principle below follows from it.
2. **Capture only training-gap, signal-positive knowledge.** Step 0 enforces gate 1; the capturer's judgment (or a quick ask) enforces gate 2.
3. **Categories collapse without strict definitions.** Last time, `recipe` got eaten by `gotcha`. Every entry carries a mandatory `disambiguation` field naming the next-closest category and explaining why it's not that — this keeps the classification layer from becoming noise itself.
4. **System-context index enables passive recall.** All entry titles + keywords live in a single `index.md` that gets `@-imported` into CLAUDE.md, so the AI sees them every session without active retrieval. The index format is one line per entry to keep the always-on token cost minimal.
5. **Repeat violations strengthen, not duplicate.** When the AI is about to repeat a captured mistake, the existing entry's severity is bumped and its prescription is rewritten more forcefully — only for *behavioral* types (lesson/rule/habit/best-practice), not for *descriptive* types (architecture/topology). Concentration of signal, not addition.

## Category identifiers

The `type:` field uses the English identifier. Chinese names are kept as glosses.

| `type` | Gloss | Shape | Typical content |
|---|---|---|---|
| `lesson` | 教训 | Narrative + prescription (the "gotcha") | Specific tool/library gotcha you hit; corrective rule extracted |
| `rule` | 规则 | Prescription (MUST) | Project-specific MUST, or user override of AI default behavior |
| `habit` | 习惯 | Prescription (soft preference) | Personal / team preference not enforceable elsewhere |
| `best-practice` | 最佳实践 | Prescription (SHOULD with external justification) | Class-level recommendation the AI wouldn't already follow |
| `architecture` | 架构 | Descriptive (design intent) | How a project's components fit together and why |
| `topology` | 拓扑 | Descriptive (location facts) | Where services / files / endpoints live |

**Six categories, not seven.** `experience` (pure narrative, no prescription) was dropped in v0.6.0 — it accumulated dead weight (history without behavior change). For session history and observations, use `claude-mem` instead; iris-gotcha is for knowledge that changes future behavior or orients understanding.

Full definitions and disambiguation tests live in `definitions.md` (sibling file). Re-read it before classifying — the categories evolve, and trusting stale memory is what caused the prior `recipe`-collapse.

## Data layout

```
~/.claude/iris-gotcha/                # user scope (cross-project)
├── index.md                          # auto-maintained; CLAUDE.md imports this
├── overview.md                       # optional, generated by action=overview
└── lesson/ rule/ architecture/ topology/ habit/ best-practice/

<project>/.claude/iris-gotcha/        # project scope (project-only)
├── index.md                          # project CLAUDE.md imports this
├── overview.md                       # optional, generated by action=overview
└── (same 6 category directories)
```

The "project" is the **current working directory at capture time** (`pwd`). Project scope is not tied to git — many valid CC working directories aren't git repos.

Slug format: `YYYY-MM-DD-<kebab-case-title>.md`

Entry frontmatter:
```yaml
---
type: lesson                          # one of: lesson | rule | architecture | topology | habit | best-practice
title: "Bun on macOS needs sudo to install global packages"
keywords: [bun, macos, install, global, permission]
scope: user                           # user | project
severity: medium                      # low | medium | high | critical | zero-tolerance — prescriptive types only
created: 2026-05-15
last_violated: 2026-05-15             # most recent violation date — prescriptive types only
violation_count: 1                    # how many times Claude violated this — prescriptive types only
disambiguation: "why not best-practice: this prescription comes from one specific incident on macOS, not an industry-consensus recommendation"
---
```

**Initial capture (no prior incident)**: set `violation_count: 0` and omit `last_violated`. This is the typical shape for a `rule` / `best-practice` / `habit` captured preemptively. The above example shows the post-incident shape (`violation_count: 1`, `last_violated` set to the incident date) — typical for a `lesson` captured right after the mistake.

Body in Markdown. Prescriptive types (`lesson` / `rule` / `habit` / `best-practice`) use:

```markdown
## Background
What happened or what's the context.

## Prescription
The actual rule, in current severity language.
```

Descriptive types (`architecture` / `topology`) use a free-form paragraph. No prescription section.

## When to invoke

| Situation | Action |
|---|---|
| User says "记一下", "这是个坑", "以后记得", "remember this", "save as gotcha" | `action=capture` (run the **triage announcement** first — see below) |
| You've tried 3+ distinct approaches at the same sub-problem this turn | `action=capture` (typically `lesson`) |
| **End of a non-trivial debugging arc** (problem resolved OR abandoned after multi-turn isolation work) | **Run end-of-debug retrospective** — see below; if it yields a gotcha, `action=capture` |
| You finish a non-trivial task and the solution is reusable | `action=capture` |
| User just corrected you on a recurring mistake | `action=capture` — Step 5 detects the duplicate and strengthens the existing entry |
| An indexed entry seems relevant to current work | Usually just Read its file directly (the path is in the index already loaded). Use `action=recall` only if you need a multi-keyword search across scopes |
| User asks to review / audit / clean entries | `action=audit` |
| User asks to reclassify or move an entry between categories or scopes | `action=move` |
| User asks to generate / regenerate a project overview / architecture summary | `action=overview` |
| New session in an unfamiliar project — orient yourself by reading the synthesized overview if it exists, or generating one if it doesn't | `action=overview` |
| User asks to save / sync to git | `action=push` |

Triggers 2, 3, 4 are autonomous — capture without waiting for an explicit request. Catching these proactively is the whole point.

### End-of-debug retrospective (since v0.8.0)

A debugging arc is **multi-turn isolation work**: tracing a problem through code/logs/configs, trying hypotheses, eventually landing on a root cause (or giving up). When such an arc concludes — **whether resolved or abandoned** — pause for a one-step retrospective before moving on:

> **"Was the root cause / fix something a fresh Claude session would not have known a priori?"**

If **yes**: surface it to the user as a gotcha proposal. Something like:
> "That took a few turns — the root cause was X (a Bun macOS install permission quirk). Worth capturing as a `lesson`? It would land in `~/.claude/iris-gotcha/lesson/`."

Then await user confirmation (don't capture unilaterally — debug arcs are common and not every one is gotcha-worthy).

If **no** (it was a routine bug fix, well-known issue, or the AI should have known): say so explicitly to close the arc, but don't capture.

This trigger is **stronger than the retry-count trigger**: retry-count fires during the arc (mid-struggle); retrospective fires at the end and looks back at what was actually learned. They complement each other.

Skip the retrospective only if the arc was trivial (1-2 turns, obvious fix). For anything that felt like real investigation, run it.

### Triage announcement for user-triggered captures (since v0.8.0)

When the user explicitly invokes capture (says "记一下", "remember this", "save as gotcha", "这是个坑"), **before writing any file**, surface what you found in the index. This makes the new-vs-update-vs-reference decision visible at the entry point, not buried in a Step 9 report after the fact.

Format:

> **Triage**: scanning user index (N entries) + project index (M entries) for keywords [...].
>
> Found: [one of]
> - **0 candidates** → proceeding as a fresh capture.
> - **1 candidate at `<path>` (identical prescription)** → this is a re-occurrence; I'll **strengthen** it (severity X → Y) instead of duplicating.
> - **1 candidate at `<path>` (related but distinct)** → I'll capture as new with `## Related` cross-reference to the existing entry.
> - **1 candidate at `<path>` (contradictory)** → **stopping**, surfacing the contradiction: existing says A, this says B. Which is right?
> - **2+ candidates** → listing them; asking user which is the right relationship.

Then proceed only after the user has the information. For the unambiguous cases (0 candidates / clear strengthen / clear cross-ref), proceed immediately after announcing. For ambiguous or contradictory cases, **wait for user input**.

For autonomous captures (triggers 2, 3, 4) the triage can be more terse (you're acting on your own initiative, the user hasn't asked yet), but still announce findings.

## Capture procedure

Execute these steps in order. The ordering matters: Step 0 is the gate that decides whether to capture at all; Step 1 loads ground truth; Step 2 picks the scope (which determines where Step 5 searches); Step 5 may short-circuit to strengthening; and so on. Skipping Step 0 produces redundant noise in the index; skipping later steps produces mis-classified or duplicated entries.

### 0. Training-gap gate (skip if AI would already handle this)

Before doing anything, ask:

> **Would a fresh Claude session, with no iris-gotcha notebook, handle this correctly anyway?**

If yes → **skip capture entirely**. Briefly explain to the user what you understood and why you're not capturing it (so they can override if they disagree).

Don't capture:
- Security defaults the model enforces by default (no secrets in logs, no SQL injection, no leaking API keys)
- Standard error-handling / input-validation / safety practices
- Industry-standard ML alignment behaviors
- Generic best-practices the AI follows by default (REST naming, immutability where appropriate, clean code basics)
- Restating something already in the project's CLAUDE.md / rules

Do capture:
- Specific tool versions and their specific quirks the AI may not have seen
- Project-internal facts (services, ports, package layouts, design choices, codenames)
- Personal/team preferences that diverge from neutral defaults
- Workflow recipes specific to your environment
- Specific past mistakes — especially recurring ones (those trigger Step 6 strengthening rather than fresh capture)

The notebook is a supplement to training, not a re-statement of it. If capturing this entry would, after the fact, feel like "the AI didn't need to be told this", you guessed wrong at this step — go back and skip.

### 1. Re-read `definitions.md`

Read the sibling file. Definitions evolve, and the prior plugin's failures came from classifying-from-memory.

### 2. Determine scope (user vs project) and project root

Two coupled decisions: is this user-scope or project-scope? And if project-scope, what's the actual project root?

**Scope decision**:
- **Project** if the content references a project's files, services, env vars, business logic, or codename. Specific.
- **User** if it's about a general tool, language, personal preference, or generalizable practice. Tool-agnostic.
- **Ambiguous** → ask.

**Project root determination (since v0.8.0)** — for `scope=project` only:

`pwd` is a *hint*, not the answer. Use judgment to find the actual project the capture is about:

- **Read the content.** Does it clearly name a specific project (codename, repo name, service)? If yes, locate that project's directory:
  - Often it's the current `pwd`, but it might be a sibling, child, or ancestor.
  - Common case: capturing about `iris-gotcha` from `~/Workspace` (pwd) → project root is `~/Workspace/iris-gotcha`, not `~/Workspace`.
- **Check the candidate path exists** (`ls <path>/.claude/iris-gotcha/` or `ls <path>` — the project dir should be a real directory).
- **If the content is project-specific but the project isn't obvious**, **ask the user**: "This content seems specific to project X but I'm in directory Y. Should I capture this under X, Y, or somewhere else?"
- **If the content names multiple projects**, that's usually a sign it should be split or moved to user-scope.

Don't blindly default to `pwd` when the content is obviously about a different project — captures landing in the wrong directory are hard to relocate later. **Ask once at write time** beats untangling misfiled entries.

The fallback chain when the AI genuinely can't infer:
1. Content names a specific project → use that project's root
2. Content is project-specific but ambiguous → ask user
3. Content is generic → `scope=user`, no project root needed
4. Only as last resort: use `pwd` as the project root

### 3. Classify

Apply the disambiguation tests in `definitions.md`. The category is exactly one of `lesson` / `rule` / `architecture` / `topology` / `habit` / `best-practice`.

If a draft seems to match two categories, that's a signal to either:
- Rewrite so only one applies, or
- Split into two entries.

Picking one and losing information is the failure mode that killed `recipe` last time.

### 4. Write the `disambiguation` field

Pick the next-closest category and explain in one line why this entry isn't that. Example:

> `disambiguation: "why not best-practice: derived from one specific incident, not from class-level / industry consensus"`

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

Applies only to prescriptive types (`lesson` / `rule` / `habit` / `best-practice`). Descriptive types (`architecture` / `topology`) accumulate by editing or by adding new entries — there's no "severity" to bump.

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

#### Initial severity (first capture)

When creating a *new* prescriptive entry (not strengthening an existing one), pick the starting severity by what a violation would actually cost. The ladder is for escalating over time as repeat violations accumulate — not a one-size-fits-all starting point.

**Use this concrete qualifier table — don't reason from the consequence class abstractly. If the entry matches a qualifier, start at that level.**

| Level | Qualifies if violation could… | Concrete example |
|---|---|---|
| `critical` | Violate a **trust boundary** (unauthorized access, accepting untrusted auth source as trusted, bypassing classifier / firewall / RBAC) — **OR** — corrupt data integrity, leak secrets, breach compliance, take production down | "Ein must require local-terminal auth for prod ops; TG/iris-channel isn't trusted" (trust boundary). "Never bypass the auth-gateway when calling internal services" (trust boundary) |
| `high` | Break production or a critical dev workflow; cause a real incident even if recoverable; produce wrong data that downstream systems consume | "Bump shared ECR tag → must sync both ArgoCD apps; missing one means production runs old code" |
| `medium` | Waste an hour or more; break a local build / CI; cause a fix-up commit; force a rollback that's recoverable in minutes | "Bun on macOS — use bunx, not `bun install -g`"; "psql needs the `?pgbouncer=true` param stripped" |
| `low` | Style nitpick, mild aesthetic preference, minor inefficiency you'd notice but not block on | "Prefer single TEXT column over structured enum/code/category" |
| `zero-tolerance` | (not for initial captures — only reached via strengthening after critical entries keep being violated) | — |

Two anchoring rules:

1. **Trust-boundary violations always start at `critical`.** Anything that means "the AI accepted an untrusted thing as trusted" or "the AI bypassed an access check" is a security-class failure. Even if no incident occurred yet.
2. **Don't default everything to `medium`.** A secrets-in-logs rule that starts at `low` will sit ignored at the bottom of the index for months; starting it at `critical` puts it where the consequence demands. If you're unsure between two levels, pick the higher one — escalation needs a violation, de-escalation needs an audit.

#### When language stops working

If an entry reaches `zero-tolerance` and *still* keeps getting violated, the prescription has saturated — louder text won't help. The real next step is outside iris-gotcha: a structural fix that removes the choice (lint rule, pre-commit hook, CI check, IDE save action, snippet, project scaffold). The brain has been told; the prescription needs to leave the brain and live in tooling.

When you strengthen an entry into `zero-tolerance`, surface this in Step 9: suggest the concrete structural fix that would obviate the rule.

### 7. Write the entry and wire injection

Two writes, both idempotent.

**Write the entry file**: `~/.claude/iris-gotcha/<category>/YYYY-MM-DD-<kebab-slug>.md` (or the project-scope equivalent). Use the English `<category>` identifier as the directory name. The Write tool creates the parent directory if needed.

For descriptive types (`architecture` / `topology`), omit `severity` / `last_violated` / `violation_count` from frontmatter.

#### `## Related` body section (mandatory for split entries)

When Step 3's split test produces multiple entries from the same raw observation, **every entry must reference the others** via a `## Related` body section. This is the only way future Claude can recover the full picture from any starting point in the split.

Format:

```markdown
## Related

- See `topology/2026-05-16-unee-shared-ecr-tag.md` — the structural fact this lesson references.
- See `architecture/2026-05-16-unee-split-for-restart-cadence.md` — why the two apps are split in the first place.
```

Each bullet: `- See <relative-or-absolute-path> — <one-line aspect this related entry covers>`. The "aspect" half is required so the reader knows whether to chase the link.

`## Related` may also appear on non-split entries when they happen to be relevant to existing entries (cross-reference is always allowed). But for split entries it is **mandatory** — write it before you finalize the file, or the split discipline is incomplete.

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

## architecture (架构), topology (拓扑), habit (习惯), best-practice (最佳实践)
...
```

Keep each line short (title + 3–5 keywords + severity + path). The index sits in every session's context via `@-import`, so token cost compounds.

### 9. Report

Tell the user, **in this order**:

1. **Step 5 outcome** (mandatory — even when nothing matched): one line like
   - `Step 5: scanned user index (5 entries), 0 matched — fresh capture.`
   - `Step 5: scanned user + project indexes (8 entries), 1 matched (`lesson/2026-04-20-run-lint-before-commit.md`) — strengthening, not new entry.`
   - `Step 5: scanned project index (3 entries), 1 matched with contradictory prescription — stopped, surfaced to user for resolution.`

   This makes the previously-invisible scan auditable. Don't omit it just because nothing matched — the "0 matched" report is the proof that the scan happened.

2. **Path(s)** of new (or strengthened) entry/entries. If Step 3 produced a split, list every file written and note they cross-reference each other via `## Related`.
3. **Type and scope** chosen, with severity if prescriptive.
4. **Disambiguation reason** (the "why not X" sentence).
5. **If strengthened**: old → new severity.
6. **CLAUDE.md wiring**: any file modified or created in Step 7.
7. **If your classification differs from a type the user explicitly named** (e.g. they said "记一下这条规则" but the content matches `lesson`), surface the disagreement plainly and offer `action=move` as the recovery path: *"Classified as `lesson` (not `rule` as you mentioned) because [reason]. If you'd prefer `rule`, I can `action=move` it."*

## Handling user disagreement with classification

If the user pushes back on the type or scope you chose, **do not** silently rewrite the file or recapture. Use `action=move` instead — it properly updates `type` / `scope` / `disambiguation`, regenerates both source and destination indexes, and (if scope changed) wires the destination CLAUDE.md. Silent rewriting loses the audit trail and may leave a stale index entry.

The same applies if you classify differently from a type the user explicitly named at capture time: state the disagreement in Step 9 and offer the move. Don't pre-emptively defer to the user's label without applying definitions.md — that's the rationalization that killed `recipe` last time. But once the disagreement is surfaced and the user maintains their view, *that* is when move runs.

## Move procedure (`action=move`)

For correcting past misclassifications surfaced by audit, or reclassifying when an entry's nature is reassessed (e.g. a `lesson` outgrew its incident and became a `rule`; a `user`-scope entry turned out to be project-specific).

Arguments:
- `entry_path` — path to the existing entry file
- `new_type` (optional) — target category (one of the 6 identifiers)
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

## Overview (`action=overview`)

Generates a **synthesized project overview** by reading all `architecture` and `topology` entries in the current scope (plus optionally critical-severity lessons / rules), and writing a digested document to `<scope>/.claude/iris-gotcha/overview.md`.

The overview is a **derived view**, not a canonical store — entries remain the source of truth. Editing `overview.md` directly is a mistake because the next regeneration overwrites changes. To change content, edit the source entries and regenerate.

### Why this exists

The index lists titles + keywords (good for at-a-glance recall) but doesn't carry the *prose* of architecture/topology entries. For onboarding a new AI session — or a new human collaborator — into the project, you want one readable document, not 8 file paths to chase. `overview.md` synthesizes those into a single project orientation.

### When to invoke

- User says "summarize the project" / "generate project overview" / "give me an architecture snapshot"
- New session lands in a project and the user asks "what is this codebase about" (run overview to populate yourself, then answer)
- After several new architecture/topology entries were added in this or recent sessions (the existing overview is now stale)
- During audit, if the user requests it

### Arguments

- `scope` (optional, defaults to `project` if `<pwd>/.claude/iris-gotcha/` has any entries, else `user`)
- `include_critical_rules` (optional, default true): also pull `severity: critical` or `zero-tolerance` entries from `lesson` / `rule` so the overview surfaces "things that would surprise you"

### Procedure

1. Determine scope.
2. List entries:
   - `<scope>/architecture/*.md`
   - `<scope>/topology/*.md`
   - (if `include_critical_rules`) `<scope>/lesson/*.md` and `<scope>/rule/*.md` with severity ≥ `critical`
3. Read each. Note: `last_modified` for the source-entries footer; `title`, `keywords`, and `body` for content.
4. **Synthesize** (this is real work, not concatenation). Organize entries into a coherent narrative under fixed section headings — group related architecture entries (e.g. "auth design" + "data flow" → one paragraph), keep topology entries closer to fact-listing format. Resolve overlaps between entries gracefully.
5. Write to `<scope>/.claude/iris-gotcha/overview.md`.
6. Run wiring routine: ensure the project's CLAUDE.md (or `<scope>/.claude/CLAUDE.md`) imports `@./.claude/iris-gotcha/overview.md` in addition to the index. Idempotent — grep before append.
7. Report: number of entries consumed, write path, whether the CLAUDE.md wiring changed.

### Output format

`overview.md` is structured for both AI parsing (predictable sections) and human reading (prose where appropriate). Layout:

```markdown
---
generated: 2026-05-15T14:23:00Z
source_entries: 5
scope: project
project_root: /Users/daniel/Workspace/iris-island
oldest_source: 2026-05-01T10:00:00Z
newest_source: 2026-05-15T12:00:00Z
---

# <project-name> — Architecture Overview

> Auto-generated by iris-gotcha from architecture/topology entries (+ critical lessons/rules).
> Source of truth: `<scope>/.claude/iris-gotcha/architecture/` and `topology/`.
> Edit those, then `action=overview` to regenerate. **Do not edit this file directly — changes will be overwritten.**

## Architecture

[Synthesized prose. Organize by topic, not by source filename. Make it read as one coherent description of the system, not a list of disconnected entries. ~200-500 words depending on entry count.]

## Topology

[Synthesized but more fact-oriented. Lists or short paragraphs are fine. Services, ports, paths, env vars, package layout. Keep it scannable.]

## Critical conventions

[Only if include_critical_rules and there are matching entries. Brief — "These behaviors would surprise you if you didn't know them in advance: <list>". Each item references the source entry path.]

## Source entries

- `architecture/2026-05-15-iris-sidecar.md` (modified 2026-05-15)
- `architecture/2026-05-15-auth-design.md` (modified 2026-05-15)
- `topology/2026-05-15-services-and-ports.md` (modified 2026-05-15)
- `lesson/2026-05-15-bun-macos-global-install.md` (modified 2026-05-15) [critical]
```

### Staleness handling

When invoked, before writing, check the existing `overview.md`'s frontmatter `newest_source` against the actual newest entry's `last_modified`. If they match (no entries changed since last generation), report "overview is current; nothing to regenerate" and stop. This avoids needless rewriting and keeps git diffs clean.

### Why not auto-regenerate

Two reasons:
1. **Trigger ambiguity**: auto-regen on every capture is too eager (overview thrashes on a flurry of unrelated entries); auto-regen on a schedule is too coarse. The user knows when it's worth a regen.
2. **Cost**: synthesis is real LLM work, not just file concatenation. Doing it on demand keeps the cost predictable.

The user (or AI noticing stale wording on session start) explicitly requests a regen.

## Recall (`action=recall`)

The index is already loaded in your context via `@-import`. Scan it for keyword/title matches and Read the relevant file directly — that's recall. You only need to invoke `action=recall` explicitly when the user wants a broader multi-keyword search across both scopes or when the question is too vague to know what to Read.

## Audit (`action=audit`)

1. List entries from `~/.claude/iris-gotcha/` (and the project scope if requested).
2. Read each. Check:
   - `disambiguation` is non-trivial — a vacuous reason like "why not rule: it's not a rule" hints at miscategorization.
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
