# Iris Gotcha — 7 Category Definitions

> **Canonical taxonomy for all iris-gotcha entries.** Read this file before classifying any new entry. Do not rely on memory: definitions evolve.

Every entry in iris-gotcha is exactly **one** of seven categories. Categories are designed to be mutually exclusive when used with the disambiguation tests below. If an entry feels like it could be two categories at once, **stop and rewrite or split**.

The `type:` frontmatter field on every entry uses the **English identifier** (the canonical machine-readable name). Chinese names are kept as glosses for context only.

## Shape map

| Shape | Categories | Tense / mood |
|---|---|---|
| **Narrative** (past) | `experience` (经验), `lesson` (教训) | "I observed X happened" |
| **Prescriptive** (imperative) | `rule` (规则), `habit` (习惯), `best-practice` (最佳实践) | "from now on, do Y" |
| **Descriptive** (factual) | `architecture` (架构), `topology` (拓扑) | "this system is Z" |

`lesson` sits on the boundary: it is a narrative event that **also** carries a prescription.

---

## 1. `experience` (经验) — Narrative

**Essence**: a faithful, descriptive record of a single past episode. Pure narration. **No prescription.**

✅ Good
- "Spent 3 days integrating Sentry into Iris Island, most of it on SDK version vs project Node version incompatibility"
- "Discovered OAuth redirect_uri is case-sensitive while testing the flow"

❌ Wrong
- "Sentry SDK integration should verify version first" — this is a **`lesson`**, it has prescriptive content
- "OAuth redirect_uri must match in case" — this is a **`lesson`** or **`rule`**

**Disambiguation tests**
- **vs `lesson`**: does the entry contain any "from now on / should / must / never again / next time" phrasing? If yes → it's `lesson`, not `experience`.
- **vs `best-practice`**: is the entry tied to a specific time/place/incident? If yes → `experience`. If it generalizes to any project in this class → `best-practice`.

---

## 2. `lesson` (教训) — Narrative + Prescription (the "gotcha")

**Essence**: a specific past mistake **plus** a concrete corrective rule. Must have **both** parts. This is the old "gotcha" category.

✅ Good
- "Bun on macOS needs sudo to install global packages; use bunx instead of `bun install -g`"
- "PostgreSQL 14 BRIN index doesn't work on NULL values; ALTER COLUMN SET NOT NULL before creating the index"

❌ Wrong
- "Never run as root in production" — no specific incident → **`rule`**
- "Prefer bunx" — no incident, no reason → **`habit`** or **`best-practice`**

**Disambiguation tests**
- **vs `rule`**: does the entry cite a specific incident as its source? If yes → `lesson`. If the authority is external (policy/security/user mandate) with no specific incident → `rule`.
- **vs `experience`**: does the entry tell you what to do next time? If yes → `lesson`.
- **vs `best-practice`**: is the prescription derived from one specific incident? If yes → `lesson`. If from multiple incidents or industry consensus → `best-practice`.

---

## 3. `rule` (规则) — Prescription (MUST)

**Essence**: a non-negotiable command. Authority comes from outside the engineer's discretion: user mandate, security/compliance, project policy, legal. Violation = real problem.

✅ Good
- "Never add AI attribution to commit messages"
- "API endpoints must go through the JWT auth middleware"
- "Secrets must not appear in code or logs"

❌ Wrong
- "Our team usually uses tabs for indent" — soft preference → **`habit`**
- "REST APIs should use nouns for resources" — external recommendation → **`best-practice`**

**Disambiguation tests**
- **vs `habit`**: would violation cause an actual problem (security incident, compliance failure, user blow-up)? If yes → `rule`. If just mild annoyance → `habit`.
- **vs `best-practice`**: is this MUST or SHOULD? MUST = `rule`, SHOULD = `best-practice`.
- **vs `lesson`**: is there a specific incident behind it? If yes → `lesson`. Rules can be a priori, lessons cannot.

---

## 4. `architecture` (架构) — Descriptive (intent)

**Essence**: a high-level description of how a system is **designed**. Talks about component responsibilities, interaction style, and design philosophy. Answers *"why is it shaped this way?"*

✅ Good
- "Iris Island uses the sidecar pattern: CLI talks to daemon over local socket, daemon handles cross-device sync"
- "Auth service is separated from business services; auth issues short-lived JWTs"

❌ Wrong
- "auth-server runs on :8080, db is postgres on :5432" — measurable facts → **`topology`**
- "New services must extend BaseService" — imperative → **`rule`** or **`best-practice`**

**Disambiguation tests**
- **vs `topology`**: would changing the answer change the design intent? If yes → `architecture`. If it's just an address/port/path change → `topology`.
- **vs `rule`**: is the sentence imperative ("should / must / never")? If yes → `rule`, not `architecture`.

---

## 5. `topology` (拓扑) — Descriptive (location)

**Essence**: a measurable, enumerable map. Where services live, what ports/paths/endpoints exist, which env var goes where. Answers *"where is it?"*

✅ Good
- "auth-service on :8080, gateway on :3000, DB at postgres://localhost:5432/myapp"
- "Project root has packages/ with api, web, shared subprojects"
- "SENTRY_DSN env var must be set in packages/api/.env"

❌ Wrong
- "We use a microservice architecture" — high-level intent → **`architecture`**
- "We chose PostgreSQL because we need JSONB" — historical reasoning → **`experience`**

**Disambiguation tests**
- **vs `architecture`**: can you produce the answer by running a command (e.g. `ls`, `netstat`, `cat .env`)? If yes → `topology`. If it requires reading the minds of past engineers → `architecture`.

---

## 6. `habit` (习惯) — Prescription (soft preference)

**Essence**: personal or team preference. Breakable without serious consequence. No external authority.

✅ Good
- "Daniel prefers short commit messages without conventional prefix"
- "In the Iris project we tend to name constants after the channel name"

❌ Wrong
- "No AI attribution in commit messages" — user mandate → **`rule`**
- "TypeScript functions should declare explicit return types" — external recommendation → **`best-practice`**

**Disambiguation tests**
- **vs `rule`**: imagine breaking this — does anyone get hurt or yelled at? Real harm → `rule`. Just a "huh, why'd they do it that way?" → `habit`.
- **vs `best-practice`**: is the justification "I prefer it" / "we do it this way"? Then → `habit`. Is there a citable external source (data, community consensus, ergonomic study)? Then → `best-practice`.

---

## 7. `best-practice` (最佳实践) — Prescription (recommended)

**Essence**: SHOULD-strength prescription with **external justification**. Applies to a class of problems, not a single project.

✅ Good
- "Use a mutex instead of sleep-loops for race conditions"
- "REST APIs use nouns for resources and HTTP verbs for actions"
- "Database migration filenames carry a timestamp prefix"

❌ Wrong
- "Our project uses mutex for race conditions" — single-project fact → **`topology`** or **`experience`**
- "Never use sleep for race conditions" — overstated as MUST → **`rule`**

**Disambiguation tests**
- **vs `habit`**: can you cite a source (article, RFC, community consensus, benchmark)? If yes → `best-practice`. If not → `habit`.
- **vs `rule`**: is violation a real-world problem (security/compliance/breakage)? Then → `rule`. If it's just suboptimal → `best-practice`.
- **vs `lesson`**: is the prescription derived from one specific incident? Then → `lesson`. If from a class of incidents or external consensus → `best-practice`.

---

## Anti-collapse protocol

When in doubt, **prefer the more specific category**. The hierarchy of specificity:

1. `lesson` (most specific: one event + prescription)
2. `rule` (a priori MUST)
3. `best-practice` (class-level SHOULD)
4. `habit` (preference)
5. `experience` (one event, no prescription)
6. `topology` (one fact, measurable)
7. `architecture` (one design intent)

If you can't decide between two categories, the safer move is usually **split the entry into two**, one in each category, rather than picking one and losing information.

## The "why not the other category" gate

For every capture, the entry's frontmatter must include a `disambiguation` field of the form:

> "why not [next closest category]: [reason]"

If you can't fill in this field with a non-trivial reason, the entry is mis-classified. Rewrite or split.

## Directory layout

Entries are stored under the English category name (matches the `type:` field):

```
~/.claude/iris-gotcha/
├── experience/
├── lesson/
├── rule/
├── architecture/
├── topology/
├── habit/
└── best-practice/
```
