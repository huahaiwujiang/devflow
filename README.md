# huahai-workflow — 开发工作流

基于 mattpocock 方法论，从追问到归档的完整开发链路。三个 skill 请**一并安装**（并排），以便 gf / issue 能读到主 skill 的共享 `references/`。

## 工具有什么

| Skill | 作用 |
|-------|------|
| `huahai-workflow` | 核心工作流。追问→设计→拆分→编码→审查→发布→归档；细节在 `references/`。无工单新特性走此流程。 |
| `huahai-workflow-gf` | Git 流程。状态→暂存→提交→拉取→推送；提示词含归档时再归档。 |
| `huahai-workflow-issue` | Issue 闭环。查/筛任务、影响分析、确认后流转与交测（可选，需 Issue 环境变量）。 |

## 安装（推荐）

用开源 [skills CLI](https://github.com/vercel-labs/skills) 从 GitHub 安装（需 Node.js）。仓库公开后把 `owner/repo` 换成实际地址。

### 一行装齐（项目级）

在目标项目根目录执行：

```bash
npx skills add huahaiwujiang/huahai-workflow --skill '*' -y
```

或完整 URL / git 地址：

```bash
npx skills add https://github.com/huahaiwujiang/huahai-workflow --skill '*' -y
npx skills add git@github.com:huahaiwujiang/huahai-workflow.git --skill '*' -y
```

### 常用变体

```bash
# 先列出仓库内 skill
npx skills add huahaiwujiang/huahai-workflow --list

# 装到 Claude Code（项目 .claude/skills/）
npx skills add huahaiwujiang/huahai-workflow --skill '*' -a claude-code -y

# 同时装到 Claude Code + Cursor
npx skills add huahaiwujiang/huahai-workflow --skill '*' -a claude-code -a cursor -y

# 用户级全局（全项目可用）
npx skills add huahaiwujiang/huahai-workflow --skill '*' -g -y

# Windows 或 symlink 不稳时用复制
npx skills add huahaiwujiang/huahai-workflow --skill '*' --copy -y
```

三个 skill 必须一起装。装完后 Claude Code 执行 `/reload-skills`（或新开会话）。

### 依赖（主流程步骤 1–4）

```bash
claude plugin install mattpocock-skills@mattpocock
```

### 把仓库地址交给 AI

把下面整段（或只贴仓库 URL +「按 README 安装」）发给 Agent 即可：

> 请用 skills CLI 安装 huahai-workflow 全部 skill（三个必须一起装）：  
> `npx skills add https://github.com/huahaiwujiang/huahai-workflow --skill '*' -a claude-code -y`  
> 若本机用 Cursor，再加上 `-a cursor`。装完提醒我 `/reload-skills` 或重启会话。  
> 另需：`claude plugin install mattpocock-skills@mattpocock`（主流程依赖）。

### 手动安装（兜底）

```bash
git clone https://github.com/huahaiwujiang/huahai-workflow.git
cp -r huahai-workflow/skills/huahai-workflow .claude/skills/huahai-workflow
cp -r huahai-workflow/skills/huahai-workflow-gf .claude/skills/huahai-workflow-gf
cp -r huahai-workflow/skills/huahai-workflow-issue .claude/skills/huahai-workflow-issue
```

### Issue skill（可选）

使用 `huahai-workflow-issue` 时，首次会门禁探测 `JIRA_BASE_URL` + `JIRA_TOKEN`；未配置则索取并写入 `~/.claude/settings.json` 的 `env`（其他宿主用等价环境变量）。也可预先配置：

- `JIRA_BASE_URL`
- `JIRA_TOKEN`
- `JIRA_TESTER`（可选；交测分配前若缺再问）

## 使用

- 新特性 / `/huahai-workflow` → 追问 → 设计+拆分（串联）→ 编码前确认 → 编码；发布前默认审查（可跳过；发布/归档手动）
- `/huahai-workflow-gf` 或明确提交推送 → git 全流程
- `/huahai-workflow-issue`、`PROJ-123` 或明确修缺陷 → Issue 查修与交测

开工时读项目根 `todolist.md` 续跑。默认 `doc_root` 为 `docs/grillme`，见 `skills/huahai-workflow/references/defaults.md`。

## 文档目录

| 位置 | 文件 | 说明 |
|------|------|------|
| 项目根 | `CONTEXT.md` | 领域术语（跨任务累积） |
| 项目根 | `todolist.md` | 进度指针（归档时删除） |
| `<doc_root>/adr/` | ADR | 步骤1 |
| `<doc_root>/specs/` | 设计 | 步骤2 |
| `<doc_root>/tickets/` | 票 | 步骤3 |
| `<doc_root>/archive/` | 归档 | 步骤7 |

## 许可

MIT
