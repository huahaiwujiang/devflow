# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

huahai-workflow 是一个 Claude Code skill 集合，提供基于 mattpocock 方法论的开发工作流——从追问到发布。通过 SKILL.md 文件定义 AI 行为规范，安装到目标项目的 `.claude/skills/` 下使用。

## 目录结构

```
huahai-workflow/SKILL.md          # 核心工作流（追问→设计→拆分→编码→发布→归档），启动时轻量检查 mattpocock-skills 依赖
huahai-workflow-gf/SKILL.md       # 独立 git flow skill（状态→暂存→提交→拉取→推送）
```

## 修改 skill 的注意事项

- SKILL.md 的 YAML frontmatter 中 `name` 和 `description` 字段决定 skill 的触发和显示
- 修改 `huahai-workflow/SKILL.md` 的步骤链路时，需要同步更新 CHECKPOINT 门禁和失败处理表
- 步骤1-4 依赖 mattpocock-skills 插件，不可移除。若 Skill 不可用，走兜底机制
- 嵌套 skill（如 `gf`）的目录名即 skill 名，Claude Code 通过目录结构发现
