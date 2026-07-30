# todolist.md 规约

- **位置**：项目根（不在 doc_root）
- **定位**：进度指针；任务细节只在票文件
- **gitignore**：`todolist.md` 不入库；`<doc_root>/specs/`、`<doc_root>/tickets/` 应入库

## 初始形态（步骤1 开始时唯一合法内容）

```markdown
<!-- project_root: . -->
<!-- doc_root: docs/grillme -->
<!-- task: feature-slug -->       （可选；仅当启动时已能确定任务身份）
<!-- 阶段: 步骤1 -->
```

`task:` 不参与阶段推断；新任务最迟在步骤1收敛时写入，旧 todolist 没有它不算损坏。**禁止**在步骤1 出现：`context:`、`spec:`、`tickets_root:`、`- [ ]`、`- [x]`。

## 生命周期

| 阶段 | todolist 变更 | 磁盘必有 | 恢复入场动作 |
|------|--------------|----------|--------------|
| 启动 | `project_root` + `doc_root` + `阶段:步骤1` | doc_root 四子目录 | 继续步骤1，不重置 |
| 步骤1 完成 | + `task:`（若缺）+ `context:` + 阶段→2（**自动，不问**） | CONTEXT.md | 从步骤2继续 |
| 步骤2 完成 | + `spec:` + 阶段→3（**自动串联步骤3，不问**） | specs/*.md | 票缺则只补拆票 |
| 步骤3 产物就绪 | + `tickets_root:` + `- [ ]`；**阶段仍为步骤3** | tickets/*.md | **直接展示编码前 CHECKPOINT** |
| 编码前 CHECKPOINT 通过 | 阶段→4 | （同上） | 从首张未完成票编码 |
| 步骤4 | 逐票 `[x]`；全部勾完仍停在步骤4 | 代码变更（未 commit，除非用户已手动 `/devflow-publish`） | 部分完成则续票；全完成且本轮无发布/归档意图则停住，否则转手动挡 |
| 步骤5 | （仅用户触发发布）代码审查 | — | 按发布意图恢复 |
| 步骤6 | （审查通过/可推/已跳过）`/devflow-publish`；提示词含归档则**先**步骤7再 git | git push（含归档路径时一次推完） | 报告发布结果 |
| 步骤7 | （仅用户触发，或与发布同句且**先于**提交）对齐 + 删 todolist；**保留**四空目录 | archive/YYYY-MM-DD-*/；adr/specs/tickets/archive 壳保留 | 无 todolist，下一次按新任务启动 |

## 元信息模板

```markdown
<!-- project_root: . -->
<!-- doc_root: docs/grillme -->
<!-- task: feature-slug -->       （可选；用于区分当前任务；旧文件缺少不算损坏）
<!-- fast-track: ... -->           （可选）
<!-- context: CONTEXT.md -->       （相对 project_root；非 doc_root）
<!-- spec: docs/grillme/specs/YYYY-MM-DD-feature.md -->
<!-- tickets_root: docs/grillme/tickets/ -->
<!-- 阶段: 步骤N -->
<!-- 编码完成待发布 -->             （步骤4 全部 [x] 后可选）
<!-- skip: 步骤X — 用户要求 YYYY-MM-DD — 原因 -->

- [ ] 来自 01-xxx.md 的票标题
```
