---
name: huahai-workflow-issue
description: >
  Use when the user calls /huahai-workflow-issue, gives an issue key like PROJ-123,
  or clearly wants to view/fix bugs, browse issues, run JQL, or check project progress
  on the self-hosted Issue system. Do not use for greenfield features without a ticket
  (/huahai-workflow); do not run grilling→spec→tickets when an issue key or bugfix is in play.
user_invocable: true
---

# Issue 任务 Skill（huahai-workflow-issue）

调用：`/huahai-workflow-issue` 或 Skill(`huahai-workflow-issue`)。无工单新特性走 `/huahai-workflow`。

推送前审查权威副本：同级安装时读 [`../huahai-workflow/references/pre-push-review.md`](../huahai-workflow/references/pre-push-review.md)。

## 约定

- 配置（权威源 = **系统/用户环境变量** → `process.env`）：
  - `JIRA_BASE_URL`、`JIRA_TOKEN`（必填）
  - `JIRA_TESTER`（交测用）
  - 缺则门禁向用户一次性索取；当前会话 `export`（Windows PowerShell：`$env:NAME=...`）跑通；并提示写入**用户环境变量**持久化（Win：系统设置或 `setx`；改完需新开 IDE/终端）。正文不写真实地址/人名。
- curl：一律 `-k`（内网自签常见；仅限可信内网）。JSON 用 `node -e`。后文 `curl ...` =  
  `curl -s -k -H "Authorization: Bearer $JIRA_TOKEN" -H "Accept: application/json"`（POST/PUT 再加 `Content-Type: application/json`）。
- 确认：步骤 4 写状态前要确认；步骤 7 成功路径自动。只出计划不改代码时（只分析）到步骤 3 结束。

## 工作流

步骤 4 写操作前需确认；步骤 7 失败才问人。

### 1. 配置门禁与鉴权

**1a. 探测配置（必做，禁止跳过）**

只查 `process.env`：

```bash
node -e "console.log(JSON.stringify({hasUrl:!!process.env.JIRA_BASE_URL,hasToken:!!process.env.JIRA_TOKEN,hasTester:!!process.env.JIRA_TESTER}))"
```

| 探测结果 | 动作 |
|----------|------|
| `hasUrl` 与 `hasToken` 均为 true | → 1b |
| 缺 `JIRA_BASE_URL` 或 `JIRA_TOKEN` | **立刻停下**，一次性索取，**禁止**继续步骤 2 |
| 仅缺 `JIRA_TESTER` | 不阻塞；建议一并收集 |

**索取（必填）**：Issue 系统 HTTPS 根地址（`JIRA_BASE_URL`）；Personal Access Token（`JIRA_TOKEN`）。可选：`JIRA_TESTER` 显示名。

用户回复后：

1. 当前会话 `export` / `$env:` 注入三项（已有的可跳过）
2. 告知用户用系统「用户环境变量」或 `setx` 持久化
3. → 1b。**提供之前不要调 Issue API。**

**1b. 鉴权**

```bash
curl -s -k -o /dev/null -w "%{http_code}" -H "Authorization: Bearer ${JIRA_TOKEN:-MISSING}" -H "Accept: application/json" "$JIRA_BASE_URL/rest/api/2/myself"
```

- `200` → 步骤 2
- `401/403` → 要新 token；会话内更新 env，并提示用户改用户环境变量
- 连接失败 → 查 VPN / 地址；地址错则更新 `JIRA_BASE_URL`（同上）
- token/url 仍空 → 回 1a，不可用 `MISSING` 蒙混

### 2. 定位 Issue

**路径 A — 有 issue key：** 鉴权后拉详情。

**路径 B — 无 key：**

1. 确认：项目（必填）、时间窗（默认 `14d`）、代码关键词（可选）
2. 精确 project key 直接用；中文名/简称才拉项目列表：

```bash
curl ... "$JIRA_BASE_URL/rest/api/2/project" | node -e "process.stdin.on('data',d=>JSON.parse(d).forEach(p=>console.log(JSON.stringify({key:p.key,name:p.name}))))"
```

匹配：key+名称大小写不敏感；0→修正；多→用户选。

**JQL 强制前缀：** `assignee = currentUser() AND resolution = Unresolved`  
再追加：`project in (PROJ)` → `updated >= -Nd` → `text ~ "keyword"` → `ORDER BY priority ASC, updated DESC`。默认 `updated >= -14d`。

3. 搜索展示最多 4 个候选：

```bash
curl ... -X POST "$JIRA_BASE_URL/rest/api/2/search" -d '{"jql":"<JQL>","maxResults":20,"fields":["summary","status","priority","issuetype","created","updated","project"]}'
```

表格：`key / summary / status / priority`；选定后再拉详情。

**获取详情：**

```bash
curl ... "$JIRA_BASE_URL/rest/api/2/issue/ISSUE_KEY?fields=summary,description,status,priority,issuetype,assignee,reporter,comment,subtasks,issuelinks,attachment,labels,components"
```

展示：标题、状态/优先级/类型、描述、最近 5 条评论、子任务/关联、标签/组件。wiki markup 转可读文本。

### 3. 影响分析与修复方案

1. `git remote -v`：当前目录是仓库优先用；否则让用户给路径
2. 关键词 Grep → Glob → 调用链 → 测试覆盖
3. 输出：可能根因、影响范围、修改点、风险、测试缺口、复杂度

只查看 → 结束。要修复 → 只出计划不改代码则停；否则进步骤 4。

### 4. 开始编码 → 状态 = 处理中

```bash
curl ... "$JIRA_BASE_URL/rest/api/2/issue/ISSUE_KEY/transitions" | node -e "process.stdin.on('data',d=>JSON.parse(d).transitions.forEach(t=>console.log(JSON.stringify({id:t.id,name:t.name}))))"
```

模糊匹配「处理中」/ In Progress / 进行中 → **向用户确认** → POST（204 无 body，勿 pipe node）：

```bash
curl ... -w "%{http_code}" -X POST "$JIRA_BASE_URL/rest/api/2/issue/ISSUE_KEY/transitions" -d '{"transition":{"id":"ID"}}'
```

匹配不到 → 列全部状态让用户选。确认并流转成功后再写业务代码。

### 5. 代码审查（推送前）

读并遵循 [pre-push-review.md](../huahai-workflow/references/pre-push-review.md)。验收依据 = issue 描述与验收点。

若该文件不可用：有审查 skill 则调用；否则自审验收/逻辑/安全（密钥与注入）；或用户跳过后进步骤 6。

### 6. 推送 → /huahai-workflow-gf

审查通过或已跳过 → Skill(`huahai-workflow-gf`) 或 `/huahai-workflow-gf`。完成后进步骤 7。分批修复时：全部推送后再统一交测。

### 7. 交测回写 → 修复待验证 + `$JIRA_TESTER`

先总结：文件、问题、测试结果、未覆盖风险。缺 `JIRA_TESTER` → 分配前向用户要显示名；会话 `export` 并提示写入用户环境变量。

**流转（自动）**：GET transitions → 匹配「修复待验证」/「待验证」/「测试」→ 直接 POST；失败才列状态让用户选。

**分配（自动）**：

```bash
curl ... --get --data-urlencode "issueKey=ISSUE_KEY" --data-urlencode "query=$JIRA_TESTER" "$JIRA_BASE_URL/rest/api/2/user/assignable/search"
```

唯一匹配后 PUT `assignee`；仅缺测试人、状态/用户匹配失败、非 200/204 时才问用户。

## 错误处理

| 场景 | 处理 |
|------|------|
| 缺 URL/TOKEN | 步骤 1a 停下索取；会话 export + 提示写入用户环境变量 |
| 401/403 | 换 token 重试 |
| 网络/地址 | VPN；更新 BASE_URL |
| 缺 TESTER | 步骤 7 前再问 |
| 未提供项目 / 无匹配 / 多匹配 | 补项目；修正；用户选 |
| 未提供时间窗 | 默认 14d |
| 搜索无结果 | 展示条件，建议放宽 |
| 步骤4 未确认 | 不执行写操作 |
| 步骤7 失败 | 报告并询问 |
| 只分析不改代码 | 步骤 3 后结束，不写业务代码 |
