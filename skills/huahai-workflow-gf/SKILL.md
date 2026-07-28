---
name: huahai-workflow-gf
description: >
  Use when the user calls /huahai-workflow-gf, or explicitly asks to commit and/or push
  (e.g. 提交并推送、推送代码、git flow). Also use when that same prompt mentions 归档/archive
  for grillme docs. Do not use for discussing commit style only, or for viewing diffs without
  a request to commit/push.
---

# Git Flow（huahai-workflow-gf）

状态 → 暂存 → 提交 → 拉取 → 推送。全程自动，提交信息按变更生成。归档**仅当**本轮提示词写明归档意图。

归档权威步骤：同级安装时读 [`../huahai-workflow/references/archive.md`](../huahai-workflow/references/archive.md)；若不可用，用下文「归档」步骤。

---

## 归档开关（先于流程，全局生效）

扫描**本轮用户提示词**，在回复写明一行：`归档：是（推送后执行）` 或 `归档：否（仅提交推送）`。

| 提示词 | 行为 |
|--------|------|
| **含**归档措辞（`归档`、`一并归档`、`发布并归档`、`gf 并归档`、`提交并归档`、`推送并归档`、`archive` 指 workflow 归档） | git 全流程**之后**归档 |
| **不含** | **只**做 git；**禁止**归档、**禁止**删 `todolist.md` |

不算归档：业务文案「合同归档」等且未要求整理 grillme；单独说「归档」未触发本 skill → 由 `huahai-workflow` 步骤7 / [archive.md](../huahai-workflow/references/archive.md) 处理。

后续步骤一律按此判断，不再重述。

---

## 执行流程

### 1. 检查状态（自动）

```bash
git status
```

- 有未跟踪 / 已修改 → 列出，继续
- 工作区干净 → 终止（若本轮要求归档且仅有文档待整理，可单独归档，仍须提示词含归档）

### 2. 暂存（自动）

若 `.gitignore` 规则存在但文件仍被跟踪，先从缓存移除（不删本地）：

```bash
git ls-files -i --exclude-from=.gitignore   # 有输出则继续下行
git rm --cached <上述列出的文件>
git add -A
```

`todolist.md` 若在 `.gitignore` 中不要强行入库。暂存后再 `git status`。

### 3. 提交（自动）

```bash
git diff --cached --stat
git commit -m "<自动生成的提交信息>"
```

**提交信息**（conventional commits，中/英均可）：

- subject ≤50 字符，祈使语气，无句号
- 前缀：`feat` / `fix` / `docs` / `refactor` / `style` / `test` / `chore`
- 单文件取功能名，多文件归纳目的
- **禁止** `Co-Authored-By:` 或任何 AI 归因尾注

**粒度**：一个 commit ≈ 一子任务；宜 1–8 文件、50–300 行。超过约 500 行或明显多主题 → **拆成多个 commit**（仍自动，按主题拆）；每个须能编译或过项目既有验证。

### 4. 拉取（自动）

```bash
git pull origin <当前分支>
```

冲突 → 显示冲突文件、暂停、等用户解决。**未推送成功前不归档**。

### 5. 推送（自动）

```bash
git push origin <当前分支>
```

### 6. 归档（仅当开关为是）

前置：开关为是，且步骤 5 推送成功（无代码变更仅归档时，跳过 1–5 中无意义步骤，仍须提示词含归档）。

读并执行 [archive.md](../huahai-workflow/references/archive.md)。若该文件不可用，按下列兜底：

1. 从 `todolist.md` 读 `doc_root`（无则用户指定或 `docs/grillme`）
2. 归档前对齐（as-built / 实际结果）；无偏差写「对齐检查：通过」
3. 创建 `archive/YYYY-MM-DD-<feature>/`，移入 adr/specs/tickets（CONTEXT 留项目根），删 `todolist.md`
4. 有路径变更则再提交推送：`docs: 归档 <feature>`

---

## 特殊情况

- **无变更**：未要求归档 → 提示终止；要求归档 → 仅步骤6，有文件移动再提交推送
- **分支不存在**：列可用分支，询问选择

---

## 快捷用法

| 用户输入 | 执行 |
|---------|------|
| `/huahai-workflow-gf` | 仅 git（不归档） |
| 同上且提到归档 | git + 归档 |
| `git flow` / 明确提交或推送代码 | 同左；提示词含归档则追加归档 |

---

## 注意事项

- 在 git 仓库下执行；需有远程
- 与 `huahai-workflow` 一致：本 skill **不暗含归档**
- 入口：`/huahai-workflow-gf` 或 Skill(`huahai-workflow-gf`)
