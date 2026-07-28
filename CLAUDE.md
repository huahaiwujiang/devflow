# CLAUDE.md

This file provides guidance when working on this repository (any coding agent).

## 项目概述

huahai-workflow 是面向多 Agent 的 skill 集合，提供基于 mattpocock 方法论的开发工作流——从追问到发布。通过 SKILL.md 定义行为规范；用 `npx skills add` 从 GitHub 安装到当前宿主的 skills 目录。

## 目录结构

```
skills/huahai-workflow/SKILL.md              # 核心工作流骨架
skills/huahai-workflow/references/           # todolist、恢复、审查、归档、doc_root（权威细节）
skills/huahai-workflow-gf/SKILL.md           # git flow；归档引用主 skill references/archive.md
skills/huahai-workflow-issue/SKILL.md        # Issue 闭环；审查引用主 skill references/pre-push-review.md
```

安装后三个目录并排（宿主 skills 下的 `huahai-workflow*`），相对路径 `../huahai-workflow/references/` 仍有效。

## 修改 skill 的注意事项

- SKILL.md 的 YAML frontmatter 中 `name` 和 `description` 决定触发与显示
- **description 只写触发/反触发，不要摘要流程步骤**
- 改主流程步骤链路时，同步硬门禁表与 `references/todolist.md`、`references/recovery.md`
- 审查 / 归档分别以 `references/pre-push-review.md`、`references/archive.md` 为唯一权威；发布 / issue skill 只引用
- 步骤1-4 依赖 mattpocock-skills（`npx skills add mattpocock/skills`，按宿主选择 Cursor/Claude/Codex 等）；不可用则手搓兜底
- 目录名与 frontmatter `name` 一致；slash 为 `/<name>`
- 无工单新特性 → `/huahai-workflow`；`PROJ-123` / 修缺陷 / `/huahai-workflow-issue` → issue skill
- 发布：`/huahai-workflow-gf` 或 Skill(`huahai-workflow-gf`)
- 发布 skill 套件时保持 `skills/<name>/` 布局，便于 `npx skills add` 发现
