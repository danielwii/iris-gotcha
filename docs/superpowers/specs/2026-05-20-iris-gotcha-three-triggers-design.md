# iris-gotcha v0.11.0 — Three New Triggers (T1 / T2 / T3) Design

**Status**: Approved (brainstorming concluded 2026-05-20)
**Target version**: 0.11.0 (minor bump from 0.10.0)
**Scope**: SKILL.md additions only; no schema changes, no breaking changes to existing entries.

## Problem statement

User identified a gap: when an architecture / spec / plan is produced (whether via `/make-plan`, conversational design, long-term `SPEC.md`, or `CLAUDE.md` sections), subsequent AI sessions may not know it and develop in non-conforming ways. Three failure-mode shapes were named:

- **(a)** Spec exists in project files; AI doesn't proactively read it.
- **(b)** Spec exists only in past-session conversation; new session has no path to recover it.
- **(c)** Spec exists in a plan file; AI reads it but still drifts in implementation.

The user's stated intent ("自动识别并形成 rules") points at the **capture side** — write more decisions into `iris-gotcha`'s always-on layer — plus an explicit **pre-implementation recall step** to address (c)'s drift-despite-reading aspect.

Three triggers are designed to cover these failure modes:

| Trigger | Side | Failure mode addressed |
|---|---|---|
| **T1 — End-of-planning retrospective** | Capture | (b), part of (c) |
| **T2 — Correction-with-novel-reason** | Capture | (b) at correction-moment |
| **T3 — Pre-implementation rule check** | Recall | (c) — attention-as-mechanism |

Failure mode (a) is acknowledged but the dedicated session-start scan trigger ("T0") was **not** selected by user. The design bets that T1/T2/T3 partially mitigate (a) — captured rules enter the always-on layer (visible without scanning), and T3 forces attention before implementation. A dedicated T0 is deferred until pull is clear.

## Design principle (load-bearing)

**Capture writes; Recall reads.** Both serve `signal-to-noise`. Existing iris-gotcha already has both axes (capture procedure + `## Recall` section), but recall has been informal ("scan the @-imported index, Read what's relevant"). T3 introduces the first **automatic-trigger structured procedure** on the recall axis. Capture vs. recall are not promoted to top-level conceptual sections — that would be premature abstraction (only one recall trigger exists). Surgical placement preserves existing reader hierarchy.

## SKILL.md changes (surgical — no restructure)

```
## When to invoke                    → expand table with 3 new rows
## Capture procedure                 → unchanged (Step 0-9)
  ### End-of-debug retrospective    → unchanged
  ### Triage announcement            → unchanged
  ### End-of-planning retrospective → NEW (T1, after end-of-debug)
  ### Correction-with-novel-reason  → NEW (T2, after T1)
## Handling user disagreement         → unchanged
## Doc Follows Code                  → unchanged (v0.10.0)
## Move / Overview                    → unchanged
## Recall (`action=recall`)           → expand: add Pre-implementation
                                       rule check (T3) as a subsection
## Audit / Push                       → unchanged
## Common failure modes               → expand with 1–2 entries covering
                                       new trigger-specific failure modes
```

No files renamed, no sections moved.

## T1 — End-of-planning retrospective

### Mirror analogy

T1 is structurally symmetric to the existing **end-of-debug retrospective** (v0.8.0). Debug arc → "was the root cause non-obvious?"; planning arc → "what architectural decisions need future-session attention?". Both are autonomous AI-detected triggers proposing capture, awaiting user confirmation.

### Trigger signals (any one fires T1)

- `/make-plan` (or equivalent planning skill) completes.
- A plan file appears in conventional plan locations (`docs/plans/` / `.claude/plans/` / `plans/` / similar). These are heuristic signals — if the user uses a non-standard plan-storage path, the verbal cue or AI-judgment signals still apply.
- User verbal cue: "好开始", "let's implement", "go ahead", "可以了" (any explicit "transition from design to implementation" signal).
- AI judgment: 3+ turns of architectural discussion converging with no remaining design questions.

### Skip conditions

- Planning arc was trivial (1–2 turns, no architectural decisions).
- All candidate decisions are training-redundant (Step 0 gate would reject all).

### Procedure

Inserted as a precondition to the standard capture procedure (runs before Step 0):

1. **Scan the planning artifact** — plan file, conversation history, or both. Identify candidate decisions that would change AI behavior in implementation.
2. **Run the 3-probe split test on each candidate** (per `definitions.md`): intent / fact / prescription. Multi-probe yes-answers produce multiple cross-referenced entries via `## Related` (per v0.7.0+ doctrine).
3. **Apply the Step 0 training-gap filter to each candidate** — drop ones that a fresh Claude would handle correctly anyway (this is Capture procedure's Step 0 logic applied here as a pre-filter; candidates that survive proceed to permanence pre-classify and proposal).
4. **Permanence pre-classify (T1-specific gate)** — for each surviving candidate, AI labels as `permanent` (architectural commitment) or `tactical` (this-implementation-only). Tactical candidates are recommended `skip` (use `claude-mem` for session history if needed).
5. **Structured proposal** — present candidates to user. See example below.
6. **User confirms per-candidate** — full approve / partial approve / skip / edit. User can override AI's permanence pre-classification.
7. **Each confirmed candidate runs full Capture Step 1–9**, including Step 5 strengthening check (avoid duplicating an existing entry).

### Permanence gate — the load-bearing innovation

Plans mix two kinds of content:

| Class | Example | Action |
|---|---|---|
| **Permanent** (architectural commitment) | "Auth services must be split from business services." | Capture |
| **Tactical** (this-implementation-only) | "During the rollout, dual-write to old + new tables." | Skip (claude-mem if needed) |

Without this discriminator, T1 would pollute the always-on layer with time-bounded tactical decisions that lose relevance fast and pay context tax. The gate is mandatory; AI must pre-classify and the proposal must surface the classification for user verification.

### Proposal output format (what user sees)

```
End-of-planning retrospective triggered
  Signal: /make-plan completed → docs/plans/2026-05-20-auth.md

Scanning plan for capture candidates... 5 candidates found.

[1] PERMANENT — architecture
    Title: "Auth and business services are split; business validates JWT only"
    Disambiguation: why not topology — design intent, not port/path facts
    Source: plan §2.1
    Related: [2]

[2] PERMANENT — topology
    Title: "auth-service on :8080; business services validate via JWT"
    Source: plan §2.1
    Related: [1]

[3] PERMANENT — rule (critical, trust-boundary)
    Title: "All API endpoints MUST go through auth-gateway"
    Source: plan §2.3

[4] TACTICAL → propose skip
    Title: "Dual-write to old + new auth tables during rollout"
    Reason: explicitly time-bounded ("for the migration period")
    Recommendation: skip (or capture in claude-mem if needed)

[5] PERMANENT — best-practice (borderline)
    Title: "TLS 1.3 for all internal comms"
    ⚠️ Step 0 borderline — industry default may already be enforced by training.
    Recommendation: you decide.

Reply: "all" / "1 2 3" / "1 2 3 5" / "none" / per-candidate edits
```

### Disallowed rationalizations

- "Plan is too detailed, skip capture" — long plans need distilling, not skipping; the value is the high-signal subset.
- "User will capture manually if they want" — the entire point of T1 is removing the manual remember-to-capture cost.
- "Every plan decision is permanent" — false; many are tactical. Permanence gate is mandatory.
- "Capture everything; audit later" — audit fires only on explicit request; bad captures pollute the always-on layer until cleaned.

## T2 — Correction-with-novel-reason

### Relationship to existing Step 5 strengthening (key disambiguation)

| | Existing Step 5 | T2 |
|---|---|---|
| Trigger | About to repeat an entry-recorded mistake | User corrects with a stated reason not in the index |
| Outcome | Strengthen existing entry (severity ↑) | Capture new entry |
| Detection | AI's self-monitoring | User-input pattern matching |

These are complementary, not redundant.

### Trigger pattern

User statement matches one of:

- `[do/don't] X, because Y`
- `this violates [Y]` / `这违反了 Y` / `we [must/should] [do/avoid] X`

**AND** AI's current output/intent diverges from Y (otherwise it's an informational statement, not a correction).

### Skip conditions

- User correction without a reason ("不对", "改一下") — nothing to capture.
- AI's output already conforms to user's statement — informational, not a correction.
- Reason Y is already in the index — route to Step 5 strengthening instead.
- Reason Y is training-redundant (Step 0).
- User just invoked an explicit capture flow ("记一下") — avoid double-triggering.

### Procedure

1. **Extract reason Y** — verbatim quote preferred (avoid AI's paraphrase introducing drift).
2. **Search index** (user + project scope) by keyword. Index is already in context via `@-import` — keyword scan first, body Read only on title-ambiguous candidates.
3. **If matched** — route to Step 5 strengthening. Announce: "This is already in index at `<path>`; will strengthen instead of new entry." Do NOT propose new.
4. **If novel** — proceed to permanence gate.
5. **Permanence gate (required ASK)** — T2 has less context than T1 (no plan to read), so AI MUST explicitly ask: "Is Y a permanent rule, or just for this case?" Cannot infer from tone — users often state permanent rules conversationally.
6. **If permanent** — run full Capture Step 0–9. Disambiguation field MUST cite source: `"via user correction on YYYY-MM-DD"`.
7. **If just-for-this-case** — apply Y for current turn only. Tell user: "Applying Y here. Not capturing — if it recurs, we'll capture next time."

### Disallowed rationalizations

- "User is correcting me, they'll capture if they want" — silent application loses the rule for next session; T2 catches at moment of greatest specificity.
- "It's just one correction, not worth capturing" — by definition, a novel-reason correction points at an AI-unknown rule. That IS the gap iris-gotcha exists for.
- "Wait for 记一下" — by the time user remembers to invoke, the conversation has moved on and context is degraded.
- "Permanent vs. tactical can be inferred from tone" — false; ask explicitly.

## T3 — Pre-implementation rule check

### Position

T3 is a subsection of the existing `## Recall (action=recall)` — it is the **first automatic-trigger structured recall procedure**. Recall existed as informal capability ("the index is loaded, scan it"). T3 makes a specific high-leverage moment structured.

### Trigger signals (ALL must apply)

- AI is about to make code changes that materially affect behavior (new feature, refactor, multi-file work, new endpoint).
- The current task has not been T3-scanned earlier in this session (session-scoped cache).

### Skip conditions (ANY skips T3)

- Trivial edits — single-line change, typo, formatting, comment-only, lint fix.
- Routine maintenance — dependency bump, version sync, `.gitignore` edits.
- User explicitly says "skip the rule check" (or equivalent).
- Same task already T3-scanned and scope unchanged.

### Procedure

1. **Identify upcoming task** — express in one short line.
2. **Keyword scan against index** — use the `[k1, k2, k3]` lists in index lines (already in context via `@-import`).
3. **Tiered body Read** (this is the token-cost control point):
   - `severity: critical` or `zero-tolerance` → Read body unconditionally if even loosely matched.
   - `severity: high` → Read body if keyword match is strong.
   - Other severities → index line is sufficient unless explicitly relevant.
4. **Filter on reflection** — drop matches that don't actually apply to this task on second look.
5. **Announce** — concise output to user (see example below).
6. **Implement** — listed rules in active attention. Falsifiable: implementation should trace back to cited rules where applicable.

### Announce output format

```
Pre-implementation check (task: implement /auth/login endpoint)
  Index scan: 12 entries → 3 matched (keywords: auth, endpoint, jwt)
  Applying:
    - rule/2026-05-15-auth-gateway-only.md (critical)
    - architecture/2026-05-10-jwt-stateless.md
    - habit/2026-04-20-prefer-zod-validation.md
  Implementing.
```

If zero matches: `Pre-implementation check: scanned 12 entries, none matched. Implementing.`

### Token-cost guard — three defenses

| Defense | What it bounds |
|---|---|
| **Tiered body Read** | Avoids reading every entry's body — only critical-tier always loaded |
| **Session-scoped per-task cache** | One scan per task, not per edit. Cache key = the one-line task description from procedure step 1; cache invalidates when the task substantively changes (different file-set, different feature, user signals a context shift) |
| **Trivial-task skip** | Small edits don't pay full scan cost |

### Disallowed rationalizations

- "Index is in context, I don't need to scan" — being-in-context ≠ attention; the scan IS the attention mechanism.
- "Task is too small, skip" — small tasks accumulate violations; use the explicit `trivial-task skip`, not vague judgment.
- "I'll check after writing, fix if needed" — defeats the purpose; rules should constrain writing, not validate post hoc.
- "User said implement, they don't want a pause" — T3 is one short scan, not a dialogue. A few lines of announce don't constitute "pausing".

## CLAUDE.md doctrine increments (Skill design invariants)

Three new invariants, appended to the existing list:

1. **End-of-planning retrospective fires at planning-arc end, mirrors end-of-debug** (since v0.11.0). Multi-turn architectural discussion converging → AI scans the artifact, runs 3-probe split, pre-classifies permanent vs. tactical, surfaces a structured proposal. Skipping at planning end is the trigger that closes failure mode (b) — losing decisions to past-session conversation.
2. **Novel-reason correction = missing-rule signal, not just one-off feedback** (since v0.11.0). When user corrects AI with a stated reason absent from the index, that IS the gap iris-gotcha exists for. T2 captures it at moment of greatest specificity; the permanence gate must be explicitly asked (no tone inference). Existing Step 5 strengthening handles the orthogonal case (matched existing entry).
3. **Pre-implementation rule check is the first automatic-trigger of action=recall** (since v0.11.0). The @-imported index being in context ≠ AI attending to it; T3 makes attention an explicit structured step at the highest-leverage moment (before code is written). Token cost is controlled via three defenses: tiered body Read, per-task cache, trivial-task skip.

These are narrow design rules in the existing invariant-list style. No framework-level reframe ("Capture vs. Recall as orthogonal axes" was considered and rejected as premature abstraction with only one recall discipline existing).

## Version impact

| File | Change |
|---|---|
| `.claude-plugin/plugin.json` | `version: "0.10.0" → "0.11.0"` |
| `.claude-plugin/marketplace.json` | `metadata.version` and `plugins[0].version` both → `"0.11.0"` |
| `README.md` | Add `0.11.0` entry to versioning list |
| `CLAUDE.md` | Append 3 invariants per above |
| `skills/iris-gotcha/SKILL.md` | The bulk of the work (per "SKILL.md changes" section) |

Bump justification: **minor** (0.10.0 → 0.11.0). Adds new behavior without changing schema. Existing prescriptive entries unaffected. Descriptive-type `references:` requirement from v0.10.0 unaffected.

## Out of scope (deferred to future versions)

- **Session-start project scan** (failure mode (a)) — not selected by user; would need a "T0" trigger that scans `SPEC.md` / `ARCHITECTURE.md` / `docs/` at first code work in an unfamiliar project. Defer until pull is clear.
- **Capture vs. Recall as top-level concept** — would require restructuring SKILL.md. Defer until at least 2 recall disciplines exist.
- **Auto-regenerate `overview.md` after T1 captures** — could be useful but mixes T1 with overview's separate concern. Out-of-scope here.
- **Plan-file location customization** — currently relies on conventional locations (`docs/plans/`, `.claude/plans/`); making this configurable adds complexity for niche cases.

## Risks and observability

These let v0.11.1 iterate based on real-world signal:

| Risk | Signal to watch | Action if signal triggers |
|---|---|---|
| T1 permanence misclassification — too many tactical entries leak through as permanent | User overrides of pre-classification, ratio > 30% | Tighten permanence pre-classify heuristic; consider always-ask mode |
| T1 over-eager proposing — fires too often, becomes noise | Users dismissing all candidates frequently | Tighten trigger signals; raise the "3+ turn architectural discussion" bar |
| T2 false positives — captures things that should have been just-this-case | Captured T2 entries getting moved/deleted in audits | Stronger permanence-gate prompting |
| T3 token cost — scan + tiered Read costs too much per session | Time/tokens spent in T3 vs. session total | Lower body-Read tier thresholds; lengthen the per-task cache; expand trivial-task skip list |

## Future work signal

If the following patterns emerge in v0.11.x usage, they warrant follow-up design:

- Second `recall discipline` candidate emerges → revisit "Capture vs. Recall" as top-level concept.
- T1 permanence gate consistently produces same misclass pattern → enrich the pre-classify heuristic with examples.
- Users repeatedly hit failure mode (a) → design T0 (session-start scan).
- T3 cache invalidation between related-but-distinct tasks becomes painful → design explicit task-boundary signaling.
