# Integrity、损坏恢复与自检

## 完整状态表模板

仅门禁时刻使用（见主 skill「进度展示」）。**不要**在普通追问轮输出。

```markdown
---
## Workflow 状态

| 项 | 内容 |
|----|------|
| 当前步骤 | 步骤 N — \<名称\> |
| doc_root | \<路径\> |
| 本步产出 | \<文件路径列表，无则「无」\> |
| ADR | \<路径或无 — 无时写原因\> |
| 已跳过 | \<步骤 + 原因；无则「无」\> |
| 待你确认 | \<CHECKPOINT / 例外停顿；无则「无」\> |
| 下一步 | \<具体动作；步骤4 完成后用主 skill 固定提示\> |
---
```

## Integrity 校验（步骤4 编码前必跑）

读取 todolist.md 元信息，**逐项验证文件存在**，在**完整状态表**写「Integrity：通过」或列出失败项：

```
project_root 目录存在（CONTEXT.md / todolist.md 在此，不在 doc_root）
doc_root 目录存在
spec 元信息指向的文件存在
tickets_root 目录存在且含 ≥1 个 .md
阶段 ≥ 步骤4（即编码前 CHECKPOINT 已通过）
```

任一失败 → **停止编码**，报告损坏项，从缺失步骤重做。禁止「元信息先写、文件后补」。若票已齐但阶段仍为步骤3 → 不是损坏，应展示编码前 CHECKPOINT。

## 损坏处理（todolist 与磁盘不一致）

1. 跑 Integrity，列出缺失项
2. 按优先级推断真实进度：`tickets/*.md` 且阶段≥步骤4 → 步骤4+；`tickets/*.md` 且阶段仍为步骤3 → **编码前 CHECKPOINT**；`specs/*.md` → 步骤3；`CONTEXT.md` 或 `adr/*.md` → 步骤2+；否则 → 步骤1
3. **重建 todolist**（删虚假元信息），从推断步骤继续
4. 展开完整状态表，「已跳过」写「上次 workflow 损坏已修复」

## 自检清单

### 编码前

- [ ] 本轮为 CHECKPOINT / Integrity 时已附**完整状态表**；普通追问轮仅一行进度
- [ ] doc_root 四目录已存在
- [ ] CONTEXT.md / todolist.md 在项目根，**未**误入 doc_root
- [ ] Integrity 已跑并写入完整状态表
- [ ] spec 与 tickets 已落盘；todolist 任务来自票文件
- [ ] 编码前 CHECKPOINT 整包已展示且用户已确认
- [ ] CONTEXT 无 API/路径等技术 HOW
- [ ] 未在 CHECKPOINT 确认前写业务代码

### 步骤4 结束后

- [ ] **未**自动 commit / push / 归档
- [ ] 已停住并附完整状态表；提示：查看变更 → 发布（先审查再 `/huahai-workflow-gf`）/ 「归档」

### 发布前（步骤5–6）

- [ ] 已走步骤5：审查 skill / 自审 / 或用户跳过
- [ ] 未审查且未跳过时**未**调用 `/huahai-workflow-gf`
