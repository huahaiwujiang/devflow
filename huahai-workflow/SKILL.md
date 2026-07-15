---
name: huahai-workflow
description: 开发工作流——从需求追问到代码发布，启动时自动检查依赖。当用户提出开发任务、新功能、要做某个特性、检查依赖、setup、环境检测、/huahai-workflow 时触发。即使用户想跳过步骤直接写代码，也应先走此流程确保方向正确。
---

# 开发工作流

> grilling + domain-modeling 管追问，to-spec 模板管设计，to-tickets 模板管拆分，tdd 管编码。

## 🧭 启动决策树（读到本 skill 后第一件事）

**前置：轻量依赖检查（<1s）**

```bash
ls ~/.claude/plugins/cache/mattpocock/mattpocock-skills/ 2>/dev/null
```
- ❌ 目录不存在或无版本子目录 → 提示安装 `claude plugin install mattpocock-skills@mattpocock`，安装后 `/reload-skills`，重新触发工作流
- ✅ 就绪 → 继续

```
读 todolist.md（始终在项目根目录）
├─ 不存在 ──→ 1. 问用户：「设计文档输出到哪个目录？」（默认 docs/grillme/）
│             2. 创建 todolist.md，写入顶部元信息（doc_root + 阶段: 步骤1）
│             3. 从步骤1开始
├─ 存在 + 用户说"从头开始"/"重新来"/"新任务" ──→ 清空 todolist.md 全部内容，重新走流程
├─ 存在 + 有未勾选任务 ──→ 跳到对应步骤继续
└─ 存在 + 全部已勾选
   ├─ 已发布（git log origin/main..HEAD 为空，无未推送提交）──→ 删除 todolist.md，问"新功能？"→ 重新走启动流程
   └─ 未发布 ──→ 提示用户先执行步骤5发布 + 步骤6归档
```

## 工作流链路

每一步都有明确的 **输入 → Skill → 自然产出 → 完成标志**。Skill 按其自身流程运行，工作流只做编排，不截断输出格式。

todolist.md 在启动时创建（仅含 doc_root + 阶段），随步骤推进逐步填充元信息和任务清单。

```
步骤1: 追问（WHAT — 用户要什么，领域模型是什么）
  执行: Skill("mattpocock-skills:grilling") 为主驱动追问（decision tree，一次一问，每问附推荐答案）；Skill("mattpocock-skills:domain-modeling") 在追问过程中伴随记录术语表/ADR
  说明: grill-with-docs 有 disable-model-invocation，但其内容即 "grilling + domain-modeling"，直接调用底层 skill 等价
  输入: 用户提供的需求文档/描述
  产出:
    - 项目根目录/CONTEXT.md（领域术语表，项目级 ubiquitous language，跨任务累积。若已存在则追加/更新，不覆盖）
    - <doc_root>/adr/NNNN-<slug>.md（架构决策记录，仅在满足三条件时创建：难逆转、缺上下文会惊讶、真实权衡结果）
  规则:
    - 关注业务 WHAT，不涉及技术 HOW（API 路径、表结构、技术栈选型）
    - 文档不足时可读代码补充理解，但追问输出为纯业务视角
  兜底: 若上述 Skill 不可用 → 引导用户运行 `claude plugin install mattpocock-skills@mattpocock`
  完成: grilling + domain-modeling 流程结束 → 总结领域模型关键点（核心术语、边界上下文），用户确认理解无误后，更新 todolist.md：
    1. 写入 `<!-- context: CONTEXT.md -->`（根目录，如果创建/更新了）
    2. 更新阶段为「步骤2」

步骤2: 设计（HOW — 怎么实现）
  执行: Skill("mattpocock-skills:to-spec")
  失败执行: 直接 Read ~/.claude/plugins/cache/mattpocock/mattpocock-skills/<最新版本>/skills/engineering/to-spec/SKILL.md，遵循其 process（explore → seams 确认 → 按 spec-template 写 spec）
  输入: 步骤1的 CONTEXT.md（根目录）+ adr/ + 项目现有代码
  偏离: 跳过「publish to issue tracker」等 tracker 相关步骤 —— spec 写入 <doc_root>/specs/YYYY-MM-DD-<feature>.md 即可
  完成: 🔴 CHECKPOINT — 展示设计文档，等待用户明确确认后，更新 todolist.md：
    1. 写入 `<!-- spec: <设计文档路径> -->`
    2. 更新阶段为「步骤3」

步骤3: 拆分（切成垂直切片）
  执行: Skill("mattpocock-skills:to-tickets")
  失败执行: 直接 Read ~/.claude/plugins/cache/mattpocock/mattpocock-skills/<最新版本>/skills/engineering/to-tickets/SKILL.md，遵循其 process（gather → explore → draft vertical slices → quiz user）
  输入: todolist.md 元信息 `<!-- spec: -->` 指向的设计文档 + 代码库
  偏离: 跳过「publish tickets to tracker」——用其 local-ticket-template，票写入 <doc_root>/tickets/<NN>-<slug>.md
  完成: 🔴 CHECKPOINT — 展示票列表（含阻塞关系），用户确认拆分粒度后，更新 todolist.md：
    1. 写入 `<!-- tickets_root: <doc_root>/tickets/ -->`
    2. 从票文件中提取验收条件摘要，写入 `- [ ] <票标题>`
    3. 更新阶段为「步骤4」

步骤4: 编码（逐票实现）
  执行: Skill("mattpocock-skills:tdd")
  输入: todolist.md 任务清单 + <doc_root>/tickets/ 中的票文件
  每个票的流程:
    1. 确认测试缝（seams）— 与用户确认在哪个公共接口层测试
    2. Skill("mattpocock-skills:tdd") — 先写失败测试 → 最小实现让测试通过 → 循环
    3. 单票实现完成后，勾选 todolist.md 中对应任务
  规则:
    - 🚫 禁止 commit。只写代码和测试，提交统一由步骤5执行
    - 测试只验证外部行为，不测试实现细节
    - 工作顺序遵循票依赖图（blocked-by 关系），先做无依赖票
    - 频繁类型检查，单文件测试循环，全量测试最后过一遍
  说明: 不用 Skill("implement")——implement 有 disable-model-invocation + 内嵌 commit/review，改调底层 tdd 原语
  注意: tdd 启动时会读 CONTEXT.md（步骤1产出，在项目根目录）对齐术语与接口词汇
  完成: 所有票 ✅，测试全绿，票依赖图所有边已满足，变更待提交。提示用户「如需代码审查可手动运行 /code-review」（可选，不阻塞发布）

步骤5: 发布（git flow）
  执行: Skill("huahai-workflow-gf")
  输入: 步骤4积累的全部未提交变更
  完成: 代码已提交 + 推送

步骤6: 归档
  输入: todolist.md 元信息 `<!-- spec: -->`、`<!-- tickets_root: -->`、`<!-- context: -->`
  动作:
    1. 创建 <doc_root>/archive/YYYY-MM-DD-<feature>/
    2. 将 adr/、spec 文档、tickets 目录全部移入归档目录
    3. CONTEXT.md 留在项目根目录（跨任务累积，不归档）
    4. 删除 todolist.md
  完成: 设计产物已归档，临时文件已清理，CONTEXT.md 保留供下次任务复用
```

## todolist.md 规约

- **位置**：始终在项目根目录（不在 <doc_root>/ 下）
- **定位**：进度指针，不重复计划内容。真正的任务细节在票文件中
- **生命周期**：

  | 阶段 | 操作 | 内容 |
  |------|------|------|
  | 启动 | 创建 | `<!-- doc_root: -->` + `<!-- 阶段: 步骤1 -->` |
  | 步骤1 完成 | 更新 | `<!-- context: CONTEXT.md -->` (根目录，如果创建/更新了) + 阶段 → 步骤2 |
  | 步骤2 完成 | 更新 | `<!-- spec: -->` + 阶段 → 步骤3 |
  | 步骤3 完成 | 更新 | `<!-- tickets_root: -->` + 票清单 + 阶段 → 步骤4 |
  | 步骤4 | 勾选 | 逐个 `[x]` 完成票 |
  | 步骤5 | 发布 | gf 提交+推送 |
  | 步骤6 完成 | 删除 | 归档后删除 todolist.md（CONTEXT.md 留根目录保留） |

- **顶部元信息**：
  ```
  <!-- doc_root: <绝对或相对路径> -->
  <!-- context: CONTEXT.md -->
  <!-- spec: <设计文档路径> -->
  <!-- tickets_root: <票目录路径> -->
  <!-- 阶段: <当前步骤> -->
  ```
- **格式**：每行 `- [ ] 任务描述`（步骤3从票文件中提取摘要），已完成改为 `- [x]`
- **gitignore**：应加入 `.gitignore`
- **损坏处理**：格式无法解析时，按以下优先级推断当前进度：
  1. `<doc_root>/tickets/` 有文件 → 步骤4+
  2. `<doc_root>/specs/` 有文件 → 步骤3+
  3. `<doc_root>/adr/` 有文件，或项目根目录有 CONTEXT.md → 步骤2+
  4. 以上均无 → 步骤1
  推断后从该步骤重新开始，重建 todolist.md

## 文档目录约定

工作流产出分两处：项目根目录放跨任务的 CONTEXT.md + todolist.md，其余单任务产物统一在 `<doc_root>/`（默认 `docs/grillme/`）下。

```
<doc_root>/
├── adr/                                # 步骤1: 架构决策记录
│   └── NNNN-<slug>.md
├── specs/                              # 步骤2: 设计文档
│   └── YYYY-MM-DD-<feature>.md
├── tickets/                            # 步骤3: 票文件
│   ├── 01-<slug>.md
│   └── ...
└── archive/                            # 步骤6: 归档（仅 adr/spec/tickets）
    └── YYYY-MM-DD-<feature>/
        ├── adr/
        ├── spec.md
        └── tickets/
```

项目根目录的 CONTEXT.md（步骤1产出，项目级 ubiquitous language）和 todolist.md（进度指针）不在 <doc_root>/ 下。

## 用户绕路处理

用户可能说"直接写代码"、"跳过设计"、"这步不用做了"。处理原则：

| 用户意图 | 处理 |
|---------|------|
| "跳过步骤X" | 说明该步为什么不可跳过（一句话），若用户坚持则记录到 todolist.md 顶部注释 |
| "这步已经做过了" | 验证产物是否存在（如 spec/tickets 文件），存在即可跳过 |
| "换个方案" | 回到步骤2设计，重新产生 spec |
| "需求变了" | 回到步骤1追问，追加 todolist.md 顶部注释说明变更 |
| 步骤中被打断 | 下次会话通过 todolist.md 恢复，从当前步骤继续 |

## 失败处理

| 步骤 | 触发条件 | 一线修复 | 仍失败兜底 |
|------|---------|---------|-----------|
| 1. 追问 | 需求模糊 | grilling 自有多轮追问机制 | 超过3轮无结论 → 列出待澄清项让用户逐条确认 |
| 1. 追问 | grilling 或 domain-modeling 不可用 | 引导安装 mattpocock-skills（见启动依赖检查） | — |
| 2. 设计 | 方案分歧大 | 列出方案对比，推荐默认方案 | 用户不选 → 采用最简单方案，记录理由到 todolist.md |
| 3. 拆分 | 票粒度过大 | 进一步拆分，每票不超过单次上下文窗口 | 仍过大 → 标记需人工拆分 |
| 3. 拆分 | 阻塞关系复杂 | 简化依赖图，合并可并行的票 | 展示完整依赖图让用户确认 |
| 4. 编码 | 测试写不出 | tdd 遵循 red-green 循环，先写最小实现 | 仍卡住 → 标记该票需人工处理，继续下一票 |
| 4. 编码 | tdd 不可用 | AI 手动遵循 tdd 模式 | 票不可跳过 → 标记暂停，等待用户修复依赖 |
| 5. 发布 | 合并冲突 | 显示冲突文件，等待用户手动解决 | 解决失败 → `git merge --abort`，记录冲突到 todolist.md |
| 6. 归档 | 归档目录已存在 | 追加序号 `-2`、`-3` | 仍冲突 → 手动指定归档名 |
