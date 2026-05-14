# Iris Gotcha — 7 Category Definitions

> **Canonical taxonomy for all iris-gotcha entries.** Read this file before classifying any new entry. Do not rely on memory: definitions evolve.

Every entry in iris-gotcha is exactly **one** of seven categories. Categories are designed to be mutually exclusive when used with the disambiguation tests below. If an entry feels like it could be two categories at once, **stop and rewrite or split**.

## Shape map

| Shape | Categories | Tense / mood |
|---|---|---|
| **Narrative** (past) | 经验, 教训 | "我观察到 X 发生了" |
| **Prescriptive** (imperative) | 规则, 习惯, 最佳实践 | "以后要 Y" |
| **Descriptive** (factual) | 架构, 拓扑 | "这个系统是 Z" |

教训 sits on the boundary: it is a narrative event that **also** carries a prescription.

---

## 1. 经验 (Experience) — Narrative

**Essence**: a faithful, descriptive record of a single past episode. Pure narration. **No prescription.**

✅ Good
- "在 Iris Island 集成 Sentry 用了 3 天,主要花在 SDK 版本和项目 Node 版本不兼容上"
- "测 OAuth 流程时发现 redirect_uri 大小写敏感"

❌ Wrong
- "Sentry SDK 集成应该先验证版本" — this is a **教训**, it has prescriptive content
- "OAuth redirect_uri 必须大小写一致" — this is a **教训** or **规则**

**Disambiguation tests**
- **vs 教训**: does the entry contain any "以后要 / 应该 / 必须 / 别再 / next time" phrasing? If yes → it's 教训, not 经验.
- **vs 最佳实践**: is the entry tied to a specific time/place/incident? If yes → 经验. If it generalizes to any project in this class → 最佳实践.

---

## 2. 教训 (Lesson) — Narrative + Prescription (the "gotcha")

**Essence**: a specific past mistake **plus** a concrete corrective rule. Must have **both** parts. This is the old "gotcha" category.

✅ Good
- "Bun 在 macOS 装 global package 需要 sudo;以后用 bunx 而不是 bun install -g"
- "PostgreSQL 14 的 BRIN index 在 NULL 值上不工作;建索引前先 ALTER COLUMN SET NOT NULL"

❌ Wrong
- "永远不要在生产环境用 root" — no specific incident → **规则**
- "用 Bun 优先 bunx" — no incident, no reason → **习惯** or **最佳实践**

**Disambiguation tests**
- **vs 规则**: does the entry cite a specific incident as its source? If yes → 教训. If the authority is external (policy/security/user mandate) with no specific incident → 规则.
- **vs 经验**: does the entry tell you what to do next time? If yes → 教训.
- **vs 最佳实践**: is the prescription derived from one specific incident? If yes → 教训. If from multiple incidents or industry consensus → 最佳实践.

---

## 3. 规则 (Rule) — Prescription (MUST)

**Essence**: a non-negotiable command. Authority comes from outside the engineer's discretion: user mandate, security/compliance, project policy, legal. Violation = real problem.

✅ Good
- "永远不要在 commit message 加 AI attribution"
- "API 端点必须走 JWT 中间件鉴权"
- "密钥不允许出现在代码或日志里"

❌ Wrong
- "我们团队一般用 Tabs 缩进" — soft preference → **习惯**
- "REST API 用名词作 resource" — external recommendation → **最佳实践**

**Disambiguation tests**
- **vs 习惯**: would violation cause an actual problem (security incident, compliance failure, user blow-up)? If yes → 规则. If just mild annoyance → 习惯.
- **vs 最佳实践**: is this MUST or SHOULD? MUST = 规则, SHOULD = 最佳实践.
- **vs 教训**: is there a specific incident behind it? If yes → 教训. Rules can be a priori, lessons cannot.

---

## 4. 架构 (Architecture) — Descriptive (intent)

**Essence**: a high-level description of how a system is **designed**. Talks about component responsibilities, interaction style, and design philosophy. Answers *"why is it shaped this way?"*

✅ Good
- "Iris Island 用 sidecar 模式:CLI 通过本地 socket 与 daemon 通信,daemon 负责跨设备同步"
- "认证服务与业务服务分离,认证服务用 JWT 颁发短期令牌"

❌ Wrong
- "auth-server 跑在 :8080, db 是 postgres 跑在 :5432" — measurable facts → **拓扑**
- "新增服务时必须继承 BaseService" — imperative → **规则** or **最佳实践**

**Disambiguation tests**
- **vs 拓扑**: would changing the answer change the design intent? If yes → 架构. If it's just an address/port/path change → 拓扑.
- **vs 规则**: is the sentence imperative ("应该 / 必须 / 不要")? If yes → 规则, not 架构.

---

## 5. 拓扑 (Topology) — Descriptive (location)

**Essence**: a measurable, enumerable map. Where services live, what ports/paths/endpoints exist, which env var goes where. Answers *"where is it?"*

✅ Good
- "auth-service 在 :8080, gateway 在 :3000, DB 在 postgres://localhost:5432/myapp"
- "项目根目录下 packages/ 有 api、web、shared 三个子项目"
- "环境变量 SENTRY_DSN 必须在 packages/api/.env 里"

❌ Wrong
- "服务之间是微服务架构" — high-level intent → **架构**
- "我们选 PostgreSQL 是因为要 JSONB" — historical reasoning → **经验**

**Disambiguation tests**
- **vs 架构**: can you produce the answer by running a command (e.g. `ls`, `netstat`, `cat .env`)? If yes → 拓扑. If it requires reading minds of past engineers → 架构.

---

## 6. 习惯 (Habit) — Prescription (soft preference)

**Essence**: personal or team preference. Breakable without serious consequence. No external authority.

✅ Good
- "Daniel 喜欢简短的 commit message,不加 conventional prefix"
- "在 Iris 项目我们倾向用 channel name 命名常量"

❌ Wrong
- "commit message 不要 AI attribution" — user mandate → **规则**
- "TypeScript 函数推荐显式标返回类型" — external recommendation → **最佳实践**

**Disambiguation tests**
- **vs 规则**: imagine breaking this — does anyone get hurt or yelled at? Real harm → 规则. Just a "huh, why'd they do it that way?" → 习惯.
- **vs 最佳实践**: is the justification "I prefer it" / "we do it this way"? Then → 习惯. Is there a citable external source (data, community consensus, ergonomic study)? Then → 最佳实践.

---

## 7. 最佳实践 (Best Practice) — Prescription (recommended)

**Essence**: SHOULD-strength prescription with **external justification**. Applies to a class of problems, not a single project.

✅ Good
- "处理 race condition 用 mutex 而不是 sleep 等待"
- "REST API 用名词作 resource,动词作 HTTP method"
- "数据库迁移文件命名带时间戳前缀"

❌ Wrong
- "我们项目用 mutex 处理 race" — single-project fact → **拓扑** or **经验**
- "永远不要用 sleep 处理 race" — overstated as MUST → **规则**

**Disambiguation tests**
- **vs 习惯**: can you cite a source (article, RFC, community consensus, benchmark)? If yes → 最佳实践. If not → 习惯.
- **vs 规则**: is violation a real-world problem (security/compliance/breakage)? Then → 规则. If it's just suboptimal → 最佳实践.
- **vs 教训**: is the prescription derived from one specific incident? Then → 教训. If from a class of incidents or external consensus → 最佳实践.

---

## Anti-collapse protocol

When in doubt, **prefer the more specific category**. The hierarchy of specificity:

1. **教训** (most specific: one event + prescription)
2. **规则** (a priori MUST)
3. **最佳实践** (class-level SHOULD)
4. **习惯** (preference)
5. **经验** (one event, no prescription)
6. **拓扑** (one fact, measurable)
7. **架构** (one design intent)

If you can't decide between two categories, the safer move is usually **split the entry into two**, one in each category, rather than picking one and losing information.

## The "why not the other category" gate

For every capture, the entry's frontmatter must include a `disambiguation` field of the form:

> "why not [next closest category]: [reason]"

If you can't fill in this field with a non-trivial reason, the entry is mis-classified. Rewrite or split.
