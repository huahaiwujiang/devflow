# 归档（权威副本）

用于：仅「归档」、或 `/devflow-publish` 提示词含归档。

**与发布同句时**：只要提示词含归档意图，**必须先归档、再提交/推送**（见 `devflow-publish`），避免先提交代码、再归档、再提交第二轮。

## 归档前对齐

对比实际代码变更与 spec、tickets（可用 `git log --oneline -20 --stat`），归档「实际产物」而非「初始计划」：

- 查：有无变更未被任何票覆盖；有无票验收与实现不符；spec 是否与最终代码一致
- 有偏差 → spec 追加 `## as-built 备注`；受影响票追加 `## 实际结果`（「已实现 / 部分 / 合并到XX票」+ 一句话）
- 无偏差 → 进度行或回复写「对齐检查：通过」

## 搬移

1. 从 `todolist.md` 读 `doc_root`（无则用户指定或已有 `*/docs/grillme` / `docs/grillme`）
2. 创建 `<doc_root>/archive/YYYY-MM-DD-<feature>/`（重名则 `-2`、`-3`…）
3. 将 `<doc_root>/adr/`、`specs/`、`tickets/` 下**当前任务产物文件**移入该归档目录（**`CONTEXT.md` 留项目根**）
4. **保留**空的 `<doc_root>/adr/`、`specs/`、`tickets/`、`archive/` 四个目录；**禁止** `rmdir` / 删除这四个壳目录（下次启动直接复用，不必再问再建）
5. 删除项目根 `todolist.md`
6. 若本轮**仅**归档（未与提交/推送同句）：路径变更再走一轮 git：暂存 → 提交（`docs: 归档 <feature>`）→ 拉取 → 推送  
   若与提交/推送同句：本步**不单独再提交**；由 `devflow-publish` 在归档完成后统一暂存/提交/推送（含代码与归档路径）
