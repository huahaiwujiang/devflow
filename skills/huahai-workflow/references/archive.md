# 归档（权威副本）

用于：仅「归档」、或 gf 提示词含归档且推送成功后。

## 归档前对齐

对比实际代码变更与 spec、tickets（可用 `git log --oneline -20 --stat`），归档「实际产物」而非「初始计划」：

- 查：有无变更未被任何票覆盖；有无票验收与实现不符；spec 是否与最终代码一致
- 有偏差 → spec 追加 `## as-built 备注`；受影响票追加 `## 实际结果`（「已实现 / 部分 / 合并到XX票」+ 一句话）
- 无偏差 → 状态报告或回复写「对齐检查：通过」

## 搬移

1. 从 `todolist.md` 读 `doc_root`（无则用户指定或 `docs/grillme`）
2. 创建 `<doc_root>/archive/YYYY-MM-DD-<feature>/`（重名则 `-2`、`-3`…）
3. 移入 `<doc_root>/adr/`、`specs/`、`tickets/` 下当前任务产物（**`CONTEXT.md` 留项目根**）
4. 删除项目根 `todolist.md`
5. 若因归档产生入库路径变更，再走一轮 git：暂存 → 提交（`docs: 归档 <feature>`）→ 拉取 → 推送
