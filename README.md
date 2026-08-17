# devflow

基于 mattpocock 方法论的开发工作流（追问→设计→拆分→编码→审查→发布→归档）。本仓库 skill **一并安装**，不再依赖 `mattpocock/skills` 插件。

| Skill | 用途 |
|-------|------|
| `devflow` | 无工单新特性（内含 domain-modeling / to-spec / to-tickets / tdd） |
| `grilling` | 追问原语（独立 skill） |
| `devflow-publish` | 提交 / 推送（提示词含「归档」则一并归档） |
| `devflow-issue` | `PROJ-123` / 修缺陷 / Issue 查修交测 |

## 安装

```bash
npx skills@latest add huahaiwujiang/devflow --skill '*' -y
```

安装器会提示装到哪个 Agent（Cursor / Claude Code / Codex 等）。常用参数：`-g` 全局；`-a cursor` / `-a claude-code` / `-a codex` 指定宿主；`--copy` 用拷贝代替 symlink。装完后按宿主习惯重载 skills 或新开会话。

Issue：`JIRA_BASE_URL` / `JIRA_TOKEN` / `JIRA_TESTER` 存**用户环境变量**（缺则门禁索取，由 Agent 写入 User + 当前会话）。

## 使用

- `/devflow` — 新特性
- `/devflow-publish` — 发布
- `/devflow-issue` / `PROJ-123` — 缺陷

进度在项目根 `todolist.md`；文档默认 `docs/grillme`（`adr` / `specs` / `tickets` / `archive`）。

## 许可

MIT
