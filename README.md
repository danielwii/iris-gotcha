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

1. **Seven mutually-exclusive categories**, each with strict definitions and a mandatory `disambiguation` field on every entry.
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

To version-control your knowledge, `git init` inside `~/.claude/iris-gotcha/`. The skill provides an `action=push` that commits and pushes. Cross-machine sync is out of scope — manage that with git or any sync mechanism you prefer.

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
- `0.10.0` — **doc-follows-code drift discipline** for `architecture` / `topology` entries. Adds a new top-level `## Doc Follows Code` section to SKILL.md requiring every descriptive entry to declare a `references: [path1, path2, ...]` frontmatter field listing the code paths the entry depends on, and extends `action=audit` to compare each referenced file's mtime against the entry's `last_modified` — flagging stale entries as "potentially drifted" rather than letting them silently misinform sessions. Frontmatter schema gains `references:` (for descriptive types) and optional `unverified: true` (fallback when referenced code is inaccessible). The @-import mechanism that makes good architecture/topology entries valuable also makes stale ones actively harmful — the AI cites them confidently in downstream reasoning; `references:` is the smallest structural lever that converts drift from invisible decay to audit-detectable signal. No procedure changes for prescriptive types; signal-density improvement = converting one previously-invisible failure mode (stale descriptive entries) into one auditable one.
- `0.9.1` — adds **"AI confabulation / reasoning failures"** as an explicit high-signal capture class in SKILL.md's `What goes in` list. Driven by a real session where an AI confidently derived an "by design" rationale from a filename + grep without log verification, and required user challenge to surface the error. Pattern is: AI has context, extrapolates incorrectly with confident tone; silent record-and-recall is the only counter. Capture as `lesson` with `## Related` linking any abstract discipline rule (e.g. `.claude/rules/diagnosis-discipline.md`) — incident-evidence makes the abstract rule embodied. No procedure changes; doctrine addition only.
- `0.9.0` — **purpose reframe to signal-to-noise**. iris-gotcha's H1 changes from "Training-Gap Knowledge Notebook" to "Signal-to-Noise Optimizer for AI Context". Adds an explicit `## Why this exists` section establishing the **two-gate capture model** (gate 1: training-gap; gate 2: signal value — will future sessions actually reference this?) and a mechanism-by-mechanism mapping of how every existing piece of doctrine serves signal density. The signal-to-noise criterion is also installed as the **evaluation standard for future changes**: any new feature, doctrine change, or capture entry must articulate how it improves the behavior-changing-knowledge / context-token ratio. No procedure changes in v0.9.0 — purpose and evaluation framing only. (Training-gap stays as Step 0 of capture, now framed as the first-stage signal filter.)
- `0.8.0` — shift from rigid defaults to judgment + ask. Surfaced by ein's v0.7.0 capture run and a dogfood capture:
  - **Probe answers are 3-state, not 2-state.** Previously, "can't articulate" meant "no, skip entry." Now: `yes` / `confirmed-no` / `unknown → ask the user`. Intent and prescription probes especially depend on user-only knowledge; defaulting to "no" silently loses that information. One short clarifying question beats untangling missing entries later.
  - **Project root is content-led, not pwd-led.** Step 2 of capture now uses judgment: if the content names a specific project that exists as a real directory, that's the project root — regardless of where `pwd` happens to be. The earlier rigid "project = pwd" rule misfiled the dogfood capture (running from `~/Workspace` while capturing about `~/Workspace/iris-gotcha`).
  - **End-of-debug retrospective trigger added.** When a multi-turn debugging arc concludes (resolved or abandoned), the AI must ask itself: "was this root cause something a fresh Claude wouldn't have known a priori?" If yes, propose a gotcha capture to the user. Stronger than the existing retry-count trigger (which fires during, not at end).
  - **Triage announcement at user-triggered captures.** When the user says "记一下" / "remember this", the AI's first action is now to scan the index and announce findings (0 candidates → new; 1 candidate identical → strengthen; 1 candidate related → cross-ref; contradictory → stop and ask). Makes the new-vs-update-vs-reference decision visible upfront, instead of buried in a Step 9 report.
- `0.7.0` — enforcement-tightening pass driven by ein's first v0.6.0 capture run. ein's analysis was mostly correct but missed three things at the SKILL-protocol level (not its fault — the SKILL was informally complete but formally incomplete):
  - **Split test generalized to a three-probe form** (intent / fact / prescription). The old "lesson vs descriptive" test only covered 2-way splits; real concepts often have all three aspects (e.g. the unee shared-app case = `topology` + `architecture` + `lesson`). definitions.md now lists the three probes and a worked three-way example.
  - **Severity ladder gets concrete qualifiers**. Old guidance described consequence classes abstractly; v0.7.0 names what counts as each level. Notably: trust-boundary violations (accepting untrusted auth source as trusted, bypassing access checks) always start at `critical`, even without a prior incident. Anchoring rule: "when unsure, pick higher."
  - **`## Related` body section is now spec'd** with canonical format (`- See <path> — <aspect>`). Mandatory whenever Step 3 produced a split, because otherwise the cross-reference discipline is incomplete and the future-Claude can't recover the full picture from any starting point.
  - **Step 9 must report the Step 5 scan outcome** — even when nothing matched. Previously, the Step 5 strengthening-check was internal and invisible; now it's a mandatory line in the user-facing report, making it auditable.
- `0.6.0` — first-principle restructure based on real-world usage data (ein's transcript):
  - **New Step 0: training-gap gate.** Before doing anything, the skill asks "would the AI handle this without the notebook?" — if yes, skip capture. Reframes the skill's purpose: iris-gotcha is a supplement to training, not a re-statement of what the AI already does. Many "rule"-like things (no secrets in logs, sanitize input, etc.) are now correctly rejected at this gate.
  - **`experience` category dropped.** Pure narrative without a behavioral takeaway accumulated as dead weight in the index. `claude-mem` already records session history; iris-gotcha now only captures knowledge that changes future behavior or orients understanding. Six categories instead of seven.
  - **`rule` narrowed.** Now restricted to project-specific MUSTs the AI wouldn't know, plus user overrides of AI default behavior. Generic safety/security MUSTs the AI already enforces are explicitly out of scope.
  - **New `action=overview`.** Synthesizes architecture + topology entries (plus critical lessons/rules) into a single readable project overview document at `<scope>/.claude/iris-gotcha/overview.md`. Designed for both AI session orientation (auto-wired into project CLAUDE.md) and human onboarding (committable to git). On-demand, not auto-regenerate. The overview is a derived view; entries remain canonical.
  - **Active split test added** to definitions.md: before finalizing a category, the skill asks "does this entry contain a descriptive fact that would survive even if the lesson part were forgotten?" — to catch hybrid entries that should be split into topology + lesson with cross-references.
- `0.5.0` — fills four small gaps surfaced by subagent pressure tests:
  - Initial severity is chosen by violation consequence class (not always `medium`); a small table guides starting points.
  - Frontmatter spec clarifies the `violation_count: 0` + omit `last_violated` shape for preemptive captures (vs the post-incident shape shown in the example).
  - Step 9 and a new "Handling user disagreement" section direct the AI to surface classification disputes and offer `action=move` rather than silently re-classifying.
  - The severity ladder gets a "When language stops working" note: at `zero-tolerance` with continued violations, the right next step is a structural fix (lint rule, pre-commit hook, automation) outside iris-gotcha.

## License

MIT
