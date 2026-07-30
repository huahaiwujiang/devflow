# doc_root 默认与目录约定

## 两个根（勿混）

| 根 | 含义 | 放什么 |
|----|------|--------|
| **项目根** | git 仓库根；无 git 则 cwd | `CONTEXT.md`、`todolist.md`（跨任务累积，不随特性归档） |
| **doc_root** | 本任务文档根（启动时确认） | `adr/`、`specs/`、`tickets/`、`archive/` |

**禁止**把 `CONTEXT.md` / `todolist.md` 放进 doc_root。

## doc_root 选取优先级

在回复中列出选项并等用户确认（禁止静默默认）：

| 优先级 | 路径 | 适用 |
|--------|------|------|
| 1 | 用户指定 | 消息中给出的目录 |
| 2 | `docs/grillme` | 通用默认（推荐） |
| 3 | 仓库内已有 `*/docs/grillme` | 若探测到唯一匹配可作候选（如部分 monorepo 子项目） |

确认后立即创建（不存在则建，空目录也要建）：

```
<doc_root>/adr/
<doc_root>/specs/
<doc_root>/tickets/
<doc_root>/archive/
```

## 文档树

```
<doc_root>/
├── adr/           # 步骤1，可为空
├── specs/         # 步骤2，必有文件
├── tickets/       # 步骤3，必有文件
└── archive/       # 步骤7（手动）

项目根/
├── CONTEXT.md     # 跨任务累积
└── todolist.md    # 进度指针，gitignore
```
