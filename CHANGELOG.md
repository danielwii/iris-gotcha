# Changelog

Each entry explains *what changed* and *why* — design rationale, not just bullet-point release notes. Reverse chronological.

## v0.12.2 — the keyword budget belongs to the rendered line, not to the entry

Step 8 contradicted itself. Step 1 read `keywords` **first 5 only** — a render-time truncation. Step 4 then validated `Each entry's keyword count ≤ 5` as a hard budget, i.e. a write failure. Only one of the two can be operative: if the budget refuses, truncation is unreachable dead text; if truncation happens, the budget can never fail. On the user-scope notebook this was not academic — **26 of 29 entries stored more than 5 keywords** (the largest had 22), so under the refusing reading no index could ever be written until keywords were deleted from most of the notebook.

Measurement settled it. Rendering every stored keyword puts **12 of 30 lines over the 220-code-point line budget** (longest 532) — an index that is structurally unwritable. Rendering the first 5 puts **0 of 30** over. Truncation is not an alternative to the line budget; it is the precondition for the line budget being satisfiable. And because both readings produce a **byte-identical index** (4.3k tokens either way), enforcing the stored count as a hard budget would have meant irreversibly deleting real search terms from 26 entries to obtain exactly the index truncation already produces.

So the keyword budget now reads as what it always had to be: a bound on the **rendered** line, guaranteed by step 1's truncation and therefore unable to fail. Entries may store as many keywords as retrieval warrants, ordered most-distinctive-first since position decides what reaches the index. Trimming stored lists remains available as an `action=audit` soft-budget suggestion — never a regeneration refusal.

This is the fourth defect of this shape found in v0.12.0's budget machinery, after the missing presence check, the undefined unit, and the undocumented whole-file semantics (all v0.12.1). The common root: budgets were written as a table of numbers without stating **which artifact each number governs** — the stored entry, the rendered line, or the finished file.

Spec-only change. No schema change, no migration.

## v0.12.1 — enforce the mandatory field; a declaration is not a guardrail

v0.12.0 declared `index_cue` "mandatory on every entry" and then never checked for it. Step 8's validation table held four budget rows and no presence row; `action=audit`'s Applicability and Atomicity checks both *read* `index_cue`, silently presupposing it exists. The gap is invisible by construction, because **a missing field trivially satisfies every budget** — there is no text to measure, so a budget-only pass reports "no violations" on a notebook where no entry has the field at all.

Found by running `action=audit` against the user-scope notebook on 2026-09-05: **28 of 30 entries had no `index_cue`**, two had no conforming frontmatter at all (one had no frontmatter block whatsoever — its metadata was a bullet list in the body), and the audit, executed faithfully, flagged none of it. The 28 were only visible because the operator ran a schema check the spec does not specify. This is the same shape as the failure v0.12.0's own release note describes and as the notebook's `lesson/2026-09-03-registered-but-not-in-the-pipeline.md`: **registered is not the same as in the pipeline.** Declaring a field mandatory in a spec section does nothing; only a step that refuses on its absence does.

**Step 8 now validates schema presence before budgets**, and refuses to write on a missing frontmatter block, `index_cue`, `type`, or `keywords`. Explicitly forbidden: synthesizing a missing cue by summarizing the entry body. Step 8 does not read bodies by design, and improvising there would reintroduce precisely the unbudgeted, unreviewed prose `index_cue` exists to keep out — invisibly, since the output would look well-formed. **`action=audit` runs schema completeness first** and must report Applicability/Atomicity as `blocked` rather than passing on entries lacking the field, and must state how many entries each check actually ran against — a count of flags raised says nothing about how many entries were examined.

**Budget units disambiguated.** The whole-file row said "characters" while every other row said "code points", and nothing in the spec defined the term. For the notebook that exposed this, the two readings disagree on the verdict: 26,302 code points (under the 32,000 hard budget) versus 34,363 UTF-8 bytes (over it). All budgets are now stated in Unicode code points. The whole-file budget is also documented for what it actually is — an **entry-count backstop**, not a verbosity backstop, since the per-line budget binds first (at 220 code points/line the file cannot reach 32,000 until ~145 entries). An over-budget file whose every line is within budget means too many entries; the fix is audit and deletion, not further compression.

Spec-only change. No schema change, no new field, no migration.

## v0.12.0 — `index_cue`: bound the always-on index by construction, not by discipline

The index is the plugin's entire always-on cost, and until now nothing bounded it. `SKILL.md` said "keep each line short" — an adjective, not a criterion — so the AI writing a capture had its attention on the entry, and the index line absorbed whatever prose came out. A real project-scope index (`unee/.claude/iris-gotcha/index.md`) reached **57.8k characters**, past the 40k injection warning threshold. Two mechanisms drove it: the `Recently strengthened` section duplicated nearly the whole index instead of holding a rolling 7-day window, and individual entries absorbed multiple unrelated scenario→constraint mappings across repeated strengthening (one `lesson` mixed "paired web+scheduler deployments must release together" with "ArgoCD kustomize image keys must be full URIs" — two different rules, only one conditional on the project's CD tooling).

**`index_cue` frontmatter field** (mandatory on every entry) is now the *only* natural language regeneration may copy into `index.md`; everything else on an index line — path, keywords, severity — is copied mechanically from other fields, and `title` no longer appears in the index at all. Prescriptive cues must read as *scenario + constraint* in one sentence; descriptive cues state the shortest accurate current-state fact. Budgets are explicit rather than adjectival: cue 60/100 code points, keywords 4/5, each keyword 24/40, rendered line 160/220, whole file 24,000/32,000 characters (soft/hard).

**Step 8 regeneration becomes a pure function of entry frontmatter.** It never reads or carries text forward from the existing `index.md` — which matters because re-reading the old index as a starting point is exactly how bloat reproduces itself. A hard-budget violation is a **write failure**, not a truncation trigger: the offending entries are reported and the old index is left untouched. This is the load-bearing change — the constraint now sits in the regeneration step's accept/reject decision instead of in a reminder the capture step is free to dilute.

**Two new capture gates.** Step 0.5 (scenario-behavior atomicity): can this be stated as one "if scenario then constraint" sentence, or does it need splitting? Step 0.6 (world-state vs behavioral-constraint): is this a fact about the project's current state — which belongs in `architecture`/`topology` — or a conditional constraint, which belongs in a prescriptive type with an explicit scenario clause? Baking world-state into a prescription as a permanent premise goes silently wrong the moment the project's mechanism changes, and nothing catches the staleness. Step 5's strengthening gate now compares scenario+constraint explicitly instead of matching on topic; Step 6.5 self-checks the cue against budget; `action=audit` gains applicability, atomicity, and recent-region eligibility checks.

No schema-breaking change beyond the new mandatory field; six categories stay six.

> **Release note (2026-09-05).** This version was built and installed on 2026-07-15 but never committed, and was recovered from an orphaned `~/.claude/plugins/cache/` copy seven weeks later. The work was done inside `~/.claude/plugins/cache/../marketplaces/iris-gotcha` — a directory that has a `.git` and the correct remote, and so reads as the source repo, but is a Claude Code-managed install artifact that `/plugin marketplace remove/add` re-clones over. The plan for this version warned "do not edit the cache copy"; it did not know the marketplaces copy is equally disposable. Meanwhile `installed_plugins.json` still pointed at 0.11.0, so every subsequent launch loaded 0.11.0 and marked 0.12.0 orphaned — silently, with no error at any layer. **Edit the repo under version control, commit before installing, and treat anything under `~/.claude/plugins/` as disposable regardless of whether it looks like a checkout.**

## v0.11.0 — three new triggers (T1 / T2 / T3) for the doctrine-doesn't-reach-AI gap

Addresses the "AI doesn't know my architecture/spec" gap with three new procedures.

**T1 end-of-planning retrospective (capture-side):** when a planning arc concludes (`/make-plan` done, verbal "好开始" cue, or 3+ turn architectural discussion converging), AI scans the artifact, runs the 3-probe split test, pre-classifies each candidate as `permanent` (architectural commitment → capture) or `tactical` (this-implementation-only → skip), surfaces a structured proposal for per-candidate user confirmation. Mirror-symmetric to v0.8.0's end-of-debug retrospective.

**T2 correction-with-novel-reason (capture-side):** when user corrects AI with a stated reason absent from the index, treat it as a missing-rule signal — capture as new entry (vs. existing Step 5 which strengthens already-matched entries). Permanence gate must be explicitly asked (wording accommodates descriptive Y as well as prescriptive Y); novel-reason corrections often span all three probes (intent + fact + prescription) so the 3-probe split test is called out explicitly in the procedure.

**T3 pre-implementation rule check (recall-side, first automatic-trigger structured procedure under `action=recall`):** before writing non-trivial code, AI scans the index by keyword, tiered-Reads relevant entry bodies (`critical` / `zero-tolerance` always; lower tiers cap at index line), and announces the applicable rules. Token cost controlled via tiered Read + session-scoped per-task cache + trivial-task skip.

CLAUDE.md gains 3 narrow invariants (one per trigger). Common failure modes expanded with 2 new entries ("Skipping the permanence gate" and "T3 scan as ceremony, not attention"). No schema change, no breaking change.

Failure mode (a) — session-start project scan — deferred: the design bets T1/T2/T3 partially mitigate by putting captured rules in the always-on layer plus T3 forcing attention before implementation; revisit when pull is clear.

## v0.10.0 — doc-follows-code drift discipline

Adds a new top-level `## Doc Follows Code` section to SKILL.md requiring every descriptive entry (`architecture` / `topology`) to declare a `references: [path1, path2, ...]` frontmatter field listing the code paths the entry depends on, and extends `action=audit` to compare each referenced file's mtime against the entry's `last_modified` — flagging stale entries as "potentially drifted" rather than letting them silently misinform sessions.

Frontmatter schema gains `references:` (for descriptive types) and optional `unverified: true` (fallback when referenced code is inaccessible).

Motivation: the `@-import` mechanism that makes good architecture/topology entries valuable also makes stale ones actively harmful — the AI cites them confidently in downstream reasoning. `references:` is the smallest structural lever that converts drift from invisible decay to audit-detectable signal.

No procedure changes for prescriptive types; signal-density improvement = converting one previously-invisible failure mode (stale descriptive entries) into one auditable one.

## v0.9.1 — adds "AI confabulation / reasoning failures" as a capture class

Driven by a real session where an AI confidently derived an "by design" rationale from a filename + grep without log verification, and required user challenge to surface the error. Pattern: AI has context, extrapolates incorrectly with confident tone; silent record-and-recall is the only counter.

Capture as `lesson` with `## Related` linking any abstract discipline rule (e.g. `.claude/rules/diagnosis-discipline.md`) — incident-evidence makes the abstract rule embodied.

No procedure changes; doctrine addition only.

## v0.9.0 — purpose reframe to signal-to-noise

iris-gotcha's H1 changes from "Training-Gap Knowledge Notebook" to "Signal-to-Noise Optimizer for AI Context".

Adds an explicit `## Why this exists` section establishing the **two-gate capture model**:

- **Gate 1: training-gap** — would a fresh AI handle this correctly anyway?
- **Gate 2: signal value** — will future sessions actually reference this?

Plus a mechanism-by-mechanism mapping of how every existing piece of doctrine serves signal density.

The signal-to-noise criterion is installed as the **evaluation standard for future changes**: any new feature, doctrine change, or capture entry must articulate how it improves the behavior-changing-knowledge / context-token ratio.

No procedure changes — purpose and evaluation framing only. (Training-gap stays as Step 0 of capture, now framed as the first-stage signal filter.)

## v0.8.0 — judgment + ask (shift from rigid defaults)

Surfaced by ein's v0.7.0 capture run and a dogfood capture:

- **Probe answers are 3-state, not 2-state.** Previously "can't articulate" meant "no, skip entry." Now: `yes` / `confirmed-no` / `unknown → ask the user`. Intent and prescription probes especially depend on user-only knowledge; defaulting to "no" silently loses that information. One short clarifying question beats untangling missing entries later.
- **Project root is content-led, not pwd-led.** Step 2 of capture now uses judgment: if the content names a specific project that exists as a real directory, that's the project root — regardless of where `pwd` happens to be. The earlier rigid "project = pwd" rule misfiled the dogfood capture (running from `~/Workspace` while capturing about `~/Workspace/iris-gotcha`).
- **End-of-debug retrospective trigger added.** When a multi-turn debugging arc concludes (resolved or abandoned), the AI must ask itself: "was this root cause something a fresh Claude wouldn't have known a priori?" If yes, propose a gotcha capture. Stronger than the existing retry-count trigger (which fires during, not at end).
- **Triage announcement at user-triggered captures.** When the user says "记一下" / "remember this", the AI's first action is now to scan the index and announce findings (0 candidates → new; 1 candidate identical → strengthen; 1 candidate related → cross-ref; contradictory → stop and ask). Makes the new-vs-update-vs-reference decision visible upfront, instead of buried in a Step 9 report.

## v0.7.0 — enforcement-tightening pass

Driven by ein's first v0.6.0 capture run. ein's analysis was mostly correct but missed three things at the SKILL-protocol level (not its fault — the SKILL was informally complete but formally incomplete):

- **Split test generalized to a three-probe form** (intent / fact / prescription). The old "lesson vs descriptive" test only covered 2-way splits; real concepts often have all three aspects (e.g. the unee shared-app case = `topology` + `architecture` + `lesson`). `definitions.md` now lists the three probes and a worked three-way example.
- **Severity ladder gets concrete qualifiers.** Old guidance described consequence classes abstractly; v0.7.0 names what counts as each level. Notably: trust-boundary violations (accepting untrusted auth source as trusted, bypassing access checks) always start at `critical`, even without a prior incident. Anchoring rule: "when unsure, pick higher."
- **`## Related` body section is now spec'd** with canonical format (`- See <path> — <aspect>`). Mandatory whenever Step 3 produced a split, because otherwise the cross-reference discipline is incomplete and the future-Claude can't recover the full picture from any starting point.
- **Step 9 must report the Step 5 scan outcome** — even when nothing matched. Previously, the Step 5 strengthening-check was internal and invisible; now it's a mandatory line in the user-facing report, making it auditable.

## v0.6.0 — first-principle restructure (drops `experience`, narrows `rule`)

Based on real-world usage data (ein's transcript):

- **New Step 0: training-gap gate.** Before doing anything, the skill asks "would the AI handle this without the notebook?" — if yes, skip capture. Reframes the skill's purpose: iris-gotcha is a supplement to training, not a re-statement of what the AI already does. Many "rule"-like things (no secrets in logs, sanitize input, etc.) are now correctly rejected at this gate.
- **`experience` category dropped.** Pure narrative without a behavioral takeaway accumulated as dead weight in the index. `claude-mem` already records session history; iris-gotcha now only captures knowledge that changes future behavior or orients understanding. **Six categories instead of seven.**
- **`rule` narrowed.** Now restricted to project-specific MUSTs the AI wouldn't know, plus user overrides of AI default behavior. Generic safety/security MUSTs the AI already enforces are explicitly out of scope.
- **New `action=overview`.** Synthesizes architecture + topology entries (plus critical lessons/rules) into a single readable project overview document at `<scope>/.claude/iris-gotcha/overview.md`. Designed for both AI session orientation (auto-wired into project CLAUDE.md) and human onboarding (committable to git). On-demand, not auto-regenerate. The overview is a derived view; entries remain canonical.
- **Active split test added** to `definitions.md`: before finalizing a category, the skill asks "does this entry contain a descriptive fact that would survive even if the lesson part were forgotten?" — to catch hybrid entries that should be split into topology + lesson with cross-references.

## v0.5.0 — fills four small gaps surfaced by subagent pressure tests

- Initial severity is chosen by violation consequence class (not always `medium`); a small table guides starting points.
- Frontmatter spec clarifies the `violation_count: 0` + omit `last_violated` shape for preemptive captures (vs the post-incident shape shown in the example).
- Step 9 and a new "Handling user disagreement" section direct the AI to surface classification disputes and offer `action=move` rather than silently re-classifying.
- The severity ladder gets a "When language stops working" note: at `zero-tolerance` with continued violations, the right next step is a structural fix (lint rule, pre-commit hook, automation) outside iris-gotcha.

## v0.4.1 — SKILL.md frontmatter `description` doctrine fix

Rewrites the SKILL.md frontmatter `description` per the writing-skills CSO doctrine: pure "Use when..." trigger conditions with no workflow summary. Prior versions began the description with "Capture, classify, recall, audit, move, and push..." which (per writing-skills testing) can cause Claude to act on the description's process summary rather than read the full SKILL body.

## v0.4.0 — `action=move` + SKILL.md tightening

Adds `action=move` for reclassifying entries between categories or scopes.

Tightens SKILL.md (removed redundant Bootstrapping / Project-scope sections; merged the inline severity-list duplicate; compressed the Recall section to one paragraph).

Softens tone (explains *why* instead of leaning on `MUST` / `NEVER` where reasoning is more reliable than imperatives).

## v0.3.0 — automatic CLAUDE.md `@-import` wiring

Automatic, idempotent CLAUDE.md `@-import` wiring on every capture. The skill now ensures the relevant CLAUDE.md (user-scope: `~/.claude/CLAUDE.md`; project-scope: `<pwd>/CLAUDE.md` or `<pwd>/.claude/CLAUDE.md`) imports the right index file, creating the project-level CLAUDE.md if absent.

No manual setup needed after install.

## v0.2.0 — English category identifiers (breaking)

Switched to English category identifiers (`type:` field), with Chinese kept as glosses.

**Breaking change** for anyone who installed `0.1.0`: re-classify entries by moving from `教训/` etc. to `lesson/` etc. and update each frontmatter `type:` field to the English identifier.

## v0.1.0 — initial release

Initial release. Chinese category names in directories and frontmatter.
