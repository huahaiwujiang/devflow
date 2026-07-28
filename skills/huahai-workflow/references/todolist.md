# todolist.md 规约

- **位置**：项目根（不在 doc_root）
- **定位**：进度指针；任务细节只在票文件
- **gitignore**：`todolist.md` 不入库；`<doc_root>/specs/`、`<doc_root>/tickets/` 应入库

## 初始形态（步骤1 开始时唯一合法内容）

```markdown
<!-- project_root: . -->
<!-- doc_root: docs/grillme -->
<!-- 阶段: 步骤1 -->
```

**禁止**在步骤1 出现：`context:`、`spec:`、`tickets_root:`、`- [ ]`、`- [x]`。

## 生命周期

| 阶段 | todolist 变更 | 磁盘必有 |
|------|--------------|----------|
| 启动 | `project_root` + `doc_root` + `阶段:步骤1` | doc_root 四子目录 |
| 步骤1 完成 | + `context:` + 阶段→2（**自动，不问**） | CONTEXT.md |
| 步骤2 完成 | + `spec:` + 阶段→3（**自动串联步骤3，不问**） | specs/*.md |
| 步骤3 产物就绪 | + `tickets_root:` + `- [ ]`；**阶段仍为步骤3** | tickets/*.md |
| 编码前 CHECKPOINT 通过 | 阶段→4 | （同上） |
| 步骤4 | 逐票 `[x]`；全部勾完仍停在步骤4 | 代码变更（未 commit，除非用户已手动 gf） |
| 步骤5 | （仅用户触发发布）代码审查 | — |
| 步骤6 | （审查通过/可推/已跳过）gf | git push |
| 步骤7 | （仅用户触发）对齐 + 删除 todolist | archive/ |

## 元信息模板

```markdown
<!-- project_root: . -->
<!-- doc_root: docs/grillme -->
<!-- fast-track: ... -->           （可选）
<!-- context: CONTEXT.md -->       （相对 project_root；非 doc_root）
<!-- spec: docs/grillme/specs/YYYY-MM-DD-feature.md -->
<!-- tickets_root: docs/grillme/tickets/ -->
<!-- 阶段: 步骤N -->
<!-- 编码完成待发布 -->             （步骤4 全部 [x] 后可选）
<!-- skip: 步骤X — 用户要求 YYYY-MM-DD — 原因 -->

- [ ] 来自 01-xxx.md 的票标题
```
