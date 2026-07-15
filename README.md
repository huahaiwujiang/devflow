# huahai-workflow — 开发工作流

基于 mattpocock 方法论，从追问到归档的完整开发链路，面向 Claude Code 的项目级开发工作流。

## 工具有什么

两个 skill：

| Skill | 作用 |
|-------|------|
| `huahai-workflow` | 核心工作流。追问→设计→拆分→编码→发布→归档，启动时自动检查依赖。 |
| `huahai-workflow-gf` | Git 流程。状态→暂存→提交→拉取→推送。 |

## 安装

### 1. 克隆仓库

```bash
git clone git@github.com:huahaiwujiang/huahai-workflow.git
```

### 2. 安装到项目

将 skill 目录复制到目标项目的 `.claude/skills/` 下：

```bash
cp -r huahai-workflow/huahai-workflow .claude/skills/huahai-workflow
cp -r huahai-workflow/huahai-workflow-gf .claude/skills/huahai-workflow-gf
```

### 3. 安装依赖

```bash
claude plugin install mattpocock-skills@mattpocock
```

安装完成后 `/reload-skills`。

## 使用

安装完成后，AI 每次会话自动加载 huahai-workflow，按链路推进：追问 → 设计 → 拆分 → 编码 → 发布 → 归档。

开工时 AI 自动读 `todolist.md`（启动门禁创建，进度指针），从上次中断处继续。

## 文档目录

| 位置 | 文件 | 说明 |
|------|------|------|
| 项目根目录 | `CONTEXT.md` | 领域术语表（跨任务累积，不归档） |
| 项目根目录 | `todolist.md` | 进度指针（步骤6删除） |
| `<doc_root>/adr/` | 架构决策记录 | 步骤1产出 |
| `<doc_root>/specs/` | 设计文档 | 步骤2产出 |
| `<doc_root>/tickets/` | 票文件 | 步骤3产出 |
| `<doc_root>/archive/` | 归档文档 | 步骤6移入 |

## 许可

MIT
