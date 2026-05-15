# Iris Gotcha — 6 Category Definitions

> **Canonical taxonomy for all iris-gotcha entries.** Read this file before classifying any new entry. Do not rely on memory: definitions evolve.

iris-gotcha exists to capture **training-gap knowledge** — what the AI wouldn't already know from training. Before classifying anything, you've already passed Step 0 (training-gap gate) in SKILL.md and decided this content is worth capturing. The category decision is about which kind of training-gap knowledge it is.

Every entry is exactly **one** of six categories. Categories are designed to be mutually exclusive when used with the disambiguation tests below. If an entry feels like it could be two categories at once, **stop and rewrite or split**.

The `type:` frontmatter field uses the **English identifier**. Chinese names are kept as glosses for context only.

## Shape map

| Shape | Categories | Tense / mood |
|---|---|---|
| **Behavioral** (prescriptive — should-do / must-do) | `lesson`, `rule`, `habit`, `best-practice` | "from now on, do Y" (with varying authority) |
| **Reference** (descriptive — factual) | `architecture`, `topology` | "this system is Z" |

`lesson` is special — it's behavioral but anchored to a specific narrative event. The other three behavioral types are class-level prescriptions; `lesson` is an incident-derived one.

---

## 1. `lesson` (教训) — Behavioral, incident-derived (the "gotcha")

**Essence**: a specific past mistake **plus** a concrete corrective rule, where the AI would not naturally have known to avoid the mistake. Must have **both** parts. This is the old "gotcha" category.

✅ Good
- "Bun on macOS needs sudo to install global packages; use bunx instead of `bun install -g`"
- "PostgreSQL 14 BRIN index doesn't work on NULL values; ALTER COLUMN SET NOT NULL before creating the index"
- "Prisma v7.8 dropped `new PrismaClient({ datasourceUrl })`; use driver adapters instead"

❌ Wrong
- "Never log secrets" — the AI handles this by default → **skip (Step 0)**
- "Always validate input" — generic safety the AI already enforces → **skip (Step 0)**
- "Use `bunx` instead of `bun install -g`" without the incident — generic preference → **`habit`** or **`best-practice`**
- "API endpoints must use JWT auth" — project policy, no specific incident → **`rule`**

**Disambiguation tests**

- **vs `rule`**: does the entry cite a specific incident as its source? If yes → `lesson`. If the prescription stands a-priori from project/user mandate or external authority → `rule`.
- **vs `best-practice`**: derived from one specific incident? → `lesson`. From multiple incidents or industry consensus? → `best-practice`.
- **vs `habit`**: incident + harm → `lesson`. Personal preference with no harm story → `habit`.

---

## 2. `rule` (规则) — Behavioral, project- or user-mandated MUST

**Essence**: a non-negotiable prescription that the AI **wouldn't impose by default**. Two valid sources of authority:

- **Project-specific policy** the AI couldn't know from training (security model unique to this codebase, hard architectural constraint, team convention enforced in CI).
- **User override** of an AI default the user wants different (e.g., user explicitly doesn't want AI attribution in commits, even though AI tools commonly add them).

If the AI would already enforce the rule from generic security/safety alignment, it doesn't belong here — **skip at Step 0**. A "rule" entry has to teach the AI something it didn't already know.

✅ Good
- "Never add AI attribution to commit messages" (user override of AI default)
- "All API endpoints must go through the `auth-gateway` service" (project-specific MUST)
- "Database migrations must be reviewed by data-team before merging" (project policy)
- "Secrets only via the `secrets-manager` SDK; never via env vars directly in this codebase" (project-specific MUST that overrides the generic "use env vars" pattern)

❌ Wrong (AI already enforces or already conforms)
- "Never log secrets" → skip (Step 0)
- "Sanitize user input" → skip (Step 0)
- "Use prepared statements for SQL" → skip (Step 0)
- "REST APIs should use nouns for resources" → AI default + class-level → **`best-practice`**
- "We use Tabs for indent" → mild preference → **`habit`**
- "Bun on macOS needs sudo" — has a specific incident → **`lesson`**

**Disambiguation tests**

- **vs (skip)**: is this just generic security/safety/quality the AI already enforces by default? If yes, **don't capture**.
- **vs `habit`**: would violation cause an actual problem (security breach, build break, audit failure)? If yes → `rule`. Just a "huh, why'd they do it that way?" → `habit`.
- **vs `best-practice`**: is this MUST or SHOULD? MUST = `rule`. SHOULD = `best-practice`.
- **vs `lesson`**: is there a specific incident behind it? If yes → `lesson`. Rules are a-priori or stand-alone mandates, lessons are incident-derived.

---

## 3. `habit` (习惯) — Behavioral, soft personal/team preference

**Essence**: personal or team preference. Breakable without serious consequence. No external authority.

✅ Good
- "Daniel prefers short commit messages without conventional prefix"
- "In the Iris project we tend to name constants after the channel name"
- "Daniel prefers a single TEXT column for error messages over structured enum + code + category columns"

❌ Wrong
- "No AI attribution in commit messages" — user explicit mandate → **`rule`**
- "TypeScript functions should declare explicit return types" — external recommendation → **`best-practice`**

**Disambiguation tests**

- **vs `rule`**: imagine breaking this — does anyone get hurt or yelled at? Real harm → `rule`. Just "huh" → `habit`.
- **vs `best-practice`**: is the justification "I prefer it" / "we do it this way"? Then → `habit`. Is there a citable external source (article, data, community)? Then → `best-practice`.

---

## 4. `best-practice` (最佳实践) — Behavioral, recommended

**Essence**: SHOULD-strength prescription with **external justification**. Applies to a class of problems, not a single project. **Must be something the AI wouldn't already follow by default** — if it's a generic best practice the AI handles, skip at Step 0.

✅ Good
- "When using Prisma upsert, mirror the conditional logic in both create and update branches — it's a recurring bug class"
- "ArgoCD app sync via `kubectl patch` is more reliable than `argocd app sync` when CLI tokens expire"
- "For race conditions in Bun, use Web Lock API rather than custom mutex" (only if the AI wouldn't default to this)

❌ Wrong
- "Use mutex instead of sleep-loops" → AI defaults → skip (Step 0)
- "Always validate user input" → AI defaults → skip (Step 0)
- "Our team uses Tabs" → personal/team preference → **`habit`**
- "ArgoCD failed once; use kubectl" — anchored to one incident → **`lesson`**

**Disambiguation tests**

- **vs `habit`**: can you cite a source (article, RFC, community consensus, benchmark, accumulated team evidence)? If yes → `best-practice`. If not → `habit`.
- **vs `rule`**: is violation a real-world problem? Then → `rule`. If it's just suboptimal but the alternative works → `best-practice`.
- **vs `lesson`**: is the prescription derived from one specific incident? Then → `lesson`. From a class of incidents or external consensus → `best-practice`.

---

## 5. `architecture` (架构) — Reference (design intent)

**Essence**: a high-level description of how a system is **designed**. Talks about component responsibilities, interaction style, and design philosophy. Answers *"why is it shaped this way?"*

✅ Good
- "Iris Island uses the sidecar pattern: CLI talks to daemon over local socket, daemon handles cross-device sync"
- "Auth service is separated from business services; auth issues short-lived JWTs"
- "unee-server and unee-scheduler are deliberately split into two ArgoCD apps despite sharing an image, to allow independent scheduler restart cadence"

❌ Wrong
- "auth-server runs on :8080, db is postgres on :5432" — measurable facts → **`topology`**
- "New services must extend BaseService" — imperative → **`rule`** or **`best-practice`**

**Disambiguation tests**

- **vs `topology`**: would changing the answer change the design intent? If yes → `architecture`. If it's just an address/port/path change → `topology`.
- **vs `rule`**: is the sentence imperative ("should / must / never")? If yes → `rule`, not `architecture`.

---

## 6. `topology` (拓扑) — Reference (location)

**Essence**: a measurable, enumerable map. Where services live, what ports/paths/endpoints exist, which env var goes where. Answers *"where is it?"*

✅ Good
- "auth-service on :8080, gateway on :3000, DB at postgres://localhost:5432/myapp"
- "Project root has packages/ with api, web, shared subprojects"
- "SENTRY_DSN env var must be set in packages/api/.env"
- "unee-server and unee-scheduler share the same ECR tag `sha-xxx-main` but are two distinct ArgoCD applications"

❌ Wrong
- "We use a microservice architecture" — high-level intent → **`architecture`**
- "We chose PostgreSQL because we need JSONB" — historical reasoning → skip (training-gap test: does AI need to know the historical reason?) or include in `architecture`

**Disambiguation tests**

- **vs `architecture`**: can you produce the answer by running a command (e.g. `ls`, `netstat`, `cat .env`, `kubectl get`)? If yes → `topology`. If it requires reading the minds of past engineers → `architecture`.

---

## Anti-collapse protocol

When in doubt, **prefer the more specific category**. The hierarchy of specificity:

1. `lesson` (most specific: one event + prescription)
2. `rule` (a-priori MUST, project- or user-mandated)
3. `best-practice` (class-level SHOULD with external justification)
4. `habit` (soft preference)
5. `topology` (one fact, measurable)
6. `architecture` (one design intent)

If you can't decide between two categories, the safer move is usually **split the entry into two**, one in each category, rather than picking one and losing information.

### Active split test (since v0.6.0)

Before finalizing a category, ask:

> "Does this entry contain a *descriptive fact* (architecture/topology) that would survive even if the lesson part were forgotten?"

If yes, split: capture the fact as architecture/topology, capture the prescription as lesson, cross-reference each other in the body. Real-world example: "unee-server and unee-scheduler share an ECR tag but are two ArgoCD apps; when bumping the shared image you must sync both, I missed scheduler this time" → that's a `topology` entry (the shared-tag fact) + a `lesson` entry (the missed sync, referencing the topology).

## The "why not the other category" gate

For every capture, the entry's frontmatter must include a `disambiguation` field of the form:

> "why not [next closest category]: [reason]"

If you can't fill in this field with a non-trivial reason, the entry is mis-classified. Rewrite or split.

## Directory layout

Entries are stored under the English category name (matches the `type:` field):

```
~/.claude/iris-gotcha/
├── lesson/
├── rule/
├── architecture/
├── topology/
├── habit/
└── best-practice/
```

## Note on the dropped `experience` category (v0.6.0)

Earlier versions had `experience` for pure narrative records (no prescription). v0.6.0 dropped it because:

1. Pure narrative without a behavioral takeaway doesn't change future AI behavior — it accumulates dead weight in the index.
2. Session history is already handled by `claude-mem`; iris-gotcha shouldn't duplicate.
3. In practice, almost any "experience" worth capturing has a prescription buried in it — and that makes it a `lesson` (with the corrective rule extracted) or feeds into a `best-practice` (after multiple similar episodes).

If you find yourself wanting `experience`, ask: is there a behavioral takeaway? If yes, capture as `lesson`. If no, it probably belongs in `claude-mem` or nowhere at all.
