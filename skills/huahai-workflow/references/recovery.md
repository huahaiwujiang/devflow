# Integrity、损坏恢复与自检

## Integrity 校验（步骤4 编码前必跑）

读取 todolist.md 元信息，**逐项验证文件存在**，在状态报告写「Integrity：通过」或列出失败项：

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
4. 状态报告「已跳过」写「上次 workflow 损坏已修复」

## 自检清单

### 编码前

- [ ] 本轮末尾已附 Workflow 状态报告
- [ ] doc_root 四目录已存在
- [ ] CONTEXT.md / todolist.md 在项目根，**未**误入 doc_root
- [ ] Integrity 已跑并写入状态报告
- [ ] spec 与 tickets 已落盘；todolist 任务来自票文件
- [ ] 编码前 CHECKPOINT 整包已展示且用户已确认
- [ ] CONTEXT 无 API/路径等技术 HOW
- [ ] 未在 CHECKPOINT 确认前写业务代码

### 步骤4 结束后

- [ ] **未**自动 commit / push / 归档
- [ ] 已停住并提示：查看变更 → 发布（先审查再 gf）/ 「归档」

### 发布前（步骤5–6）

- [ ] 已走步骤5：审查 skill / 自审 / 或用户跳过
- [ ] 未审查且未跳过时**未**调用 `/huahai-workflow-gf`
