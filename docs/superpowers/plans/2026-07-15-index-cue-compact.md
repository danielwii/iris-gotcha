# iris-gotcha `index_cue` Compact — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stop `index.md` from growing unbounded by (a) giving every entry a canonical, budget-checked `index_cue` field that is the *only* natural language regeneration is allowed to copy into the index, (b) closing the two edge boundary failures that let a single entry absorb multiple unrelated scenario→constraint mappings and let world-state facts get baked into prescriptions as permanent premises, and (c) dogfooding the result against the real `unee/.claude/iris-gotcha/` notebook that triggered this work (57.8k chars, over the 40k injection warning threshold).

**Architecture:** This is a markdown-only Claude Code plugin (no runtime, no build step — see `CLAUDE.md`'s "No hooks" / "Markdown-only" invariants). All changes are edits to `SKILL.md` (the procedure), `definitions.md` (unchanged — category taxonomy is orthogonal to this work), `CLAUDE.md` (new invariant), `README.md` (versioning entry), and the two manifest files (version bump). "Tests" for this kind of change are: (1) grep/diff checks that the exact required text landed at the exact required place, (2) a dogfood run of the new Step 8 regeneration algorithm against the real over-budget index that motivated this plan, producing a concrete before/after character count.

**Tech Stack:** Markdown, YAML frontmatter, git. No code, no test runner.

## Global Constraints

- Every prescriptive `index_cue` MUST read as scenario + constraint in one sentence; every descriptive `index_cue` MUST be the shortest accurate current-state statement — see Task 1's budget table (soft/hard limits copied verbatim from the design spec: cue 60/100 code points, keywords 4/5, keyword 24/40, rendered line 160/220, whole file 24,000/32,000 chars).
- Regeneration (Step 8) is a pure function of entry frontmatter — it must never read or carry forward text from the existing `index.md`.
- Hard-budget violations during regeneration are a **write failure**, not a truncation trigger — the offending entries must be reported, and the old `index.md` must be left untouched.
- `action=audit` only reports; it never auto-fixes entries (existing invariant, unchanged, must not be violated by the new checks).
- Six categories stay six (`lesson`/`rule`/`architecture`/`topology`/`habit`/`best-practice`) — this plan adds a frontmatter field and procedure steps, not a new category.
- Any change to `unee/.claude/iris-gotcha/` entry files (splitting, rewriting `index_cue`) requires an explicit user confirmation step before the file write — this mirrors the design spec's requirement that historical-knowledge disposition (e.g. whether the ArgoCD entry still has future value) is a project-fact judgment the user must make, not one the plan executes unilaterally.

---

## Part A — Plugin skill upgrade (`/Users/daniel/.claude/plugins/marketplaces/iris-gotcha`)

This is Daniel's own plugin source repo (`github.com/danielwii/iris-gotcha`, installed via marketplace at this path; the `~/.claude/plugins/cache/...` copy is a read-only mirror refreshed by Claude Code — **do not edit the cache copy**, edits there are silently overwritten). Confirmed via `git remote -v` → `git@github.com:danielwii/iris-gotcha.git`, branch `main`.

### Task 1: Add `index_cue` field to the frontmatter spec

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md:130-134` (insert new subsection after the frontmatter code block, before the "Body in Markdown" line)

**Interfaces:**
- Produces: the `index_cue` field name, its two budget tables (soft/hard), and the split-by-type writing rule — every later task (0.5/0.6 gates, Step 5/6 comparison, Step 8 regeneration, audit checks) references this field and these exact budget numbers by name.

- [ ] **Step 1: Read the current frontmatter block to confirm anchor text**

Run: `sed -n '117,136p' skills/iris-gotcha/SKILL.md`

Expected: shows the `entry frontmatter` code block ending with `disambiguation: "..."` / `---` / closing code fence, then the "Initial capture" paragraph, then the "Descriptive types" paragraph, then a blank line, then `Body in Markdown. Prescriptive types...`.

- [ ] **Step 2: Insert the `index_cue` subsection**

Insert this text immediately after the line `Descriptive types (\`architecture\` / \`topology\`): omit \`severity\` / \`last_violated\` / \`violation_count\`; instead add \`references: [path1, path2, ...]\` listing the code paths the entry depends on (see \`## Doc Follows Code\` for the discipline). If the referenced code is genuinely inaccessible in this session, set \`unverified: true\` so future audits re-check.` and before the line `Body in Markdown. Prescriptive types (\`lesson\` / \`rule\` / \`habit\` / \`best-practice\`) use:`:

```markdown

**`index_cue`** (mandatory on every entry, prescriptive and descriptive alike): the *only* natural-language text that regeneration (Step 8) is allowed to copy into `index.md`. Everything else in an index line — path, keywords, severity — is copied mechanically from other frontmatter fields; `title` never appears in the index and can stay as detailed as the entry needs.

- **Prescriptive types** (`lesson` / `rule` / `habit` / `best-practice`): must read as *scenario + constraint* in one sentence. If the constraint is unconditionally true regardless of project state, the scenario clause can be omitted. But if the constraint only applies when a particular tool, mechanism, or environment is in play (e.g. "if using ArgoCD kustomize image override..."), the scenario clause is mandatory. Never phrase `index_cue` as a bare statement of current project state ("prod uses ArgoCD") when the actual claim is conditional on that state — that formulation silently goes wrong the moment the project's actual mechanism changes, and nothing will catch the staleness. See Step 0.6 for the underlying discipline this enforces.
- **Descriptive types** (`architecture` / `topology`): the shortest accurate statement of current project state, or the one design distinction that differentiates this entry from a default assumption. No conditional clause needed — descriptive entries state facts; prescriptive entries react to them.

Budget (soft = should stay under during Step 6.5 self-check; hard = Step 8 regeneration refuses to write the index and reports the offending entry instead):

| Field | Soft | Hard |
|---|---:|---:|
| `index_cue` | 60 code points | 100 code points |
| `keywords` count | 4 | 5 |
| each keyword | 24 code points | 40 code points |
| rendered index line (title + keywords + severity, excluding path) | 160 code points | 220 code points |
| whole `index.md` | 24,000 characters | 32,000 characters |

Example (prescriptive, scenario-qualified):
```yaml
index_cue: "If the CD mechanism is ArgoCD kustomize image override, the image key MUST be the full ECR URI (not a short name), and the resulting Deployment image must be verified"
```

Example (descriptive, current-state statement):
```yaml
index_cue: "v2 is currently deployed via manual rollout; ArgoCD is installed but not yet the CD mechanism"
```

`index_cue` has no language requirement — write it in whatever language the rest of the project's notebook uses. The requirement is structural (scenario + constraint, or shortest current-state fact), not linguistic.
```

- [ ] **Step 3: Verify the insertion landed correctly**

Run: `grep -n "index_cue" skills/iris-gotcha/SKILL.md | head -5`

Expected: at least one match around the line range you just edited (the exact line number will have shifted downstream sections by ~27 lines — that's expected and doesn't require updating other line-number references, since the plan below locates every other insertion by content grep, not by absolute line number).

- [ ] **Step 4: Commit**

```bash
cd /Users/daniel/.claude/plugins/marketplaces/iris-gotcha
git add skills/iris-gotcha/SKILL.md
git commit -m "feat(SKILL): add index_cue canonical field + budget table"
```

---

### Task 2: Add Step 0.5 and Step 0.6 gates to Capture procedure

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — insert between the end of `### 0. Training-gap gate` and the start of `### 1. Re-read \`definitions.md\``

**Interfaces:**
- Consumes: nothing new (uses concepts from Task 1: `index_cue` scenario+constraint framing).
- Produces: the "one-sentence scenario+constraint" atomicity test (referenced by Step 6.5 and by `action=audit`'s new atomicity check) and the world-state-vs-behavioral-constraint distinction (referenced by Step 0.6 itself, by `action=audit`'s new applicability check, and by the new Common-failure-modes bullet).

- [ ] **Step 1: Locate the exact anchor**

Run: `grep -n "^### 0\. Training-gap gate\|^### 1\. Re-read" skills/iris-gotcha/SKILL.md`

Expected: two lines, e.g. `337:### 0. Training-gap gate...` and `361:### 1. Re-read \`definitions.md\``. Confirm the line immediately before `### 1.` reads `The notebook is a supplement to training, not a re-statement of it. If capturing this entry would, after the fact, feel like "the AI didn't need to be told this", you guessed wrong at this step — go back and skip.`

- [ ] **Step 2: Insert the two new gates**

Insert this text immediately after that "notebook is a supplement to training..." sentence and immediately before the `### 1. Re-read \`definitions.md\`` header:

```markdown

### 0.5. Scenario-behavior atomicity gate

Before drafting anything, check whether the knowledge can be stated as **one sentence** of the form "in scenario S, do/avoid X" (prescriptive types) or one sentence of fact (descriptive types). Step 6.5 re-applies this exact test to the drafted `index_cue` later — running it here, before drafting the full entry, avoids writing a whole entry that then fails the later check.

- **Can state it in one sentence** → proceed normally through Step 1 onward.
- **Needs "if A then X; if B then Y"** → this is two (or more) entries wearing one trenchcoat. Split into separate candidates now, and run each candidate through Step 0 and this gate independently before continuing. Do not draft a single entry that tries to cover both scenarios "for completeness" — that's the failure mode that produced a single `lesson` entry mixing "web+scheduler deployments must release together" (constraint independent of deploy tooling) with "ArgoCD kustomize image keys must be the full URI" (constraint conditional on using ArgoCD), discovered during a 2026-07 index-compact audit where the merged entry had grown to 179 lines and 4 unrelated "violations" across two different rules.
- **Can state it in one sentence, but the sentence exceeds the `index_cue` hard budget** (Task 1) → usually the same problem in disguise: the sentence needed the extra length because it was silently covering more than one behavioral branch. Re-examine before assuming it's merely verbose.

### 0.6. World-state vs behavioral-constraint gate

Ask: does this knowledge describe **what the project currently is** (a fact that can change independently of anyone's actions — which CD tool is wired up, which auth flow a service uses, which environment is live) or **what must happen in a given scenario** (a constraint that holds regardless of the project's current state)?

- **World state** ("we currently deploy via ArgoCD", "v2 has no read replica yet") → must be `architecture` or `topology`, never folded into a `lesson`/`rule` prescription as an unstated premise.
- **Behavioral constraint** ("if deploying via ArgoCD kustomize override, the image key must be the full URI") → prescriptive type, and the scenario clause must be explicit in both the body and the `index_cue` (Task 1) — never assume "the current project state" as an implicit precondition.

The two are related but must not collapse into each other. A common failure: a prescriptive entry's prescription is written as if the world-state premise were permanent ("prod uses ArgoCD, so always run `argocd app set`"), when the premise is actually a fact that can change (the project switches CD mechanisms) while the underlying constraint (full-URI image keys, *if* ArgoCD is in play) remains valid indefinitely. Writing the world-state fact as a separate `architecture`/`topology` entry, and writing the prescriptive entry with an explicit conditional clause, keeps the prescription correct even after the world-state fact changes — and keeps the world-state fact itself auditable independently (via `references:` and `action=audit`'s drift check, see `## Doc Follows Code`) instead of buried as an assumption inside a lesson that has no drift-detection mechanism at all.

If a draft entry fails this gate (mixes both), split it: the world-state half becomes (or updates) an `architecture`/`topology` entry; the constraint half becomes a prescriptive entry with an explicit scenario clause, cross-referenced via `## Related`.

**Disallowed rationalizations**:

- "It's basically the same thing, I'll just write both branches in one cue" — see Step 0.5; write two entries instead.
- "The current CD mechanism is obviously ArgoCD, I don't need to say so" — that's exactly the assumption Step 0.6 exists to catch; make it explicit or split the fact into a separate `architecture`/`topology` entry.
```

- [ ] **Step 3: Verify heading numbering and order**

Run: `grep -n "^### [0-9]" skills/iris-gotcha/SKILL.md`

Expected output includes, in this exact order: `### 0. Training-gap gate...`, `### 0.5. Scenario-behavior atomicity gate`, `### 0.6. World-state vs behavioral-constraint gate`, `### 1. Re-read...`, `### 2. Determine scope...`, `### 3. Classify`, `### 4. Write the \`disambiguation\` field`, `### 5. Check for existing related entries...`, `### 6. Strengthen...`, `### 7. Write the entry...`, `### 8. Regenerate the index`, `### 9. Report`.

- [ ] **Step 4: Commit**

```bash
git add skills/iris-gotcha/SKILL.md
git commit -m "feat(SKILL): add Step 0.5/0.6 capture gates (scenario atomicity, world-state vs constraint)"
```

---

### Task 3: Upgrade Step 5's strengthening-gate table to a scenario+constraint comparison

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — replace the existing `### 5. Check for existing related entries (strengthening gate)` table, and add a one-paragraph pre-check to `### 6. Strengthen`

**Interfaces:**
- Consumes: the scenario+constraint framing from Task 2 (Step 0.6) and the `index_cue` field from Task 1.
- Produces: the four-row comparison table other tasks (Task 6's Common-failure-modes bullet) reference.

- [ ] **Step 1: Locate current Step 5 content**

Run: `grep -n "^### 5\. Check for existing\|^### 6\. Strengthen" skills/iris-gotcha/SKILL.md`

Read the block between those two headers (currently: one paragraph + a 4-row table with columns `Relationship` / `Action`).

- [ ] **Step 2: Replace the Step 5 table**

Replace the entire content between `### 5. Check for existing related entries (strengthening gate)` and `### 6. Strengthen (instead of duplicate)` with:

```markdown
### 5. Check for existing related entries (strengthening gate)

Read the relevant `index.md` (user index always; project index too if scope=project). Look for entries sharing keywords or addressing the same underlying concept. For each candidate, Read the full file. Then compare **scenario** and **behavioral constraint** explicitly — not just topic:

> Write one line for the existing entry's scenario+constraint, and one line for the new content's scenario+constraint. Compare them literally, side by side, before deciding.

| Existing entry | New content | Action |
|---|---|---|
| Same scenario, same constraint | Same scenario, same constraint | Step 6: strengthen. New evidence goes in the body; `index_cue` may be reworded for clarity but must not gain a new scenario branch. |
| Same scenario, contradictory constraint | — | Stop and surface to the user. Silent overwrite would destroy information. |
| Same scenario, different (non-contradictory) constraint | — | New entry, `## Related` cross-reference. Do **not** strengthen — this is "same topic, different rule", the exact failure mode that produced a mixed web+scheduler/ArgoCD entry (see Step 0.5's incident note). |
| Different scenario (even if topically similar, e.g. both about deployment) | — | New entry, regardless of topical similarity. Do not strengthen. |
| No match on scenario or constraint | — | Continue to Step 7 as a fresh entry. |

"Topically related" is not the test. Two entries about deployment, or two entries surfaced by the same incident, can still have different scenarios or different constraints and must stay separate entries.

```

- [ ] **Step 3: Add the pre-check paragraph to Step 6**

Immediately after the `### 6. Strengthen (instead of duplicate)` header and its existing first paragraph (`Applies only to prescriptive types...there's no "severity" to bump.`), insert this paragraph before the numbered list (`1. Bump severity one level...`):

```markdown

Before touching the file, confirm Step 5 actually reached "same scenario, same constraint" — don't strengthen on topical resemblance alone. If you skipped writing out the two comparison lines in Step 5, go back and write them now; strengthening is the highest-risk step for silently absorbing an unrelated scenario into an existing entry's `index_cue`.
```

- [ ] **Step 4: Verify structure**

Run: `sed -n '/^### 5\. Check for existing/,/^### 7\. Write the entry/p' skills/iris-gotcha/SKILL.md | grep -c "^|"`

Expected: at least 6 (the table header row + separator + 5 data rows), confirming the table replaced correctly and wasn't accidentally duplicated.

- [ ] **Step 5: Commit**

```bash
git add skills/iris-gotcha/SKILL.md
git commit -m "feat(SKILL): Step 5 strengthening gate compares scenario+constraint, not topic"
```

---

### Task 4: Add Step 6.5 (`index_cue` authoring and self-check)

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — insert between `### 6. Strengthen` and `### 7. Write the entry and wire injection`

**Interfaces:**
- Consumes: the budget table from Task 1, the atomicity test from Task 2 (Step 0.5).
- Produces: the requirement that every entry written in Step 7 already carries a self-checked `index_cue` — Task 5 (regeneration) assumes this was done and does not re-derive `index_cue` from anything else.

- [ ] **Step 1: Locate anchor**

Run: `grep -n "^### 6\. Strengthen\|^### 7\. Write the entry" skills/iris-gotcha/SKILL.md`

Confirm the content immediately before `### 7.` ends with the "When language stops working" subsection's last paragraph: `When you strengthen an entry into \`zero-tolerance\`, surface this in Step 9: suggest the concrete structural fix that would obviate the rule.`

- [ ] **Step 2: Insert Step 6.5**

Insert immediately after that paragraph and before `### 7. Write the entry and wire injection`:

```markdown

### 6.5. Write and self-check the `index_cue`

1. Draft the `index_cue` candidate per the rules in the frontmatter spec (scenario + constraint for prescriptive types; shortest current-state fact for descriptive types).
2. Count code points against the budget table (frontmatter spec, Task-1-equivalent section): soft 60 / hard 100 for the cue itself.
3. Check it doesn't fail Step 0.6: does it state a project's current state as a bare fact when the actual claim is conditional on that state? If so, rewrite with an explicit "if X" clause, or split the current-state fact into its own `architecture`/`topology` entry first and reference the condition from the prescriptive cue.
4. If the cue still can't be written as one clean sentence, or still exceeds the hard budget after rewriting, return to Step 0.5 — the entry is probably not atomic and needs splitting, not further compression.
5. Only once the cue passes 2–4, proceed to Step 7.
```

- [ ] **Step 3: Verify**

Run: `grep -n "^### 6\.5\." skills/iris-gotcha/SKILL.md`

Expected: one match, positioned (by line number) between the `### 6.` and `### 7.` matches from Step 1.

- [ ] **Step 4: Commit**

```bash
git add skills/iris-gotcha/SKILL.md
git commit -m "feat(SKILL): add Step 6.5 index_cue authoring self-check"
```

---

### Task 5: Rewrite Step 8 as a pure-function regeneration with budget enforcement

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — replace the entire `### 8. Regenerate the index` section

**Interfaces:**
- Consumes: `index_cue`, `keywords`, `type`, `severity`, `last_violated`, `violation_count`, file path from every entry's frontmatter (Task 1's field plus existing fields).
- Produces: the exact rendering format and validation contract that Task 8 (dogfood run) executes against real data, and that `action=audit`'s new recent-region check (Task 6) cross-checks.

- [ ] **Step 1: Locate current Step 8 content**

Run: `grep -n "^### 8\. Regenerate the index\|^### 9\. Report" skills/iris-gotcha/SKILL.md`

Read the current content between them (a short paragraph + a fenced index-format example + one closing sentence about keeping lines short).

- [ ] **Step 2: Replace with the pure-function version**

Replace the entire content between `### 8. Regenerate the index` and `### 9. Report` with:

```markdown
### 8. Regenerate the index

Regeneration is a **pure function of the entry files** — never read or carry forward text from the existing `index.md`. This matters: if the old index contains stale wording, augmented cues, or duplicated entries, re-reading it as a starting point reproduces the same bloat. Always rebuild from scratch off frontmatter.

1. **Read `index_cue`, `keywords` (first 5 only), `type`, `severity`, `last_violated`, `violation_count`, and file path from every entry's frontmatter** in the target scope. Do not read entry bodies for this step — the whole point of `index_cue` is that regeneration doesn't need to summarize anything.
2. **Placement — each entry appears exactly once**:
   - `type` ∈ {`lesson`,`rule`,`habit`,`best-practice`} AND `last_violated` is set AND within the last 7 days → place in `## ⚠️ Recently strengthened` **only**.
   - Otherwise → place in its `## <type>` category section **only**.
   - `architecture` / `topology` entries never appear in `Recently strengthened` (they have no `last_violated`).
3. **Render each line mechanically** — no added prose beyond the fields listed in step 1:
   - Prescriptive: `- **{index_cue}** [{keywords}] severity:{severity} → \`{path}\``
   - Descriptive: `- **{index_cue}** [{keywords}] → \`{path}\``
   - `Recently strengthened` entries additionally append ` · violated {last_violated}`.
4. **Validate before writing**:

   | Check | Hard budget |
   |---|---:|
   | Each `index_cue` | ≤ 100 code points |
   | Each entry's keyword count | ≤ 5 |
   | Each rendered line, excluding path | ≤ 220 code points |
   | Whole file | ≤ 32,000 characters |

   **If any hard budget is exceeded: do not write the file.** Report which entries are over budget and by how much, and stop. The fix is to shorten `index_cue`/`keywords` on the offending entries (Step 6.5) or split them (Step 0.5) — never truncate text or silently drop low-severity entries during regeneration.
5. Report line counts before/after and any validation failures, per the format in Step 9.

Soft budgets (`index_cue` ≤ 60, keywords ≤ 4, line ≤ 160, whole file ≤ 24,000) are advisory — regeneration still writes the file if only soft budgets are exceeded, but the report should note which entries are over the soft budget so the next `action=audit` can review them.
```

- [ ] **Step 3: Verify the old fenced example is gone**

Run: `sed -n '/^### 8\. Regenerate/,/^### 9\. Report/p' skills/iris-gotcha/SKILL.md | grep -c "auto-maintained"`

Expected: `0` (the old placeholder index-format fence containing "Auto-maintained by iris-gotcha" text has been fully replaced, not left alongside the new content).

- [ ] **Step 4: Commit**

```bash
git add skills/iris-gotcha/SKILL.md
git commit -m "feat(SKILL): Step 8 regeneration is now a pure function with hard budget enforcement"
```

---

### Task 6: Extend `action=audit` with applicability, atomicity, and recent-region checks

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — extend the `## Audit (\`action=audit\`)` numbered checklist (step 2's bullet list)

**Interfaces:**
- Consumes: the scenario+constraint framing (Task 2), `index_cue` (Task 1), the `Recently strengthened` placement rule (Task 5 step 2).
- Produces: three new report categories an audit run must be able to emit; no other task depends on this one's output directly, but it's the mechanism that catches regressions of Tasks 2–5 after this plan ships.

- [ ] **Step 1: Locate the audit checklist**

Run: `grep -n "^## Audit" skills/iris-gotcha/SKILL.md`

Read the existing 5-bullet checklist under item 2.

- [ ] **Step 2: Append three bullets to the existing checklist**

Immediately after the existing bullet `- No two entries cover the exact same prescription (potential merge).` and before the numbered item `3. Produce a report and let the user decide...`, insert:

```markdown
   - **Applicability**: for prescriptive entries, does `index_cue` (or the body's prescription) state a scenario, or does it silently assume "current project state" as the scenario (words like "prod" / "current" / "now" without an "if X" / "when X" clause)? If the entry has a sibling `architecture`/`topology` entry describing the relevant world-state fact but the prescriptive `index_cue` doesn't reference the condition, flag as "applicability unclear — may be stating world-state as a permanent premise" (Step 0.6).
   - **Atomicity**: does `index_cue` contain more than one "if X then Y" branch? Does the body read like multiple different scenarios each with their own fix (e.g. separately-numbered incidents each describing a different mechanism)? Flag as "possibly mixed — consider splitting" (Step 0.5, applied retroactively).
   - **Recent-region eligibility**: for the `## ⚠️ Recently strengthened` section specifically, confirm every listed entry is prescriptive AND has `last_violated` within the last 7 days AND does not also appear in its category section below (Step 8 placement rule). Flag any violation of these three conditions by entry path.
```

- [ ] **Step 3: Verify**

Run: `sed -n '/^## Audit/,/^## Push/p' skills/iris-gotcha/SKILL.md | grep -c "^   - \*\*"`

Expected: `3` (the three new bold-lead bullets, distinguishable from the five plain bullets above them which don't start with `**`).

- [ ] **Step 4: Commit**

```bash
git add skills/iris-gotcha/SKILL.md
git commit -m "feat(SKILL): audit checks applicability, atomicity, and recent-region eligibility"
```

---

### Task 7: Add two Common-failure-modes bullets

**Files:**
- Modify: `skills/iris-gotcha/SKILL.md` — append to `## Common failure modes`

**Interfaces:**
- Consumes: nothing new; summarizes Tasks 2–3 for the reader who skims only the failure-modes list.

- [ ] **Step 1: Locate the end of the file**

Run: `tail -5 skills/iris-gotcha/SKILL.md`

Expected: last line is the existing `**T3 scan as ceremony, not attention.**` bullet.

- [ ] **Step 2: Append two bullets**

Append to the end of the file (after the last existing bullet, no blank line needed before since it's a continuous list):

```markdown
- **Strengthening on topical resemblance.** "This is another deployment-related gotcha" is not the same test as "same scenario, same constraint" (Step 5). Strengthening on topic alone is how a single `lesson` entry absorbed two unrelated scenario→constraint mappings (paired-deployment release discipline, and ArgoCD image-key format) across several "violations" that were actually two different rules.
- **World-state premises baked into a prescription.** Writing "prod uses ArgoCD, so always X" instead of "if the CD mechanism is ArgoCD, then X" makes the entry silently wrong the moment the project's actual mechanism changes — and nothing will flag it, because the prescription doesn't reference a fact `action=audit`'s drift check can verify. State the world-state fact as its own `architecture`/`topology` entry (with `references:`) and make the prescriptive entry's scenario clause explicit instead (Step 0.6).
```

- [ ] **Step 3: Verify**

Run: `wc -l skills/iris-gotcha/SKILL.md` — confirm the file grew by exactly 2 lines from before this step (compare against the count from Task 6 Step 3's baseline, or simply confirm the new bullets are present via `tail -2`).

- [ ] **Step 4: Commit**

```bash
git add skills/iris-gotcha/SKILL.md
git commit -m "feat(SKILL): document strengthening-on-topic and baked-in-world-state failure modes"
```

---

### Task 8: Version bump, CLAUDE.md invariant, README versioning entry

**Files:**
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`
- Modify: `CLAUDE.md`
- Modify: `README.md`

**Interfaces:**
- Consumes: nothing code-level; this is the release-housekeeping task the repo's own `CLAUDE.md` "Version bump checklist" mandates for every SKILL.md content change of this size.

- [ ] **Step 1: Bump `plugin.json`**

Read current content, then edit the `"version"` field from `"0.11.0"` to `"0.12.0"`:

```json
{
  "name": "iris-gotcha",
  "description": "Personal knowledge notebook for Claude Code: capture and recall gotchas, rules, architecture, topology, habits, best practices, and lessons learned with strict 7-category typing and rule-strengthening on repeated violations.",
  "version": "0.12.0",
  "author": {
    "name": "Daniel Wei"
  },
  "repository": "https://github.com/danielwii/iris-gotcha",
  "license": "MIT"
}
```

- [ ] **Step 2: Bump `marketplace.json`**

Edit both `metadata.version` and `plugins[0].version` from `"0.11.0"` to `"0.12.0"`:

```json
{
  "name": "iris-gotcha",
  "owner": {
    "name": "Daniel Wei"
  },
  "metadata": {
    "description": "Personal knowledge notebook for Claude Code",
    "version": "0.12.0"
  },
  "plugins": [
    {
      "name": "iris-gotcha",
      "source": "./",
      "description": "Capture and recall gotchas, rules, architecture, topology, habits, best practices, and lessons learned with strict 7-category typing and rule-strengthening on repeated violations.",
      "version": "0.12.0",
      "author": {
        "name": "Daniel Wei"
      },
      "repository": "https://github.com/danielwii/iris-gotcha",
      "license": "MIT",
      "keywords": ["memory", "knowledge", "gotcha", "lessons-learned", "rule-strengthening"]
    }
  ]
}
```

- [ ] **Step 3: Add a CLAUDE.md invariant**

Locate the end of the "Skill design invariants" bullet list (last bullet currently: `**Pre-implementation rule check is the first automatic-trigger of \`action=recall\`**...`, immediately before the blank line and `## What this plugin is NOT` header). Append:

```markdown
- **`index_cue` is the only natural language regeneration may copy into the index** (since v0.12.0). Every entry's frontmatter carries an `index_cue` — a budget-checked single sentence (prescriptive: scenario + constraint; descriptive: shortest current-state fact). Step 8 regeneration reads frontmatter only, never the old `index.md`, and refuses to write the file if any entry exceeds the hard budget (cue 100 code points, 5 keywords, 220-char rendered line, 32,000-char whole file) — this is a write failure to be fixed by shortening or splitting the entry, not a truncation trigger. Two new capture gates (Step 0.5 scenario-behavior atomicity, Step 0.6 world-state vs behavioral-constraint) and a tightened Step 5 strengthening comparison (same scenario + same constraint, not just same topic) exist to keep entries from drifting into the mixed, budget-busting shape that motivated this version — a single `lesson` entry that had absorbed two unrelated scenario→constraint mappings across 179 lines and grown the whole project-scope index to 57.8k characters.
```

- [ ] **Step 4: Add a README versioning entry**

Locate `## Versioning` in `README.md`. Immediately after that header (before the existing `- \`0.1.0\` — initial release...` line — newest-first ordering matches the existing file, where `0.11.0` is listed before `0.10.0`), insert:

```markdown
- `0.12.0` — **`index_cue` compact**: adds a canonical `index_cue` frontmatter field (the only natural language `index.md` regeneration may copy in) with a hard/soft token budget; two new capture gates — **Step 0.5 scenario-behavior atomicity** (can this be stated as one "if scenario then constraint" sentence, or does it need splitting?) and **Step 0.6 world-state vs behavioral-constraint** (is this a fact about the project's current state, which belongs in `architecture`/`topology`, or a conditional constraint, which belongs in a prescriptive type with an explicit scenario clause?); Step 5's strengthening gate now requires comparing scenario+constraint explicitly instead of matching on topic; Step 8 regeneration is now a pure function of entry frontmatter that refuses to write an over-budget index rather than silently truncating or duplicating entries; `action=audit` gains applicability, atomicity, and recent-region eligibility checks. Motivated by a real project-scope index (`unee/.claude/iris-gotcha/index.md`) that grew to 57.8k characters, driven by (a) the `Recently strengthened` section duplicating nearly the entire index instead of a rolling 7-day window, and (b) individual entries absorbing multiple unrelated scenario→constraint mappings over repeated strengthening (a single `lesson` mixed "paired web+scheduler deployments must release together" with "ArgoCD kustomize image keys must be full URIs" — two different rules, only one of which is conditional on the project's CD tooling).
```

- [ ] **Step 5: Verify all four files**

```bash
grep -h '"version"' .claude-plugin/plugin.json .claude-plugin/marketplace.json
grep -c "index_cue" CLAUDE.md README.md
```

Expected: three `"version": "0.12.0"` matches total (one in `plugin.json`, two in `marketplace.json`), and at least one `index_cue` match in each of `CLAUDE.md` and `README.md`.

- [ ] **Step 6: Commit and tag**

```bash
git add .claude-plugin/plugin.json .claude-plugin/marketplace.json CLAUDE.md README.md
git commit -m "chore: bump to v0.12.0 (index_cue compact)"
git tag v0.12.0
```

Do **not** `git push` or push the tag without the user's explicit go-ahead — this repo is Daniel's public plugin (`github.com/danielwii/iris-gotcha`), and pushing/tagging a public release is exactly the kind of externally-visible action that needs a separate confirmation, per the standing "confirm before pushing" default.

---

## Part B — Dogfood migration of `unee/.claude/iris-gotcha/`

This is the real project-scope notebook that motivated the whole plan (`index.md` at 57.8k chars). This part **does not touch the plugin repo** — it applies the newly-shipped Step 0.5/0.6/6.5/5/8/audit procedure to real entries. Per the Global Constraints, splitting or rewriting entries requires explicit user confirmation before any file write — Tasks 9–10 are read-only and produce a proposal; Task 11 only executes after the user has reviewed that proposal; Task 12 is the mechanical regeneration once entries are settled.

### Task 9: Inventory — classify every existing entry as A/B/C/D

**Files:**
- Create: `/Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha/MIGRATION-INVENTORY.md` (scratch review artifact, not a canonical entry — not wired into any CLAUDE.md import)
- Read-only: every file under `/Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha/{lesson,rule,architecture,topology,habit,best-practice}/*.md`

**Interfaces:**
- Consumes: the classification categories from the design discussion — A (already atomic, just needs `index_cue`), B (atomic but verbose, compact to a decision fingerprint), C (mixed scenarios, needs splitting), D (historical detail, human disposition needed).
- Produces: `MIGRATION-INVENTORY.md`, a table consumed by Task 10 (index_cue drafting) and Task 11 (splitting).

- [ ] **Step 1: List every entry file**

```bash
find /Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha -type f -name "*.md" ! -name "index.md" ! -name "overview.md" | sort
```

Expected: a list of every `lesson/*.md`, `architecture/*.md`, `topology/*.md`, `best-practice/*.md` file (rule/habit are currently empty per the existing `index.md`'s "_None yet._" markers).

- [ ] **Step 2: Read every listed file and classify**

For each file, apply the Step 0.5 test (from Task 2) retroactively: can the entry's core knowledge be stated as one "if scenario then constraint" sentence (or one current-state fact)?

- **A** (already atomic, just missing `index_cue`): e.g. `architecture/2026-05-20-unee-server-layered-architecture.md` — single design fact, no mixed scenarios.
- **B** (atomic but verbose — the many decisions all answer one design question): e.g. `architecture/2026-06-17-grpc-client-resilience-design.md` — multiple decisions, but all answer "where does each gRPC resilience concern live".
- **C** (mixed scenarios, must split): confirmed example — `lesson/2026-05-17-bump-shared-ecr-must-sync-both-argocd-apps.md` mixes (i) paired web+scheduler deployment release discipline (unconditional) and (ii) ArgoCD kustomize image-key format (conditional on CD mechanism).
- **D** (historical detail, disposition unclear): candidates include specific bad-tag identifiers, the alpine-sidecar false-alarm note, and the stale-staging-probe note currently living inside the same C-class entry's body.

- [ ] **Step 3: Write the inventory table**

Write `MIGRATION-INVENTORY.md` with this exact structure (fill in every row from Step 2 — do not leave any entry unclassified):

```markdown
# unee/.claude/iris-gotcha — index_cue migration inventory

Scratch review artifact for the index_cue compact migration (2026-07-15). Not a canonical entry, not wired into any CLAUDE.md import. Delete after Task 12 completes.

| Entry path | Class | Notes |
|---|---|---|
| `lesson/2026-05-17-bump-shared-ecr-must-sync-both-argocd-apps.md` | C | Mixed: paired-deployment release discipline (unconditional) + ArgoCD image-key format (conditional on CD=ArgoCD). See Task 11. |
| `architecture/2026-05-20-unee-server-layered-architecture.md` | A | Single design fact — DDD layering. |
| `architecture/2026-06-17-grpc-client-resilience-design.md` | B | Multiple decisions, one design question (where each gRPC resilience concern lives). |
| ... (continue for every file found in Step 1) | | |
```

- [ ] **Step 4: No commit** — `unee/.claude/iris-gotcha/` is inside `/Users/daniel/Development/Code/mission-ai/unee/`, confirmed **not a git repository** during design discussion. Skip git steps for this and all remaining Part B tasks; the file write itself is the deliverable. Report the completed inventory table to the user before proceeding to Task 10.

---

### Task 10: Draft `index_cue` for every A/B-class entry (proposal only, no file writes yet)

**Files:**
- Modify: `MIGRATION-INVENTORY.md` (append a `Proposed index_cue` column)
- Read-only: every A/B-class entry file identified in Task 9

**Interfaces:**
- Consumes: `MIGRATION-INVENTORY.md` from Task 9.
- Produces: an updated inventory table with proposed `index_cue` text per A/B entry, which the user reviews before Task 11 writes anything.

- [ ] **Step 1: For each A/B-class row, draft an `index_cue` per the Task-1 rules**

Apply Step 6.5's self-check (Task 4) to each draft: scenario+constraint or shortest current-state fact, under the 100-code-point hard budget.

Example drafts (from entries read during the original design discussion):

```yaml
# architecture/2026-05-20-unee-server-layered-architecture.md
index_cue: "unee-server follows a DDD 8-layer architecture with enforced module boundaries"

# architecture/2026-06-17-grpc-client-resilience-design.md
index_cue: "gRPC resilience: deadline + default-deny retry live in the contract client; circuit-breaking stays in the mesh"
```

- [ ] **Step 2: Append the proposed cues as a new column in `MIGRATION-INVENTORY.md`**

```markdown
| Entry path | Class | Notes | Proposed index_cue |
|---|---|---|---|
| `architecture/2026-05-20-unee-server-layered-architecture.md` | A | Single design fact. | "unee-server follows a DDD 8-layer architecture with enforced module boundaries" |
| `architecture/2026-06-17-grpc-client-resilience-design.md` | B | Multiple decisions, one design question. | "gRPC resilience: deadline + default-deny retry live in the contract client; circuit-breaking stays in the mesh" |
| ... | | | |
```

- [ ] **Step 3: Present the updated table to the user for review**

Do not proceed to writing `index_cue` into the actual entry frontmatter until the user has reviewed and confirmed (or edited) the proposed cues — per the Global Constraints, this content becomes permanently injected into every future session, so it warrants a review gate even though it's "just" a compact.

---

### Task 11: Split C-class entries and settle D-class historical detail (requires user confirmation before writing)

**Files:**
- Modify: `lesson/2026-05-17-bump-shared-ecr-must-sync-both-argocd-apps.md` (rewritten as the unconditional paired-deployment entry, OR split into two files — final shape depends on user's Task-9/10 review)
- Create: a new file for the ArgoCD-conditional half (exact path/slug TBD by the user during review — this task cannot pre-decide the filename because Step 2 of Capture requires confirming project root and slug at write time, and because the user may prefer a different split boundary than the one identified during design discussion)
- Modify: any other C-class entries surfaced in Task 9

**Interfaces:**
- Consumes: the confirmed split decision from the user (after Task 9/10 review), the `## Related` cross-reference format (existing SKILL.md Step 7 subsection, unchanged by this plan).
- Produces: two (or more) atomic entries replacing each C-class entry, each individually passing the Step 0.5 test.

- [ ] **Step 1: Confirm with the user before any write**

Present the specific split proposed during design discussion for `lesson/2026-05-17-bump-shared-ecr-must-sync-both-argocd-apps.md`:

- Entry 1 (unconditional): "if a service has paired web + scheduler deployments, they must release together and their post-release versions/managed-key sets must be verified to match" — keeps the existing file's violation history (4 incidents, `zero-tolerance`, `violation_count: 4`).
- Entry 2 (conditional): "if the CD mechanism is ArgoCD kustomize image override, the image key must be the full ECR URI, and the resulting override + Deployment image must be verified" — new file, references the fact that unee v2 is *currently* on manual rollout (a separate `architecture`/`topology` fact, not yet captured — flag as a possible Task 9 D-class-turned-new-A-class entry: "v2 is currently deployed via manual rollout; ArgoCD is installed but not yet the CD mechanism").

Ask the user explicitly: does this split match their intent, and should the ArgoCD-conditional entry be kept (it has future value if ArgoCD is ever enabled) or retired to git history / a runbook? This is the exact question the design discussion flagged as a project-fact judgment only the user can make.

- [ ] **Step 2: On confirmation, write the split entries**

Follow the existing Capture Step 7 procedure (unchanged by this plan) for each resulting entry: correct frontmatter (including the new `index_cue` per Task 1), `## Related` cross-references between the two split entries, and — if the user confirms the manual-rollout fact is worth capturing — a new `architecture` entry for it.

- [ ] **Step 3: Repeat Step 1–2 for every other C-class entry from Task 9's inventory**

- [ ] **Step 4: Resolve every D-class row**

For each D-class item (specific bad tags, alpine-sidecar note, stale-staging-probe note), ask the user which disposition applies: keep in the surviving entry's body as evidence, move to a non-notebook document (runbook/incident doc), or rely on git history alone. Record the decision in `MIGRATION-INVENTORY.md`'s Notes column — do not decide silently.

- [ ] **Step 5: No commit** (non-git directory, per Task 9 Step 4) — report the full set of file changes to the user.

---

### Task 12: Regenerate `unee/.claude/iris-gotcha/index.md` and validate against budget

**Files:**
- Modify: `/Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha/index.md`
- Delete: `MIGRATION-INVENTORY.md` (scratch artifact, no longer needed once entries are settled)

**Interfaces:**
- Consumes: the finalized entry set from Task 11 (every entry now has a confirmed `index_cue`), the Step 8 regeneration algorithm from Task 5.

- [ ] **Step 1: Confirm every remaining entry has an `index_cue`**

```bash
for f in $(find /Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha -type f -name "*.md" ! -name "index.md" ! -name "overview.md" ! -name "MIGRATION-INVENTORY.md"); do
  grep -q "^index_cue:" "$f" || echo "MISSING index_cue: $f"
done
```

Expected: no output (every entry has the field). If any file is missing it, return to Task 10 for that entry before continuing.

- [ ] **Step 2: Apply the Step 8 algorithm (Task 5) by hand**

Read every entry's `index_cue` / `keywords` / `type` / `severity` / `last_violated` / path, and render `index.md` per the mechanical format specified in Task 5 Step 2. Do not consult the old `index.md` content at all while doing this.

- [ ] **Step 3: Validate against the hard budget**

```bash
wc -c /Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha/index.md
```

Expected: under 32,000 (hard budget). If over, identify which entries pushed it over and return to Task 10/11 to shorten or split them further — do not truncate the generated file by hand.

- [ ] **Step 4: Validate no entry appears twice**

```bash
grep -o '`\./\.claude/iris-gotcha/[a-z-]*/[0-9-]*[a-z-]*\.md`' /Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha/index.md | sort | uniq -d
```

Expected: no output (no path appears more than once — this is the exact defect that produced the original 57.8k file, where entries appeared in both `Recently strengthened` and their category section).

- [ ] **Step 5: Validate `Recently strengthened` eligibility**

```bash
sed -n '/^## ⚠️ Recently strengthened/,/^## [a-z]/p' /Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha/index.md | grep -c "severity:"
```

Manually cross-check each severity-bearing line in that section against its entry's `last_violated` (must be within 7 days of today). Confirm no `architecture`/`topology` line (no `severity:` field) appears in that section at all.

- [ ] **Step 6: Report before/after**

```bash
echo "Before: 57838 chars (original, per initial diagnosis)"
wc -c /Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha/index.md
```

Report the final character count, confirm it's under the 32,000 hard budget (ideally under the 24,000 soft budget), and confirm entry-count parity (every entry from Task 9's inventory, post-split, is represented exactly once).

- [ ] **Step 7: Remove the scratch inventory file**

```bash
rm /Users/daniel/Development/Code/mission-ai/unee/.claude/iris-gotcha/MIGRATION-INVENTORY.md
```

- [ ] **Step 8: No commit** (non-git directory) — final report to the user closes out the plan.

---

## Self-Review

**1. Spec coverage** — checked against the design spec's six sections:
- §3.1 `index_cue` field → Task 1.
- §3.2 atomicity judgment + strengthen four-state table → Task 2 (0.5/0.6 gates), Task 3 (Step 5 table).
- §3.3 Recent-region resolution → Task 5 (placement rule).
- §3.4 pure-function regeneration + budgets → Task 5.
- §3.5 capture gates (0.5/0.6/6.5) → Tasks 2 and 4.
- §3.6 audit's three new checks → Task 6.
- §4 migration phases (Inventory / Projection / Boundary / Regenerate / Validate) → Tasks 9–12.
- Version bump + CLAUDE.md/README housekeeping (repo's own convention, not explicit in the design spec but required by the repo's `CLAUDE.md` "Version bump checklist") → Task 8.

No spec section is without a corresponding task.

**2. Placeholder scan** — no TBD/TODO in any step; every insertion shows complete markdown text, not a description of text. Task 11's file path for the new ArgoCD-conditional entry is explicitly left for the user to confirm at execution time (project root / slug), which is a deliberate deferral matching the *existing* Capture Step 2 procedure's own "ask the user" fallback — not a plan-writing placeholder.

**3. Type consistency** — the field name `index_cue` and the four budget numbers (60/100, 4/5, 24/40, 160/220, 24000/32000) are copied verbatim across Tasks 1, 2, 4, 5, 6, and 8; no task introduces a differing number or a differently-spelled field name.

---

Plan complete and saved to `docs/superpowers/plans/2026-07-15-index-cue-compact.md`. Two execution options:

**1. Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?**
