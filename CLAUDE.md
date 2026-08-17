# CLAUDE.md

This file provides guidance when working on this repository (any coding agent).

## 项目概述

devflow 是面向多 Agent 的 skill 集合，提供基于 mattpocock 方法论的开发工作流——从追问到发布。通过 SKILL.md 定义行为规范；用 `npx skills add` 从 GitHub 安装到当前宿主的 skills 目录。步骤 1 用独立 skill `grilling`；domain-modeling / to-spec / to-tickets / tdd 放在 `skills/devflow/` 内，随 `devflow` 一并安装。

## 目录结构

```
skills/devflow/SKILL.md                 # 核心工作流骨架
skills/devflow/references/              # todolist、恢复、审查、归档、doc_root（权威细节）
skills/devflow/domain-modeling/         # CONTEXT / ADR
skills/devflow/to-spec/                 # 步骤2 设计
skills/devflow/to-tickets/              # 步骤3 拆票
skills/devflow/tdd/                     # 步骤4 编码
skills/grilling/SKILL.md                # 追问原语（独立 skill）
skills/devflow-publish/SKILL.md         # 发布流程；归档引用主 skill references/archive.md
skills/devflow-issue/SKILL.md           # Issue 闭环；审查引用主 skill references/pre-push-review.md
```

安装后宿主 skills 下并排的是 `devflow`、`devflow-publish`、`devflow-issue`、`grilling`。`devflow-publish` / `devflow-issue` 对 `../devflow/references/` 的相对路径仍有效。

## 修改 skill 的注意事项

- SKILL.md 的 YAML frontmatter 中 `name` 和 `description` 决定触发与显示
- **description 只写触发/反触发，不要摘要流程步骤**
- 改主流程步骤链路时，同步硬门禁表与 `references/todolist.md`、`references/recovery.md`
- 审查 / 归档分别以 `references/pre-push-review.md`、`references/archive.md` 为唯一权威；发布 / issue skill 只引用
- 步骤1 用独立 skill `grilling`；步骤2–4 模块在 `skills/devflow/` 内按相对路径读取。缺文件则手搓兜底
- 目录名与 frontmatter `name` 一致；slash 为 `/<name>`
- 无工单新特性 → `/devflow`；`PROJ-123` / 修缺陷 / `/devflow-issue` → issue skill
- 发布：`/devflow-publish` 或 Skill(`devflow-publish`)
- 发布 skill 套件时保持 `skills/<name>/` 布局，便于 `npx skills add` 发现
