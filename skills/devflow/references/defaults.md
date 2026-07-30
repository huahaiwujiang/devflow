# doc_root 默认与目录约定

## 两个根（勿混）

| 根 | 含义 | 放什么 |
|----|------|--------|
| **项目根** | git 仓库根；无 git 则 cwd | `CONTEXT.md`、`todolist.md`（跨任务累积，不随特性归档） |
| **doc_root** | 本任务文档根（优先复用已有默认） | `adr/`、`specs/`、`tickets/`、`archive/` |

**禁止**把 `CONTEXT.md` / `todolist.md` 放进 doc_root。

## doc_root 选取（有则复用，无才问）

**先探测，再决定是否停顿确认。** 禁止在已有默认目录时再问一遍。

### 探测顺序

| 优先级 | 条件 | 行为 |
|--------|------|------|
| 1 | 用户本轮消息中指定了目录 | 采用该路径；缺四子目录则补建；**不问** |
| 2 | 仓库内已有唯一 `*/docs/grillme`（含 `docs/grillme`） | **直接复用**为 doc_root；进度行注明路径；**不问** |
| 3 | 多个 `*/docs/grillme` | 列出匹配项，等用户选一个（仅此情况停顿） |
| 4 | 无任何 `*/docs/grillme`，但 todolist 已写 `doc_root:` 且目录在 | **复用**该路径；缺四子目录则补建；**不问** |
| 5 | 以上皆无 | 列出候选（推荐 `docs/grillme`）等确认；**仅此时**独立停顿 |

### 四子目录

确认或复用 doc_root 后，检查并保证存在（不存在则建，空目录也要保留）：

```
<doc_root>/adr/
<doc_root>/specs/
<doc_root>/tickets/
<doc_root>/archive/
```

四子目录已在 → **不必再问确认目录**，直接进入后续；缺哪个补哪个。

归档只搬移**文件/任务子目录**，**禁止删除**上述四个空目录（见 [archive.md](archive.md)），以便下次启动直接复用。

## 文档树

```
<doc_root>/
├── adr/           # 步骤1，可为空（归档后仍保留空目录）
├── specs/         # 步骤2，必有文件（归档后仍保留空目录）
├── tickets/       # 步骤3，必有文件（归档后仍保留空目录）
└── archive/       # 步骤7（手动）；内含 YYYY-MM-DD-<feature>/

项目根/
├── CONTEXT.md     # 跨任务累积
└── todolist.md    # 进度指针，gitignore
```
