---
name: jira-tasks
description: 连接自建 Jira/Issue 系统，查看任务、筛选未完成事项、结合代码仓库分析影响范围，并在用户确认后完成状态流转与经办人回写。当用户提到 issue、Jira、任务、缺陷、bug、需求、待办、JQL、项目进度，或给出形如 PROJ-123 的 issue key 时，都应使用此 skill
user_invocable: true
---

# Issue 任务 Skill

连接自建 Jira Server，查看任务、筛选未完成事项、分析代码影响面，并自动完成状态流转与经办人更新。

## 环境变量

以下三项**必须**在 `~/.claude/settings.json` 的 `env` 中配置（一次性，之后自动读取）：

- `JIRA_BASE_URL` — 自建 Jira 地址，如 `https://issue.example.com`
- `JIRA_TOKEN` — Personal Access Token
- `JIRA_TESTER` — 默认测试人员（Jira 显示名），其他项目组改此配置即可复用

```json
// ~/.claude/settings.json
{
  "env": {
    "JIRA_BASE_URL": "https://issue.example.com",
    "JIRA_TOKEN": "your-token-here",
    "JIRA_TESTER": "测试人员名称"
  }
}
```

Token 缺失或过期时直接问用户要。拿到后用 node 合并写入 `~/.claude/settings.json` 的 `env.JIRA_TOKEN`，同时 `export JIRA_TOKEN=xxx`。

内网自签证书，所有 curl 加 `-k`。JSON 解析统一用 `node -e`。

## curl 约定

GET 模板（`curl ...` 即沿用此 header）：

```bash
curl -s -k -H "Authorization: Bearer $JIRA_TOKEN" -H "Accept: application/json" "$JIRA_BASE_URL/REST_PATH"
```

POST/PUT 追加 `-H "Content-Type: application/json"`。模板以外的独立 curl 命令同样无需重复 header，用 `curl ...`。

## 工作原则

- 先鉴权再查询。token 缺失/过期直接问用户要，不停。
- 用户给 `ISSUE-123` 这类 key → 优先走单 issue 详情流。
- 用户未给 key → 必须先确认项目和时间窗再搜索。
- Issue 写操作：步骤 8（开始编码）需用户确认；步骤 11（修复完成→修复待验证+分配测试）自动执行，仅在出错或找不到目标用户时才询问。
- Plan Mode 下只分析不修改。
- 代码分析优先用当前仓库。
- 开始编码前自动流转状态为 **处理中**。
- 编码完成后先运行 `/code-review` 审查代码质量。
- 审查通过后运行 `/gf` 推送代码。
- 代码推送完成后流转状态为 **修复待验证(测试)** 并分配给 `$JIRA_TESTER`。

## 工作流

以下所有步骤自动执行，不需要用户手动操作。步骤 8（开始编码→处理中）写操作前需用户确认；步骤 11（修复完成→修复待验证+分配测试）自动执行，仅在出错或找不到目标用户时才询问用户。

### 1. 鉴权

```bash
curl -s -k -o /dev/null -w "%{http_code}" -H "Authorization: Bearer ${JIRA_TOKEN:-MISSING}" -H "Accept: application/json" "$JIRA_BASE_URL/rest/api/2/myself"
```

- `200` → 通过
- `401/403` → token 无效，问用户要
- 连接失败 → 提示检查 VPN
- `$JIRA_TOKEN` 未设 → 问用户要

### 2. 判断入口

**有 issue key：** 鉴权后直接获取详情，跳到步骤 5。

**无 issue key：** 确认筛选条件：

- 项目（必填，key/全名/简称）
- 时间窗（默认 `14d`，支持 `7d`/`14d`/`30d`/`90d`）
- 代码关键词（可选）

### 3. 解析项目并构造 JQL

精确 project key（如 `ZCCP`）直接使用。中文名/简称才拉项目列表：

```bash
curl ... "$JIRA_BASE_URL/rest/api/2/project" | node -e "process.stdin.on('data',d=>JSON.parse(d).forEach(p=>console.log(JSON.stringify({key:p.key,name:p.name}))))"
```

匹配规则：key+名称大小写不敏感；0 匹配→修正；多匹配→让用户选；去重后构造 JQL。

**JQL 强制前缀（不可省略）：** `assignee = currentUser() AND resolution = Unresolved`

每次搜索必须以此开头，然后依次追加：`project in (PROJ)` → `updated >= -Nd` → `text ~ "keyword"` → `ORDER BY priority ASC, updated DESC`。用户未指定时间窗时默认 `updated >= -14d`。

### 4. 搜索 Issue

```bash
curl ... -X POST "$JIRA_BASE_URL/rest/api/2/search" -d '{"jql":"<JQL>","maxResults":20,"fields":["summary","status","priority","issuetype","created","updated","project"]}'
```

先说明筛选条件，再用表格展示 `key / summary / status / priority`，最多展示 4 个候选。

### 5. 获取 Issue 详情

```bash
curl ... "$JIRA_BASE_URL/rest/api/2/issue/ISSUE_KEY?fields=summary,description,status,priority,issuetype,assignee,reporter,comment,subtasks,issuelinks,attachment,labels,components"
```

展示：标题 `[KEY] Summary`、状态/优先级/类型、描述、最近 5 条评论、子任务/关联任务、标签/组件。wiki markup 转可读文本。

### 6. 代码仓库定位与影响分析

```bash
git remote -v 2>/dev/null | head -2
```

当前目录是仓库→优先用；不是→让用户提供路径。

分析策略：关键词 Grep → Glob 结构定位 → 调用链追踪（上下双向）→ 测试覆盖搜索。

### 7. 输出修复方案

产出：可能根因、影响范围、建议修改点、风险/边界条件、测试缺口、复杂度判断。

用户只想查看→到此结束。用户想修复→Plan Mode 只出计划，否则进入步骤 8。

### 8. 开始编码 → 状态 = 处理中

获取可用流转并匹配"处理中"（模糊匹配"In Progress"/"进行中"）：

```bash
curl ... "$JIRA_BASE_URL/rest/api/2/issue/ISSUE_KEY/transitions" | node -e "process.stdin.on('data',d=>JSON.parse(d).transitions.forEach(t=>console.log(JSON.stringify({id:t.id,name:t.name}))))"
```

匹配到后向用户确认，确认后执行（POST 返回 204 无 body，不要 pipe 给 node）：

```bash
curl ... -w "%{http_code}" -X POST "$JIRA_BASE_URL/rest/api/2/issue/ISSUE_KEY/transitions" -d '{"transition":{"id":"ID"}}'
```

- `200`/`204` → 成功
- 其他 → 检查 transition id 是否正确

匹配不到则列出全部可用状态让用户手动选。

### 9. 代码审查 → /code-review

编码完成后、流转状态前，必须运行代码审查：

```bash
/code-review
```

审查通过（无严重问题）后继续步骤 10。如有问题先修复再审查，直到通过。

### 10. 推送代码 → /gf

代码审查通过后，推送代码到远程仓库：

```bash
/gf
```

推送完成后继续步骤 11。分批修复时注意：所有子任务代码推送完毕后，最后统一分配测试人员。

### 11. 修复完成 → 状态 = 修复待验证(测试) + 分配给 $JIRA_TESTER

先总结修改内容（文件、问题、测试结果、未覆盖风险）。

**流转状态（自动执行，无需确认）。** 获取 transitions 并匹配，同步骤 8：GET → node 匹配。匹配"修复待验证"/"待验证"/"测试"，匹配到后直接 POST 流转，不等待用户确认。匹配不到或 POST 失败才列全部可用状态让用户选：

```bash
curl ... "$JIRA_BASE_URL/rest/api/2/issue/ISSUE_KEY/transitions" | node -e "process.stdin.on('data',d=>JSON.parse(d).transitions.forEach(t=>console.log(JSON.stringify({id:t.id,name:t.name}))))"
```

POST 流转（返回 204 无 body）：

```bash
curl ... -w "%{http_code}" -X POST "$JIRA_BASE_URL/rest/api/2/issue/ISSUE_KEY/transitions" -d '{"transition":{"id":"ID"}}'
```

**分配经办人（自动执行，无需确认）。** 目标用户 `$JIRA_TESTER`：搜索时中文必须 URL 编码，优先用 `query`（`username` 已废弃）：

```bash
curl ... --get --data-urlencode "issueKey=ISSUE_KEY" --data-urlencode "query=$JIRA_TESTER" "$JIRA_BASE_URL/rest/api/2/user/assignable/search"
```

匹配到唯一用户后直接 PUT 分配，不等待用户确认：

```bash
curl ... -X PUT "$JIRA_BASE_URL/rest/api/2/issue/ISSUE_KEY/assignee" -d '{"name":"MATCHED_USERNAME"}'
```

仅以下情况才询问用户：
- 状态匹配不到 → 列出可用状态让用户选
- 用户匹配不到 → 降级搜名/拼音 → 仍无则列可用用户让用户选
- 任何 API 返回非 200/204 → 报告错误并等待用户指示

## 错误处理

| 场景                    | 处理                    |
| ----------------------- | ----------------------- |
| token 缺失/过期/401/403 | 直接问用户要            |
| 网络失败                | 提示检查 VPN            |
| 未提供项目              | 先补项目再搜索          |
| 项目无匹配              | 修正项目名              |
| 项目多匹配              | 让用户选精确 key        |
| 未提供时间窗            | 默认 14d                |
| 搜索无结果              | 展示筛选条件，建议放宽  |
| 步骤8写操作前未确认       | 不执行                  |
| 步骤11自动执行失败         | 报告错误，询问用户        |
| 状态匹配不到            | 列出可用状态让用户选    |
| 用户匹配不到            | 降级搜索 → 列出可用用户 |
| Plan Mode               | 只分析不修改            |
