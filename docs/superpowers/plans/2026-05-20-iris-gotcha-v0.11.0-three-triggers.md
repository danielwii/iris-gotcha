# iris-gotcha v0.11.0 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implement v0.11.0 — three new triggers (T1 end-of-planning retrospective / T2 correction-with-novel-reason / T3 pre-implementation rule check) to close the "AI doesn't know my architecture/spec" gap surfaced in brainstorming.

**Architecture:** Pure markdown edits. No code, no tests, no schema changes. Surgical insertions into existing sections of `SKILL.md` + `CLAUDE.md` + `README.md`, plus version bumps in `plugin.json` and `marketplace.json`. The 3 triggers are additive — existing 0.10.0 behavior unchanged.

**Tech Stack:** Markdown only. Validation via `grep -c` after each edit. Single coherent commit at the end (matches v0.9.x / v0.10.0 release precedent).

---

## File Structure

| File | Change |
|---|---|
| `skills/iris-gotcha/SKILL.md` | +3 new subsections (T1 / T2 / T3), +3 rows in "When to invoke" table, +2 bullets in "Common failure modes" |
| `CLAUDE.md` | +3 new entries in "Skill design invariants" list |
| `README.md` | +1 entry in Versioning list |
| `.claude-plugin/plugin.json` | `version: "0.10.0" → "0.11.0"` |
| `.claude-plugin/marketplace.json` | both `metadata.version` and `plugins[0].version` → `"0.11.0"` |

## Reference

Spec: `docs/superpowers/specs/2026-05-20-iris-gotcha-three-triggers-design.md`

---

### Task 1: Add T1 (End-of-planning retrospective) subsection to SKILL.md

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — insert T1 subsection AFTER `### End-of-debug retrospective` and BEFORE `### Triage announcement for user-triggered captures`.

- [ ] **Step 1: Verify the anchor exists**

Run:
```bash
grep -n "### Triage announcement for user-triggered captures" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: exactly one match around line 180.

- [ ] **Step 2: Apply Edit inserting T1**

Use Edit tool:
- `file_path`: `/Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md`
- `old_string`:
```
### Triage announcement for user-triggered captures (since v0.8.0)
```
- `new_string`:
``````
### End-of-planning retrospective (since v0.11.0)

A planning arc is **multi-turn design work**: discussing architecture, picking patterns, defining contracts, drafting a plan (whether via `/make-plan`, in-conversation design, or writing a spec doc). When such an arc concludes — signal-detected via any of the following — run the retrospective before transitioning to implementation.

**Trigger signals (any one fires)**:

- `/make-plan` (or equivalent planning skill) completes.
- A plan file appears in conventional plan locations (`docs/plans/` / `.claude/plans/` / `plans/` / similar). Heuristic signals — non-standard plan paths still fire via the other signals below.
- User verbal cue: "好开始", "let's implement", "go ahead", "可以了" (any explicit "transition from design to implementation" signal).
- AI judgment: 3+ turns of architectural discussion converging with no remaining design questions.

**Skip conditions**:

- Planning arc was trivial (1–2 turns, no architectural decisions).
- All candidate decisions are training-redundant (Step 0 gate would reject all).

**Procedure** (pre-procedure that runs *before* the standard Capture Step 0–9; then hands off to it per-candidate):

1. **Scan the planning artifact** — plan file, conversation history, or both. Identify candidate decisions that would change AI behavior in implementation.
2. **Apply the Step 0 training-gap filter to each candidate** — drop ones that a fresh Claude would handle correctly anyway. Surviving candidates proceed.
3. **Run the 3-probe split test on each surviving candidate** (per `definitions.md`): intent / fact / prescription. Multi-probe yes-answers produce multiple cross-referenced entries via `## Related` (per v0.7.0+ doctrine).
4. **Permanence pre-classify (T1-specific gate)** — for each candidate, label as `permanent` (architectural commitment) or `tactical` (this-implementation-only). Tactical candidates are recommended `skip` (use `claude-mem` if needed for session history).
5. **Structured proposal** — present candidates to user (format below). User confirms per-candidate; can override AI's permanence pre-classification.
6. **Each confirmed candidate runs full Capture Step 1–9**, including Step 5 strengthening check (avoids duplicating existing entries).

**Permanence gate — the load-bearing innovation**:

Plans mix two kinds of content:

| Class | Example | Action |
|---|---|---|
| **Permanent** (architectural commitment) | "Auth services must be split from business services" | Capture |
| **Tactical** (this-implementation-only) | "Dual-write to old + new tables during rollout" | Skip (claude-mem if needed) |

Without this discriminator, T1 pollutes the always-on layer with time-bounded decisions that lose relevance fast. The gate is mandatory; AI pre-classifies and the proposal surfaces the classification for user verification.

**Proposal output format**:

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

**Disallowed rationalizations**:

- "Plan is too detailed, skip capture" — long plans need distilling; the value is the high-signal subset.
- "User will capture manually if they want" — T1's entire purpose is removing the manual remember-to-capture cost.
- "Every plan decision is permanent" — false; many are tactical. Permanence gate is mandatory.
- "Capture everything; audit later" — audit fires only on explicit request; bad captures pollute the always-on layer until cleaned.

### Triage announcement for user-triggered captures (since v0.8.0)
``````

- [ ] **Step 3: Verify insertion**

Run:
```bash
grep -c "### End-of-planning retrospective" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1`

Also verify the original Triage section still appears once:
```bash
grep -c "### Triage announcement for user-triggered captures" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1` (not 0, not 2 — confirms insertion didn't duplicate or remove the anchor).

- [ ] **Step 4: No commit yet**

Tasks 1–8 batch into a single commit in Task 9. This matches v0.10.0 release precedent (single coherent commit per release).

---

### Task 2: Add T2 (Correction-with-novel-reason) subsection to SKILL.md

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — insert T2 subsection AFTER the just-inserted T1 subsection and BEFORE `### Triage announcement for user-triggered captures`.

- [ ] **Step 1: Verify the new anchor exists (the T1 subsection's last line)**

The T1 subsection ends with `- "Capture everything; audit later" — audit fires only on explicit request; bad captures pollute the always-on layer until cleaned.`

Run:
```bash
grep -n "Capture everything; audit later" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: exactly one match.

- [ ] **Step 2: Apply Edit inserting T2**

Use Edit tool. We anchor on the line right after T1's last disallowed-rationalization bullet (the blank line + `### Triage announcement` heading).

- `file_path`: `/Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md`
- `old_string`:
```
- "Capture everything; audit later" — audit fires only on explicit request; bad captures pollute the always-on layer until cleaned.

### Triage announcement for user-triggered captures (since v0.8.0)
```
- `new_string`:
``````
- "Capture everything; audit later" — audit fires only on explicit request; bad captures pollute the always-on layer until cleaned.

### Correction-with-novel-reason (since v0.11.0)

When the user pushes back on AI output with a stated reason absent from the index, that reason is a **missing-rule signal** — treat it as a capture trigger, not just one-time feedback to apply.

**Relationship to existing Step 5 strengthening** (key disambiguation):

| | Existing Step 5 | T2 |
|---|---|---|
| Trigger | AI about to repeat an **entry-recorded** mistake | User corrects with a stated reason **not** in the index |
| Outcome | Strengthen existing entry (severity ↑) | Capture **new** entry |
| Detection | AI self-monitoring | User-input pattern matching |

These are complementary, not redundant.

**Trigger pattern (must match shape AND AI's output diverged)**:

User statement matches one of:

- `[do/don't] X, because Y`
- `this violates [Y]` / `这违反了 Y`
- `we [must/should] [do/avoid] X` (with stated reason)

**AND** AI's current output/intent diverges from Y. Otherwise it's informational, not corrective.

**Skip conditions**:

- User correction without a stated reason ("不对", "改一下") — nothing to capture.
- AI's output already conforms to user's statement — informational, not corrective.
- Reason Y is already in the index — route to Step 5 strengthening instead.
- Reason Y is training-redundant (Step 0 gate would reject).
- User just invoked an explicit capture flow ("记一下") — avoid double-triggering.

**Procedure**:

1. **Extract reason Y** — verbatim quote preferred (avoid AI paraphrase drift).
2. **Search index** (user + project scope) by keyword. Index is in context via `@-import` — title + keywords scan first, body Read only on title-ambiguous candidates.
3. **If matched** — route to Step 5 strengthening. Announce: *"This is already in index at `<path>`; will strengthen instead of new entry."* Do NOT propose new.
4. **If novel** — proceed to permanence gate.
5. **Permanence gate (required ASK)** — T2 has less context than T1 (no plan document to read), so AI MUST explicitly ask: **"Is Y a permanent fact / rule / architectural distinction (capture), or just clarifying this turn (no capture)?"** The wording accommodates descriptive Y (architectural facts the AI didn't know) as well as prescriptive Y (rules). Cannot infer from tone — users state permanent knowledge conversationally.
6. **If permanent** — run full Capture Step 0–9, with **explicit attention to Step 3 (3-probe split test)**. Novel-reason corrections often span multiple aspects (intent + fact + prescription). Worked example: user clarifies "Iris has three surface types with different activity requirements" → decomposes into an `architecture` entry (why split), a `topology` entry (which-surface-needs-what), and a `lesson` entry (don't collapse them in diagnosis). Default to running all three probes; capture every `yes`-aspect as a cross-referenced entry via `## Related`. Disambiguation field MUST cite source: `"via user correction on YYYY-MM-DD"`.
7. **If just-for-this-case** — apply Y for current turn only. Tell user: *"Applying Y here. Not capturing — if it recurs, we'll capture next time."*

**Disallowed rationalizations**:

- "User is correcting me, they'll capture if they want" — silent application loses the rule for next session; T2 catches at moment of greatest specificity.
- "It's just one correction, not worth capturing" — by definition, a novel-reason correction points at an AI-unknown rule. That IS the gap iris-gotcha exists for.
- "Wait for 记一下" — by the time user remembers to invoke, the conversation has moved on and context is degraded.
- "Permanent vs. tactical can be inferred from tone" — false; ask explicitly.

### Triage announcement for user-triggered captures (since v0.8.0)
``````

- [ ] **Step 3: Verify insertion**

Run:
```bash
grep -c "### Correction-with-novel-reason" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1`

Verify ordering — T1 should come before T2, both before Triage:
```bash
grep -n "^### " /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected to include in this order: `End-of-debug retrospective`, `End-of-planning retrospective`, `Correction-with-novel-reason`, `Triage announcement`.

- [ ] **Step 4: No commit yet** (batch with Task 9)

---

### Task 3: Add T3 (Pre-implementation rule check) subsection to `## Recall`

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — append T3 subsection to the existing `## Recall (action=recall)` section (which currently has one paragraph and no subsections).

- [ ] **Step 1: Verify the anchor (current Recall section text)**

Run:
```bash
grep -n "You only need to invoke .action=recall" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: exactly one match. This is the last line of the existing Recall section paragraph.

- [ ] **Step 2: Apply Edit appending T3**

Use Edit tool:
- `file_path`: `/Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md`
- `old_string`:
```
The index is already loaded in your context via `@-import`. Scan it for keyword/title matches and Read the relevant file directly — that's recall. You only need to invoke `action=recall` explicitly when the user wants a broader multi-keyword search across both scopes or when the question is too vague to know what to Read.
```
- `new_string`:
``````
The index is already loaded in your context via `@-import`. Scan it for keyword/title matches and Read the relevant file directly — that's recall. You only need to invoke `action=recall` explicitly when the user wants a broader multi-keyword search across both scopes or when the question is too vague to know what to Read.

### Pre-implementation rule check (since v0.11.0)

The index is loaded via `@-import`, but **loading ≠ attending**. Without an explicit consultation step, AI writes code based on training defaults, even when indexed rules contradict those defaults. T3 makes attention explicit at the highest-leverage moment — before code is written. It is the first **automatic-trigger structured procedure** under `action=recall`.

**Trigger signals (ALL must apply)**:

- AI is about to make code changes that materially affect behavior (new feature, refactor, multi-file work, new endpoint).
- The current task has not been T3-scanned earlier in this session (session-scoped per-task cache).

**Skip conditions (ANY skips T3)**:

- Trivial edits — single-line change, typo, formatting, comment-only, lint fix.
- Routine maintenance — dependency bump, version sync, `.gitignore` edits.
- User explicitly says "skip the rule check" (or equivalent).
- Same task already T3-scanned and scope unchanged.

**Procedure**:

1. **Identify upcoming task** — express in one short line.
2. **Keyword scan against index** — use the `[k1, k2, k3]` lists in index lines (already in context via `@-import`).
3. **Tiered body Read** (this is the token-cost control point):
   - `severity: critical` or `zero-tolerance` → Read body unconditionally if even loosely matched.
   - `severity: high` → Read body if keyword match is strong.
   - Other severities → index line is sufficient unless explicitly relevant.
4. **Filter on reflection** — drop matches that don't actually apply to this task on second look.
5. **Announce** — concise output to user. Format:

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

6. **Implement** — listed rules in active attention. Falsifiable: implementation should trace back to cited rules where applicable.

**Token-cost guard — three defenses**:

| Defense | Bounds |
|---|---|
| **Tiered body Read** | Only `critical`/`zero-tolerance` entries always Read; lower severities cap at index line |
| **Session-scoped per-task cache** | One scan per task, not per edit. Cache key = the one-line task description from step 1; cache invalidates when the task substantively changes (different file-set, different feature, user signals a context shift) |
| **Trivial-task skip** | Small edits don't pay full scan cost |

**Disallowed rationalizations**:

- "Index is in context, I don't need to scan" — being-in-context ≠ attention; the scan IS the attention mechanism.
- "Task is too small, skip" — small tasks accumulate violations; use the explicit `trivial-task skip` rule, not vague judgment.
- "I'll check after writing, fix if needed" — defeats the purpose; rules should constrain writing, not validate post hoc.
- "User said implement, they don't want a pause" — T3 is one short scan, not a dialogue.
``````

- [ ] **Step 3: Verify insertion**

Run:
```bash
grep -c "### Pre-implementation rule check" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1`

Verify the original Recall paragraph still appears once:
```bash
grep -c "The index is already loaded in your context via" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1`

- [ ] **Step 4: No commit yet** (batch with Task 9)

---

### Task 4: Add 3 new rows to "When to invoke" table in SKILL.md

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — insert 3 new rows into the existing "When to invoke" Markdown table.

- [ ] **Step 1: Verify the anchor row**

The existing table has an "End of a non-trivial debugging arc" row. T1's row goes immediately after it. T2's row goes after the "User just corrected you on a recurring mistake" row. T3's row goes before the "An indexed entry seems relevant" row.

Run:
```bash
grep -n "End of a non-trivial debugging arc" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: exactly one match.

- [ ] **Step 2: Insert T1 row (after end-of-debug)**

Use Edit tool:
- `file_path`: `/Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md`
- `old_string`:
```
| **End of a non-trivial debugging arc** (problem resolved OR abandoned after multi-turn isolation work) | **Run end-of-debug retrospective** — see below; if it yields a gotcha, `action=capture` |
| You finish a non-trivial task and the solution is reusable | `action=capture` |
```
- `new_string`:
```
| **End of a non-trivial debugging arc** (problem resolved OR abandoned after multi-turn isolation work) | **Run end-of-debug retrospective** — see below; if it yields a gotcha, `action=capture` |
| **End of a planning arc** (multi-turn design work concluded — `/make-plan` done, verbal cue like "好开始", or 3+ turn architectural discussion converged) | **Run end-of-planning retrospective** — see below; for each confirmed candidate, `action=capture` |
| You finish a non-trivial task and the solution is reusable | `action=capture` |
```

- [ ] **Step 3: Insert T2 row (after "User just corrected you on a recurring mistake")**

Use Edit tool:
- `file_path`: `/Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md`
- `old_string`:
```
| User just corrected you on a recurring mistake | `action=capture` — Step 5 detects the duplicate and strengthens the existing entry |
| An indexed entry seems relevant to current work | Usually just Read its file directly (the path is in the index already loaded). Use `action=recall` only if you need a multi-keyword search across scopes |
```
- `new_string`:
```
| User just corrected you on a recurring mistake | `action=capture` — Step 5 detects the duplicate and strengthens the existing entry |
| User corrects you with a stated reason **absent** from the index | **Run correction-with-novel-reason check** — see below; if novel + permanent, `action=capture` |
| About to write non-trivial code (new feature / refactor / multi-file work) | **Run pre-implementation rule check** — see `## Recall`; consults the index and announces applicable rules before implementing |
| An indexed entry seems relevant to current work | Usually just Read its file directly (the path is in the index already loaded). Use `action=recall` only if you need a multi-keyword search across scopes |
```

(Note: this single Edit inserts BOTH the T2 row and the T3 row — both go before the "An indexed entry seems relevant" row. The T2 row is logically grouped with the user-correction row above; the T3 row precedes the recall row.)

- [ ] **Step 4: Verify all three rows inserted**

Run:
```bash
grep -c "End of a planning arc" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1`

```bash
grep -c "absent\*\* from the index" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1`

```bash
grep -c "About to write non-trivial code" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1`

- [ ] **Step 5: No commit yet** (batch with Task 9)

---

### Task 5: Add 2 new bullets to "Common failure modes" section in SKILL.md

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — append 2 new bullets to the existing "Common failure modes" list.

- [ ] **Step 1: Verify the anchor (last bullet of existing list)**

The existing last bullet is "Classifying from memory." Find it:
```bash
grep -n "Classifying from memory" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: exactly one match near the bottom of the file.

- [ ] **Step 2: Apply Edit appending two new bullets**

Use Edit tool:
- `file_path`: `/Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md`
- `old_string`:
```
- **Classifying from memory.** Definitions evolve. Always Read `definitions.md` before classifying, even if you "remember" how the categories work.
```
- `new_string`:
```
- **Classifying from memory.** Definitions evolve. Always Read `definitions.md` before classifying, even if you "remember" how the categories work.
- **Skipping the permanence gate (T1/T2).** Both T1 and T2 require a permanence question — is this an architectural commitment (capture) or a this-implementation-only / this-turn-only decision (skip)? Skipping the gate and capturing everything pollutes the always-on layer with time-bounded entries that lose relevance fast. The gate is mandatory because plans and corrections both mix permanent + tactical content; the discriminator must be explicit.
- **T3 scan as ceremony, not attention.** Running the pre-implementation scan and announcing matched rules, but then implementing without actually using those rules to constrain decisions, defeats T3's purpose. The scan IS the attention mechanism — listed rules should trace through to the actual code being written. If the announce reads "Applying: rule X, rule Y" but the implementation ignores them, that's T3 reduced to theater.
```

- [ ] **Step 3: Verify the bullets are in place**

Run:
```bash
grep -c "Skipping the permanence gate" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1`

```bash
grep -c "T3 scan as ceremony" /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected: `1`

- [ ] **Step 4: No commit yet** (batch with Task 9)

---

### Task 6: Add 3 new invariants to `CLAUDE.md`

**Files:**
- Modify: `CLAUDE.md` — append 3 new entries to the "Skill design invariants" bullet list (placed immediately after the existing v0.8.0 triage announcement entry, before the `## What this plugin is NOT` heading).

- [ ] **Step 1: Verify the anchor**

The anchor is the line `## What this plugin is NOT`. Find it:
```bash
grep -n "## What this plugin is NOT" /Users/daniel/Workspace/iris-gotcha/CLAUDE.md
```
Expected: exactly one match.

- [ ] **Step 2: Apply Edit appending 3 invariant entries**

Use Edit tool. The anchor includes the last existing invariant line (v0.10.0 about `references:`) so we extend the list immediately before the heading.

- `file_path`: `/Users/daniel/Workspace/iris-gotcha/CLAUDE.md`
- `old_string`:
```
- **Architecture/topology entries must declare `references:`** (since v0.10.0). The @-import mechanism that makes good descriptive entries valuable also makes stale ones actively harmful — the AI cites them confidently in downstream reasoning. `references:` (frontmatter field listing the code paths the entry depends on) is the smallest structural lever that converts drift from invisible decay to audit-detectable signal: `action=audit` mtime-compares each referenced file against the entry's `last_modified` and flags potentially-drifted entries. Fallback to `unverified: true` only when the referenced code is genuinely inaccessible; the new `## Doc Follows Code` section names "I'll add references later" as a disallowed rationalization to preempt the typical procrastination path.

## What this plugin is NOT
```
- `new_string`:
```
- **Architecture/topology entries must declare `references:`** (since v0.10.0). The @-import mechanism that makes good descriptive entries valuable also makes stale ones actively harmful — the AI cites them confidently in downstream reasoning. `references:` (frontmatter field listing the code paths the entry depends on) is the smallest structural lever that converts drift from invisible decay to audit-detectable signal: `action=audit` mtime-compares each referenced file against the entry's `last_modified` and flags potentially-drifted entries. Fallback to `unverified: true` only when the referenced code is genuinely inaccessible; the new `## Doc Follows Code` section names "I'll add references later" as a disallowed rationalization to preempt the typical procrastination path.
- **End-of-planning retrospective fires at planning-arc end, mirrors end-of-debug** (since v0.11.0). Multi-turn architectural discussion converging → AI scans the artifact, runs the 3-probe split, pre-classifies each candidate as `permanent` (architectural commitment, capture) vs. `tactical` (this-implementation-only, skip), surfaces a structured proposal for per-candidate user confirmation. The permanence gate is the load-bearing innovation: without it, T1 pollutes the always-on layer with time-bounded decisions that lose relevance fast.
- **Novel-reason correction = missing-rule signal, not just one-off feedback** (since v0.11.0). When the user corrects AI with a stated reason absent from the index, that IS the gap iris-gotcha exists for. T2 captures at the moment of greatest specificity; the permanence gate must be explicitly asked (no tone inference) and is worded to accommodate descriptive Y (architectural facts the AI didn't know) as well as prescriptive Y (rules). Existing Step 5 strengthening handles the orthogonal case (reason Y already in index → strengthen, don't add new).
- **Pre-implementation rule check is the first automatic-trigger of `action=recall`** (since v0.11.0). The @-imported index being in context ≠ AI attending to it; T3 makes attention an explicit structured step at the highest-leverage moment (before code is written). Token cost is controlled via three defenses: tiered body Read (only `critical`/`zero-tolerance` always Read full body), session-scoped per-task cache (one scan per task, not per edit), and trivial-task skip (small edits don't pay full scan cost).

## What this plugin is NOT
```

- [ ] **Step 3: Verify the 3 new invariants are present**

Run:
```bash
grep -c "End-of-planning retrospective fires at planning-arc end" /Users/daniel/Workspace/iris-gotcha/CLAUDE.md
```
Expected: `1`

```bash
grep -c "Novel-reason correction = missing-rule signal" /Users/daniel/Workspace/iris-gotcha/CLAUDE.md
```
Expected: `1`

```bash
grep -c "Pre-implementation rule check is the first automatic-trigger" /Users/daniel/Workspace/iris-gotcha/CLAUDE.md
```
Expected: `1`

- [ ] **Step 4: No commit yet** (batch with Task 9)

---

### Task 7: Version bumps in plugin.json and marketplace.json

**Files:**
- Modify: `.claude-plugin/plugin.json` — `version: "0.10.0" → "0.11.0"`
- Modify: `.claude-plugin/marketplace.json` — both `metadata.version` and `plugins[0].version` to `"0.11.0"` (one Edit with `replace_all: true` works since both occurrences are identical).

- [ ] **Step 1: Verify current versions**

Run:
```bash
grep -n "0.10.0\|0.11.0" /Users/daniel/Workspace/iris-gotcha/.claude-plugin/plugin.json /Users/daniel/Workspace/iris-gotcha/.claude-plugin/marketplace.json
```
Expected: 3 lines, all showing `0.10.0` (plugin.json:4, marketplace.json:8, marketplace.json:15). None should show `0.11.0` yet.

- [ ] **Step 2: Bump plugin.json**

Use Edit tool:
- `file_path`: `/Users/daniel/Workspace/iris-gotcha/.claude-plugin/plugin.json`
- `old_string`: `  "version": "0.10.0",`
- `new_string`: `  "version": "0.11.0",`

- [ ] **Step 3: Bump marketplace.json (both occurrences)**

Use Edit tool with `replace_all: true`:
- `file_path`: `/Users/daniel/Workspace/iris-gotcha/.claude-plugin/marketplace.json`
- `old_string`: `"version": "0.10.0"`
- `new_string`: `"version": "0.11.0"`
- `replace_all`: `true`

- [ ] **Step 4: Verify all 3 version fields updated**

Run:
```bash
grep -n "0.10.0\|0.11.0" /Users/daniel/Workspace/iris-gotcha/.claude-plugin/plugin.json /Users/daniel/Workspace/iris-gotcha/.claude-plugin/marketplace.json
```
Expected: 3 lines, all showing `0.11.0`. None should show `0.10.0`.

- [ ] **Step 5: No commit yet** (batch with Task 9)

---

### Task 8: Add v0.11.0 entry to README.md versioning list

**Files:**
- Modify: `README.md` — insert v0.11.0 entry above the existing v0.10.0 entry (which sits just below the v0.4.1 entry in the file's idiosyncratic ordering).

- [ ] **Step 1: Verify the anchor (existing v0.10.0 entry)**

Run:
```bash
grep -n "^- .0\.10\.0." /Users/daniel/Workspace/iris-gotcha/README.md
```
Expected: exactly one match.

- [ ] **Step 2: Apply Edit inserting v0.11.0 entry above v0.10.0**

Use Edit tool. The anchor is the unique starting text of the v0.10.0 entry, which we anchor on together with the v0.4.1 line above it.

- `file_path`: `/Users/daniel/Workspace/iris-gotcha/README.md`
- `old_string`:
```
- `0.10.0` — **doc-follows-code drift discipline** for `architecture` / `topology` entries. Adds a new top-level `## Doc Follows Code` section to SKILL.md requiring every descriptive entry to declare a `references: [path1, path2, ...]` frontmatter field listing the code paths the entry depends on, and extends `action=audit` to compare each referenced file's mtime against the entry's `last_modified` — flagging stale entries as "potentially drifted" rather than letting them silently misinform sessions. Frontmatter schema gains `references:` (for descriptive types) and optional `unverified: true` (fallback when referenced code is inaccessible). The @-import mechanism that makes good architecture/topology entries valuable also makes stale ones actively harmful — the AI cites them confidently in downstream reasoning; `references:` is the smallest structural lever that converts drift from invisible decay to audit-detectable signal. No procedure changes for prescriptive types; signal-density improvement = converting one previously-invisible failure mode (stale descriptive entries) into one auditable one.
```
- `new_string`:
```
- `0.11.0` — **three new triggers (T1 / T2 / T3)** addressing the "AI doesn't know my architecture/spec" gap. SKILL.md adds: **T1 end-of-planning retrospective** (capture-side): when a planning arc concludes (`/make-plan` done, verbal "好开始" cue, or 3+ turn architectural discussion converging), AI scans the artifact, runs the 3-probe split test, pre-classifies each candidate as `permanent` (architectural commitment → capture) or `tactical` (this-implementation-only → skip), surfaces a structured proposal for per-candidate user confirmation. Mirror-symmetric to v0.8.0's end-of-debug retrospective. **T2 correction-with-novel-reason** (capture-side): when user corrects AI with a stated reason absent from the index, treat it as a missing-rule signal — capture as new entry (vs. existing Step 5 which strengthens already-matched entries). Permanence gate must be explicitly asked (wording accommodates descriptive Y as well as prescriptive Y); novel-reason corrections often span all three probes (intent + fact + prescription) so the 3-probe split test is called out explicitly in the procedure. **T3 pre-implementation rule check** (recall-side, first automatic-trigger structured procedure under `action=recall`): before writing non-trivial code, AI scans the index by keyword, tiered-Reads relevant entry bodies (`critical`/`zero-tolerance` always; lower tiers cap at index line), and announces the applicable rules. Token cost controlled via tiered Read + session-scoped per-task cache + trivial-task skip. CLAUDE.md gains 3 narrow invariants (one per trigger). Common failure modes expanded with 2 new entries ("Skipping the permanence gate" and "T3 scan as ceremony, not attention"). No schema change, no breaking change. Failure mode (a) — session-start project scan — deferred: the design bets T1/T2/T3 partially mitigate by putting captured rules in the always-on layer plus T3 forcing attention before implementation; revisit when pull is clear.
- `0.10.0` — **doc-follows-code drift discipline** for `architecture` / `topology` entries. Adds a new top-level `## Doc Follows Code` section to SKILL.md requiring every descriptive entry to declare a `references: [path1, path2, ...]` frontmatter field listing the code paths the entry depends on, and extends `action=audit` to compare each referenced file's mtime against the entry's `last_modified` — flagging stale entries as "potentially drifted" rather than letting them silently misinform sessions. Frontmatter schema gains `references:` (for descriptive types) and optional `unverified: true` (fallback when referenced code is inaccessible). The @-import mechanism that makes good architecture/topology entries valuable also makes stale ones actively harmful — the AI cites them confidently in downstream reasoning; `references:` is the smallest structural lever that converts drift from invisible decay to audit-detectable signal. No procedure changes for prescriptive types; signal-density improvement = converting one previously-invisible failure mode (stale descriptive entries) into one auditable one.
```

- [ ] **Step 3: Verify v0.11.0 entry is in place and above v0.10.0**

Run:
```bash
grep -nE "^- .0\.(10|11)\.0." /Users/daniel/Workspace/iris-gotcha/README.md
```
Expected: 2 lines. The 0.11.0 line must precede the 0.10.0 line (lower line number).

- [ ] **Step 4: No commit yet** (Task 9)

---

### Task 9: Final verification and single coherent commit

**Files:** (just review, no changes)

- [ ] **Step 1: Review git status and diff**

Run:
```bash
git -C /Users/daniel/Workspace/iris-gotcha status --short
```
Expected: 5 modified files:
- `M .claude-plugin/marketplace.json`
- `M .claude-plugin/plugin.json`
- `M CLAUDE.md`
- `M README.md`
- `M skills/iris-gotcha/SKILL.md`

(Plus untracked `?? .claude/settings.local.json` — local file, don't stage.)

- [ ] **Step 2: Review diff stats**

Run:
```bash
git -C /Users/daniel/Workspace/iris-gotcha diff --stat
```
Expected (approx): SKILL.md +200 lines, CLAUDE.md +3 lines, README.md +1 line, version files +/-1 each.

- [ ] **Step 3: Spot-check SKILL.md heading order**

Run:
```bash
grep -n "^### " /Users/daniel/Workspace/iris-gotcha/skills/iris-gotcha/SKILL.md
```
Expected order to include:
- `### End-of-debug retrospective (since v0.8.0)`
- `### End-of-planning retrospective (since v0.11.0)`
- `### Correction-with-novel-reason (since v0.11.0)`
- `### Triage announcement for user-triggered captures (since v0.8.0)`
- `### Pre-implementation rule check (since v0.11.0)` (under `## Recall`)

- [ ] **Step 4: Stage the 5 modified files and commit**

Run:
```bash
git -C /Users/daniel/Workspace/iris-gotcha add .claude-plugin/marketplace.json .claude-plugin/plugin.json CLAUDE.md README.md skills/iris-gotcha/SKILL.md
```

Then commit with the message below. Use HEREDOC for correct multi-line formatting:

```bash
git -C /Users/daniel/Workspace/iris-gotcha commit -m "$(cat <<'EOF'
v0.11.0: three new triggers (T1 / T2 / T3) for the doctrine-doesn't-reach-AI gap

Brainstorming surfaced a real failure mode: user designs an architecture or
spec, AI in next session doesn't know, develops in non-conforming ways. Three
failure modes named — spec in files but AI didn't read, spec only in past
conversation, plan read but AI drifts anyway. v0.11.0 closes the second two
via two capture-side triggers and one recall-side trigger:

  - T1 end-of-planning retrospective (capture-side). When a planning arc
    concludes (/make-plan done, "好开始" cue, or 3+ turn architectural
    discussion converged), AI scans the artifact, runs the 3-probe split
    test, pre-classifies each candidate as permanent vs. tactical, surfaces
    a structured proposal. Mirror-symmetric to v0.8.0 end-of-debug
    retrospective. The permanence gate is the load-bearing innovation —
    without it, T1 pollutes the always-on layer with "this rollout only"
    decisions that lose relevance fast.

  - T2 correction-with-novel-reason (capture-side). When user corrects AI
    with a stated reason NOT in the index, that IS the gap iris-gotcha
    exists for. T2 captures at the moment of greatest specificity, vs.
    existing Step 5 which only strengthens already-matched entries.
    Permanence gate explicitly asked (wording handles descriptive Y as well
    as prescriptive Y); 3-probe split called out in procedure because
    novel-reason corrections often span intent + fact + prescription
    simultaneously (worked Iris-surface-types example included).

  - T3 pre-implementation rule check (recall-side). First automatic-trigger
    structured procedure under action=recall. Loading the index via
    @-import ≠ attending to it. T3 forces structured attention before
    code is written. Token cost controlled via three defenses: tiered body
    Read (critical/zero-tolerance always; lower tiers cap at index line),
    session-scoped per-task cache (one scan per task, not per edit),
    trivial-task skip (small edits don't pay full scan cost).

Scope:

  - SKILL.md: 3 new trigger subsections (T1, T2 in capture-trigger area
    next to end-of-debug; T3 under ## Recall as the first automatic-
    trigger of action=recall). +3 rows in "When to invoke" table.
    +2 bullets in "Common failure modes".
  - CLAUDE.md: 3 new design invariants (one per trigger), following the
    existing "since vX.Y.Z" style.
  - README.md: v0.11.0 versioning entry.
  - No schema changes. No breaking changes. Prescriptive-type entries
    unaffected; v0.10.0 references:/unverified: contract unchanged.

Surgical change — not a restructure. Considered promoting Capture vs.
Recall to top-level conceptual axes but rejected as premature abstraction
(only one recall discipline exists). Considered adding T0 (session-start
project scan) to address failure mode (a) but user did not select that
trigger; design bets T1/T2/T3 partially mitigate (a) and revisits if pull
is clear.

Spec: docs/superpowers/specs/2026-05-20-iris-gotcha-three-triggers-design.md
Plan: docs/superpowers/plans/2026-05-20-iris-gotcha-v0.11.0-three-triggers.md
EOF
)"
```

- [ ] **Step 5: Verify commit landed**

Run:
```bash
git -C /Users/daniel/Workspace/iris-gotcha log --oneline -1
```
Expected: one new commit titled `v0.11.0: three new triggers (T1 / T2 / T3) ...`.

```bash
git -C /Users/daniel/Workspace/iris-gotcha status --short
```
Expected: clean working tree (only the untracked `.claude/settings.local.json` remains).

---

### Task 10: Tag v0.11.0 and push (requires explicit user push-to-main authorization)

**Files:** (git operations only)

**Important context for the executor:** the Claude Code auto-mode classifier blocks direct push-to-main without fresh explicit user authorization per push. The user has historically authorized iris-gotcha pushes by saying "push to main 授权" or equivalent. If the executor is in an interactive session, they should ask the user; if non-interactive, they should stop after creating the tag locally and surface the push command for the user to run via `!` prefix.

- [ ] **Step 1: Create the v0.11.0 tag**

Run:
```bash
git -C /Users/daniel/Workspace/iris-gotcha tag v0.11.0
```

Then verify:
```bash
git -C /Users/daniel/Workspace/iris-gotcha tag -l 'v0.11.0'
```
Expected: `v0.11.0`

- [ ] **Step 2: Ask user for explicit push-to-main authorization (if interactive)**

Prompt: *"v0.11.0 commit and tag are local. To push commit + tag to main, I need a fresh push-to-main authorization. Confirm?"*

Wait for explicit authorization.

- [ ] **Step 3: Push commit + tag**

After authorization received, run:
```bash
git -C /Users/daniel/Workspace/iris-gotcha push origin main v0.11.0
```

Expected output:
```
To github.com:danielwii/iris-gotcha.git
   <old>..<new>  main -> main
 * [new tag]     v0.11.0 -> v0.11.0
```

If the classifier blocks the push despite authorization, fall back to surfacing the `!` command for the user to run:
```
!git -C /Users/daniel/Workspace/iris-gotcha push origin main v0.11.0
```

- [ ] **Step 4: Confirm push succeeded**

Run:
```bash
git -C /Users/daniel/Workspace/iris-gotcha log --oneline -1
git -C /Users/daniel/Workspace/iris-gotcha tag -l 'v0.11.0' --format='%(refname:short) -> %(objectname:short)'
```

Expected: commit and tag both confirmed locally; remote URL output from previous step confirms remote propagation.

---

## Optional follow-up (NOT in this plan's scope)

These are deferred per the spec's "Out of scope" section. Don't do these in this plan; surface for future planning if signal emerges:

- **T0 — session-start project scan**: deferred until pull is clear (failure mode (a) is not the user's biggest pain right now).
- **Auto-regenerate `overview.md` after T1 captures**: would couple T1 with overview's separate concern; revisit if T1 captures consistently produce overview-eligible content.
- **Dogfood entries for v0.11.0 doctrine in `.claude/iris-gotcha/`**: similar to v0.10.0's `markdown-only-doctrine.md` + `no-hooks-in-iris-gotcha.md`, could capture v0.11.0's "Capture vs. Recall as two axes" as a project-scope `architecture` entry. Lightweight, but optional.
- **GitHub release for v0.11.0**: per CLAUDE.md, the release is optional; tag push alone is sufficient for Claude Code plugin discovery.

## Risks and observability signals to watch in v0.11.x

(Surfaced in spec; not implemented as code but worth tracking in real-world usage.)

| Risk | Signal | Action if it appears |
|---|---|---|
| T1 permanence misclassification (too many tactical entries leaking through as permanent) | User overrides AI's pre-classification > 30% | Tighten the permanence pre-classify heuristic; consider an always-ask mode in v0.11.1 |
| T1 over-eager (fires too often) | Users dismissing all candidates frequently | Raise the "3+ turn architectural discussion" threshold or tighten trigger signals |
| T2 false positives (captures things that should have been just-this-case) | T2-sourced entries later moved or deleted in audits | Strengthen the permanence-gate question wording |
| T3 token cost (scan + tiered Read costs too much per session) | Time/tokens spent in T3 relative to session total | Lower body-Read tier thresholds, lengthen the per-task cache, expand trivial-task skip list |
