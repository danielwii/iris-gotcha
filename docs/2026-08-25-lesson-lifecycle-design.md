# Lesson 生命周期设计（v0.13 候选，设计输入）

> 2026-08-25。状态：设计笔记，未实现。来源：mission-ai 会话对 index 膨胀
> （71 行 / 23KB，每会话必载）的诊断 + 文献调研。

## 问题

现有生命周期是**单向棘轮**：诞生 → 常驻注入 → 违规时强化（severity 升级、
violation #N、facet 追加）。只有升级器，没有降级器——没有任何一条 lesson
曾经离开过 index。三个终态缺失：

| 缺失终态 | 含义 | 现实例子 |
|---|---|---|
| **graduated** | 长期零违规 = 已内化，该降级/归档 | — |
| **obsolete** | 技术栈变更使 lesson 失效 | graphql-17-bun（上游修好 exports 即死）、prisma-v7-datasourceurl |
| **superseded** | 已升华成更强的结构化载体，散文版该塌缩成指针 | contract-generate 教训 → mission-ai `submodule-workflow.md` Step 4b（2026-08-24，ad-hoc 发生，engine 无感知，两处并存无人对账） |

根因：**只采负信号（violation），无正信号（使用/命中）**。没有
"这条 lesson 被读过没有、防住过错没有"，就永远分不清
「内化了（毕业）」和「从没用上（退役）」。

## 文献依据

- **arXiv:2605.08538**（Human-Inspired Memory Architecture）：
  lifecycle = consolidation / forgetting / **maturation** / **reconsolidation**
  四个显式阶段。maturation 依赖**重复检索信号**；reconsolidation = 被检索的
  记忆进入可修改窗口，遇矛盾当场融合，"防止过期事实无限存续"。
  机制全部为确定性算法（分数+阈值，不调 LLM）。
- **arXiv:2606.10677**（Infini Memory）：文档型记忆 = topic documents
  （有界维护单元）+ 条目级元数据（时间/来源/updatelog）跟随每次
  rewrite/split/merge。**写入高频、结构维护低频**（eager 维护引入不稳定
  编辑）。index 只做 summary 层。
- **arXiv:2508.19005**（ELL/StuLife）：skill 生命周期必须形式化
  （acquisition → validation → invocation → evolution，**含 deprecate**），
  否则"积累脆弱冗余的行为"。**Knowledge Internalization 是设计终态**：
  高频规则蒸馏进核心流程（"second nature"），减少外部检索依赖——
  Step 4b 那次升华即此过程的实例。

## 设计（按杠杆排序）

1. **补正信号（最上游，零新机制）**：条目元数据加 `last_referenced` +
   `hit_count`。skill 已有"命中 keywords 必读全文"动作，读取时顺手打点。
2. **三个终态 + 确定性预筛**：
   - `graduated`：≥6 个月零违规 且 有引用记录（内化证据）
   - `obsolete`：依赖的栈/版本已弃用
   - `superseded_by: <path>`：结构化载体已存在 → index 行塌缩为一行指针
   - 日期/计数做候选预筛（代码，便宜可审计）；去留裁决交模型
     （语义判断：栈还在用吗、载体真的覆盖了吗）。
3. **低频结构维护 pass**（对应 Infini 的 split/merge/摘要刷新）：
   手动触发（/dreaming 或 engine 自带命令），产出**新版本供 review，
   不原地改**（沿用 Anthropic Dreams 的 copy-on-write 原则）。
   顺带把 index 行瘦成真正一行（现每条数百字，是 23KB 的直接来源）。
4. **reconsolidation 补反向半边**：session 读到 lesson 发现与现实矛盾时
   **当场改写/标 stale**（正向半边=现有 strengthening 已覆盖）。

## 非目标

- 不做自动定时后台整理（无人监督的写者，已在 mission-ai 会话中否决）。
- 不为终态判定引入外部 classifier；裁决在读取现场由当前模型做。
- 不改变现有捕获/强化机制——只补退出路径。
