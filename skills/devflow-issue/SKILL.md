---
name: devflow-issue
description: >
  调用 /devflow-issue、给出 PROJ-123 类 issue key，或明确要修缺陷/修 bug/看 issue/JQL/项目进度时使用。
  无工单新特性改用 /devflow；带 issue key 或修缺陷时不要走 grilling→spec→tickets。
user_invocable: true
---

# Issue 任务 Skill（devflow-issue）

调用：`/devflow-issue`。无工单新特性 → `/devflow`。审查权威：[pre-push-review.md](../devflow/references/pre-push-review.md)。

## 约定

- **配置**：`JIRA_BASE_URL` + `JIRA_TOKEN` 必填，`JIRA_TESTER` 交测用。只认 `process.env`。缺则索取；Agent 写 **User** 环境变量（Win：`SetEnvironmentVariable(...,'User')`）+ 会话 `$env:`/`export`。勿写 Machine；勿回显密文；其它已开窗口需新开才读到 User。
- **curl**：`-k`；`Authorization: Bearer $JIRA_TOKEN`；POST/PUT 加 Content-Type。后文 `curl ...` 即此。Win JSON：`curl -o %TEMP%\jira.json` 再 `node` 解析，忌管道过控制台。
- **回合**：一问一停。配置 ≠ 筛选 ≠ 分析，禁止同回合夹带。步骤 2 禁止 git/Grep/根因；步骤 3 须用户点头后才进。
- **写操作**：步骤 4 流转前确认；步骤 7 成功路径自动。

## 工作流

### 1. 配置门禁与鉴权

```bash
node -e "console.log(JSON.stringify({hasUrl:!!process.env.JIRA_BASE_URL,hasToken:!!process.env.JIRA_TOKEN,hasTester:!!process.env.JIRA_TESTER}))"
```

- 有 URL+TOKEN → 1b。缺任一 → **本回合只**要地址与 Token（可选 TESTER），停；**禁止**提项目/时间窗、禁止调 Issue API。
- 用户回复后：写 User + `$env:` → 1b。

**1b** `GET .../myself`：`200` → 步骤 2；`401/403` → 只要新 Token 并持久化；连不上 → 查 VPN/改 URL。

### 2. 定位 Issue（快，只出工单）

鉴权通过后。**禁止**代码排查。

- **有 key** → 直接拉详情。
- **无 key** → **另回合**只要：项目、时间窗（默认 `14d`）、关键词（可选）。再：精确 key 直用，否则拉 `/project` 匹配；JQL 强制前缀 `assignee = currentUser() AND resolution = Unresolved`，再 `project` / `updated >= -Nd` / `text ~` / `ORDER BY priority ASC, updated DESC`；`POST .../search` 最多展示 4 条，选定后拉详情。

详情 fields：`summary,description,status,priority,issuetype,assignee,reporter,comment,subtasks,issuelinks,attachment,labels,components`。

**交付后停**：标题/状态/优先级/类型/经办人/组件；描述+近 5 评论+子任务/关联/附件（wiki→文本）；缺口一句话（如「描述空」「已关闭」）。三选一：①只看过 ②继续分析→3 ③直接修/补复盘→3（再 4）。

### 3. 影响分析与方案

仅用户选 ②/③ 后。仓库：`git remote -v`（非仓库则问路径）。Grep→Glob→调用链→测试；可看相关提交。产出：根因、范围、改点、风险、测试缺口、复杂度。只分析则结束；要修→计划可停，否则→4。

### 4. 编码 → 处理中

GET transitions → 匹配「处理中」/In Progress → **确认** → POST（204 勿 pipe）。失败列状态。流转成功后再写代码。

### 5. 审查

跟 [pre-push-review.md](../devflow/references/pre-push-review.md)；验收=工单描述/验收点。无文件则：有审查 skill 用 skill，否则自审，或用户跳过→6。

### 6. 推送

Skill(`devflow-publish`) / `/devflow-publish` → 7。分批修：全部推完再交测。

### 7. 交测回写

总结改动。缺 `JIRA_TESTER` → 要显示名并写 User+`$env:`。自动匹配「修复待验证」/「待验证」/「测试」并 POST；再 `assignable/search?query=` → PUT assignee。仅匹配失败或非 200/204 才问人。

## 错误速查

| 场景 | 处理 |
|------|------|
| 缺配置 / 401 | 1a 索取或换 Token，写 User |
| 网络/地址 | VPN；改 URL |
| 项目无/多匹配、无搜索结果 | 修正/用户选；放宽条件 |
| 步骤4 未确认 | 不写代码 |
| 步骤7 失败 | 报告并问 |
