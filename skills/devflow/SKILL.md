---
name: devflow
description: >
  无工单的新特性 / 开发任务 / 要做某个功能，或调用 /devflow 时使用（含想跳过直接写代码、检查依赖、setup）。
  不要用于 PROJ-123、明确修缺陷或 /devflow-issue；不要用于仅提交/推送（/devflow-publish）。
---

# 开发工作流

grilling + domain-modeling 管追问，to-spec 管设计，to-tickets 管拆分，tdd 管编码。步骤2→3 串联不问；**编码前一次 CHECKPOINT**；自动挡停在编码结束；发布/归档手动。

**按需读取**（勿一次全载）：
- [defaults.md](references/defaults.md) — doc_root / 目录约定
- [todolist.md](references/todolist.md) — 元信息、生命周期、初始形态
- [recovery.md](references/recovery.md) — 产物完整性、编码授权、损坏恢复、状态矩阵与自检
- [pre-push-review.md](references/pre-push-review.md) — 推送前审查（权威）
- [archive.md](references/archive.md) — 归档对齐与搬移（权威）
- 步骤1–4 模块（缺文件则手搓）：独立 skill `grilling`；本目录 [domain-modeling/SKILL.md](domain-modeling/SKILL.md)、[to-spec/SKILL.md](to-spec/SKILL.md)、[to-tickets/SKILL.md](to-tickets/SKILL.md)、[tdd/SKILL.md](tdd/SKILL.md)

---

## 核心原则

1. **可观测性**：进度真相依次为 todolist 指针、枚举所得磁盘产物、本轮实时 git 状态；会话注入的历史快照仅作提示（禁止产物不存在时写元信息或勾选）。回复默认**一行进度**；完整状态表仅门禁时刻展开（见下）。
2. **防跳步**：步骤1–3 + 编码前 CHECKPOINT 不可跳过；步骤2→3 串联不问；簿记自动落盘。
3. **发布与归档解耦**：`/devflow-publish` 不暗含归档；归档不要求已 push；同一句提示词写明才联动。**含归档时一律先归档再提交/推送**（不论用户语序）。
4. **与 issue 分流**：`PROJ-123` / 修缺陷 / `/devflow-issue` → Skill(`devflow-issue`)，本流程不做 grilling→spec→tickets。

---

## 自动挡 / 手动挡

| 挡位 | 范围 | 何时 |
|------|------|------|
| **自动挡** | 1 → 2–3 串联 → 编码前 CHECKPOINT → 4 | `/devflow`、实现需求、确认写代码 |
| **手动挡** | 5 审查 → 6 发布 → 7 归档 | 用户显式：发布/推送/`/devflow-publish`/归档；审查随发布触发 |

自动挡终点 = 步骤4 全部 `[x]` 且验证通过。若本轮没有显式发布/归档意图 → **停住**，不自动 commit/push/归档；若本轮已显式要求则转手动挡。停住时「下一步」固定：

> 编码已完成。请先自行查看变更。发布请说 `/devflow-publish` 或「发布」（默认先审查再推送，可说「跳过审查」；加「归档」则**先归档再**提交推送），仅归档请说「归档」。

---

## 进度展示

**默认（每轮强制，仅此一行，勿盖过提问）：**

`进度：步骤N — <名称> · <待确认或下一步一句话>`

例：`进度：启动 · 复用 doc_root=docs/grillme` 或（无已有目录时）`待确认 doc_root（推荐 docs/grillme）`

**禁止**在普通追问 / 确认目录轮粘贴完整状态表。

**完整表**：仅编码前 CHECKPOINT、产物完整性失败、用户跳过、步骤4完成停住、会话恢复/损坏修复时展开；模板见 [recovery.md](references/recovery.md)#完整状态表模板。步骤模块文件缺失、ADR=0、待可写补建 → 写进进度行即可。恢复轮只报告判定结果与下一步，禁止叙述内部试错过程。

---

## 硬门禁

违反则停止：禁止写业务代码、禁止推进阶段。进度只认磁盘产物 + 编码前 CHECKPOINT。

| 动作 | 前置条件 |
|------|----------|
| `<!-- 阶段: 步骤2 -->` | 步骤1 已收敛 + CONTEXT 已更新（自动，不问） |
| `<!-- spec: ... -->` | **对应 spec 文件已创建**（不问；2→3 串联） |
| `<!-- tickets_root: ... -->` | **tickets/ 含 ≥1 个 .md**（不问） |
| 写入 `- [ ]` | 票文件已创建（任务来自票标题，禁止手写） |
| `<!-- 阶段: 步骤4 -->` | 🔴 编码前 CHECKPOINT 用户已确认 |
| 勾选 `- [x]` | 票文件存在 + 验收条件满足 |
| **写业务代码** | 阶段=步骤4（编码授权已获）且产物完整性通过（见 [recovery.md](references/recovery.md)） |
| 步骤5 审查 | 步骤4 全 `[x]` + 用户显式发布/推送/`/devflow-publish` |
| 步骤6 发布 | 审查通过 / 可推 / 已跳过 |
| 步骤7 归档 | 用户显式归档（不要求步骤6 完成） |

---

## 启动决策树（读到本 skill 后第一件事）

### 0. 先读 todolist.md（唯一首动作）

一律先读项目根 `todolist.md`，再做任何依赖检查、doc_root 探测或写盘。用户明确「新任务 / 从头开始」时也先读，以便展示当前任务状态，但不按旧阶段续跑。**有合法 devflow todolist 且用户未明确新开 → 恢复分支；否则 → 新任务分支。**

文件存在但缺少 `project_root` / `doc_root` / `阶段` 任一 devflow 必需元信息 → 视为非 devflow 文件，不覆盖其内容；报告命名冲突，请用户先改名或明确允许替换，再进入新任务分支。

读取 spec / tickets 时，必须先枚举元信息指向的目录，再按枚举结果读取；禁止凭标题或序号猜文件名。读不到先复查枚举结果，不得直接宣告产物缺失。

### 1. 恢复分支（有 todolist）

先读 [recovery.md](references/recovery.md)，按 todolist 阶段与磁盘证据唯一分派：

| 阶段 | 磁盘证据 | 本回合动作 |
|------|----------|------------|
| 步骤1 | 初始形态合法，且磁盘无本任务 spec / 票 | 继续追问；不重置、不重开 |
| 步骤2 | context 存在 | 从设计继续 |
| 步骤3 | spec 存在、票 ≥1 | **正常待确认态：直接展示编码前 CHECKPOINT** |
| 步骤3 | spec 存在、票为 0 | 只补拆票，再展示 CHECKPOINT |
| 步骤4 | 有未勾选票 | 产物完整性通过后，从首张未完成票继续编码 |
| 步骤4 | 全部 `[x]`，本轮无发布/归档意图 | 停住并使用编码完成固定提示 |
| 步骤4 | 全部 `[x]`，本轮显式发布/推送/归档 | 转手动挡：发布走步骤5→6；含归档则先步骤7再提交推送 |
| 任一阶段 | 磁盘产物超前于元信息 | 只补元信息；反向推断最高到步骤3，仍须 CHECKPOINT |
| 任一阶段 | 元信息与磁盘不一致 | 从最早缺失阶段精确修复；不得重做更早的完整产物 |

有未勾选项不等于一律编码：必须服从 `<!-- 阶段 -->`。步骤3 + 票齐不是损坏，也未获编码授权。

用户本轮明确给出另一份需求 / PRD，且与当前 spec 不是同一任务 → 说明当前任务状态并让用户选择续跑或新开；不得静默覆盖 todolist。用户明确「从头开始 / 新任务」→ 进入新任务分支。

### 2. 新任务分支（无 todolist，或用户明确新开）

步骤1 用独立 skill `grilling`，领域词与步骤2–4 模块在本目录内（`domain-modeling/`、`to-spec/`、`to-tickets/`、`tdd/`），按相对路径读取；缺文件或 `grilling` 不可用不阻塞，进度行注明「手搓」。

再读 [defaults.md](references/defaults.md) 探测 doc_root。

若因用户明确「新任务 / 从头开始」进入本分支，先展示当前任务状态；用户意图已明确时不再重复确认，按初始形态重建 todolist，但不得删除或覆盖旧 spec / tickets / 代码产物。

> **回合纪律（仅新任务分支适用）**：
> - **已有** `*/docs/grillme` 或可复用 doc_root（含四子目录）→ 不问，直接复用；本回复内补齐缺失子目录 + 写初始 todolist → 紧接步骤1第一问。
> - **无已有目录、须展示选项** → 本回合只确认目录，禁止同回合抛步骤1第一问。
> - **用户已确认** doc_root → 同一回复内建四子目录、写初始 todolist并直接第一问，禁止再等「继续」。

会话恢复一律以 todolist 阶段 + 枚举所得磁盘产物为准；本轮实时 git 状态只辅助判断代码是否已改，会话开头的历史快照不得推翻前两者。

---

## 快速通道

同时满足：有 PRD / 移动端原型 / PC 已对接接口，且用户已确认范围清单 → 可压缩步骤1–3 篇幅；**不可跳过** CHECKPOINT 与产物文件。步骤1 确认后在 todolist 顶部写：

```markdown
<!-- fast-track: 用户确认 YYYY-MM-DD — \<一句话范围\> -->
```

---

## 工作流链路

```
步骤1 追问 → 步骤2 设计 ⇄ 步骤3 拆分（串联不问）
→ 🔴 编码前 CHECKPOINT → 步骤4 编码（自动挡终点）
── 手动 ── 步骤5 审查 → 步骤6 发布（`/devflow-publish`）
           步骤7 归档（可单独；与发布同句则先于提交/推送）
```

### 步骤1: 追问（WHAT）

- 执行：Skill(`grilling`) + 读并遵循 [domain-modeling/SKILL.md](domain-modeling/SKILL.md)；`grilling` 不可用则按 frontier 轮次手搓（每轮问齐当前可问的问题，各带推荐答案，等回复再下一轮）
- 伴随：CONTEXT / ADR 由 domain-modeling 落盘。ADR 仅当：难逆转 + 缺上下文会惊讶 + 真实权衡
- 产出：项目根 `CONTEXT.md`（仅业务 WHAT；禁止 API/分包/表结构；已存在则追加）。`<doc_root>/adr/` 0~N
- 推进：范围确认后自动更新 CONTEXT → `task:`（若缺）+ `context:` + 阶段→2 → **立即步骤2**
- 快速通道：1 轮范围确认即可；ADR 可为 0（进度行或门禁表说明）

### 步骤2: 设计（HOW）

- 执行：读并遵循 [to-spec/SKILL.md](to-spec/SKILL.md)；缺文件则手写 mini-spec
- 产出：**先建** `<doc_root>/specs/YYYY-MM-DD-<feature>.md`（页面/路由、API 引用、组件、不在范围；技术细节只写 spec）
- 推进：不问 → `spec:` + 阶段→3 → **立即步骤3**
- 例外停顿：方案分歧 → 对比表+推荐，选定后再串联
- 快速通道：mini-spec ≤120 行

### 步骤3: 拆分

- 执行：读并遵循 [to-tickets/SKILL.md](to-tickets/SKILL.md)；缺文件则手写票
- 产出：**先建** `<doc_root>/tickets/01-<slug>.md` …
- 簿记不问：`tickets_root:` + `- [ ]`；**阶段仍为步骤3**
- 🔴 编码前 CHECKPOINT（唯一闸）：展示 CONTEXT 要点 + spec 摘要（含不在范围）+ 票列表/依赖 → 用户确认 → 阶段→4
- 例外：票过大 → 再拆或标「需人工拆分」
- 快速通道：≤5 张票，皆有验收条件

### 步骤4: 编码（自动挡终点）

- 执行：读并遵循 [tdd/SKILL.md](tdd/SKILL.md)；缺文件则 red-green / build 验证
- 前置：CHECKPOINT 已确认（阶段=步骤4）+ 产物完整性通过（结果写入当轮完整状态表）
- 每票：确认测试缝 → 实现验证 → `[x]`（遵依赖）
- 禁止自动 commit/push；全 `[x]` 且本轮无显式发布/归档意图时停住

### 步骤5: 代码审查

读并遵循 [pre-push-review.md](references/pre-push-review.md)。验收依据 = spec / tickets。

### 步骤6: 发布

用户显式发布 / `/devflow-publish` → Skill(`devflow-publish`)。提示词同时提归档时由该 skill **先归档再**提交/推送（一次推完；不论用户语序）。

### 步骤7: 归档

用户显式「归档」（或经 `/devflow-publish` 且提示词含归档）。读并遵循 [archive.md](references/archive.md)。不要求步骤6 已单独完成。归档只搬文件，**保留** `adr/`、`specs/`、`tickets/`、`archive/` 空目录。

---

## 用户绕路

| 用户意图 | 处理 |
|---------|------|
| 「直接写代码」 | 说明门禁；坚持 → `skip` + 仍须极简 mini-spec/票或书面承担；仍停步骤4 |
| 「跳过步骤 X」 | 说明原因；坚持 → `<!-- skip: 步骤X -->` |
| 「跳过审查」 | 注明后进步骤6 |
| 「按计划实现」 | 已在 CHECKPOINT → 确认进4；否则先完成 1–3 产物 |
| 第一问答全范围 | 仍要步骤1 摘要确认；可 fast-track，不可跳文件 |
| 「这步做过了」 | **验证文件存在** 才跳过 |
| 「换个方案」 | 回步骤2 → 串联3 → 新 CHECKPOINT |
| 「需求变了」 | 回步骤1，todolist 顶部注释 |
| 会话中断 | 按启动决策树恢复分支唯一分派；禁止重跑已完成阶段 |
| 用户给出另一份 PRD / 需求 | 若与当前 spec 同任务则续跑；否则展示当前状态并让用户选择续跑或新开 |

---

## 失败处理

| 步骤 | 触发 | 修复 |
|------|------|------|
| 1 | 需求模糊 / grilling 不可用 | grilling 多轮 / 手搓，不中止 |
| 2 | 方案分歧 | 对比表+推荐；不选则最简方案记入 spec |
| 3 | 票过大 | 再拆或标人工拆分 |
| 4 | 测不出 | 记「build 验证」继续 |
| 5 | 严重问题 | 先修再审或跳过 |
| 6 | 冲突 | 展示文件；失败则 merge --abort |

损坏与自检见 [recovery.md](references/recovery.md)。
