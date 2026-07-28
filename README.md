# huahai-workflow

基于 mattpocock 方法论的开发工作流（追问→设计→拆分→编码→审查→发布→归档）。三个 skill **一并安装**。

| Skill | 用途 |
|-------|------|
| `huahai-workflow` | 无工单新特性 |
| `huahai-workflow-gf` | 提交 / 推送（提示词含「归档」则一并归档） |
| `huahai-workflow-issue` | `PROJ-123` / 修缺陷 / Issue 查修交测 |

## 安装

```bash
npx skills add huahaiwujiang/huahai-workflow --skill '*' -y
claude plugin install mattpocock-skills@mattpocock   # 主流程依赖
```

装完后 `/reload-skills`（或新开会话）。全局加 `-g`；指定宿主加 `-a claude-code` / `-a cursor`；symlink 不稳加 `--copy`。

给 AI：贴仓库 URL，说「用 `npx skills add <url> --skill '*' -y` 装齐三个 skill」。

Issue 可选环境变量：`JIRA_BASE_URL`、`JIRA_TOKEN`、`JIRA_TESTER`。

## 使用

- `/huahai-workflow` — 新特性
- `/huahai-workflow-gf` — 发布
- `/huahai-workflow-issue` / `PROJ-123` — 缺陷

进度在项目根 `todolist.md`；文档默认 `docs/grillme`（`adr` / `specs` / `tickets` / `archive`）。

## 许可

MIT
