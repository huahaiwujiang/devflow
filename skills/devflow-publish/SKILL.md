---
name: devflow-publish
description: >
  调用 /devflow-publish，或明确要求提交/推送（如提交并推送、推送代码、git flow）时使用；
  同一句提示词含归档/archive（grillme 产物）时一并归档。不要用于只讨论提交规范或仅查看 diff。
---

# Git Flow（devflow-publish）

状态 → 暂存 → 提交 → 拉取 → 推送。全程自动，提交信息按变更生成。归档**仅当**本轮提示词写明归档意图。

归档权威步骤：同级安装时读 [`../devflow/references/archive.md`](../devflow/references/archive.md)；若不可用，用下文「归档」步骤。

---

## 归档开关（先于流程，全局生效）

扫描**本轮用户提示词**，在回复写明一行：`归档：是（先归档再提交推送）` 或 `归档：否（仅提交推送）`。

| 提示词 | 行为 |
|--------|------|
| **含**归档措辞（`归档`、`一并归档`、`发布并归档`、`提交并归档`、`推送并归档`、`archive` 指 workflow 归档） | **先**执行归档，**再**走 git 全流程（一次提交同时带上代码与归档路径变更） |
| **不含** | **只**做 git；**禁止**归档、**禁止**删 `todolist.md` |

**顺序硬规则**：提示词里只要出现归档意图，**一律先归档**——不论用户写成「归档，提交，推送」还是「提交，推送，归档」。**禁止**先提交一轮、再归档、再提交第二轮。

不算归档：业务文案「合同归档」等且未要求整理 grillme；单独说「归档」未触发本 skill → 由 `devflow` 步骤7 / [archive.md](../devflow/references/archive.md) 处理。

后续步骤一律按此判断，不再重述。

---

## 执行流程

### 0. 归档（仅当开关为是 — 必须最先）

前置：开关为是。无文档可整理时写明「无可归档产物」并继续 git（若仍有代码变更）。

读并执行 [archive.md](../devflow/references/archive.md)。若该文件不可用，按下列兜底：

1. 从 `todolist.md` 读 `doc_root`（无则用户指定或已有 `*/docs/grillme`）
2. 归档前对齐（as-built / 实际结果）；无偏差写「对齐检查：通过」
3. 创建 `archive/YYYY-MM-DD-<feature>/`，移入 adr/specs/tickets 下当前任务**文件**（CONTEXT 留项目根）
4. **保留**空的 `adr/`、`specs/`、`tickets/`、`archive/` 四目录；删 `todolist.md`
5. **此处不单独 commit/push**；路径变更并入后面统一提交

### 1. 检查状态（自动）

```bash
git status
```

- 有未跟踪 / 已修改 → 列出，继续
- 工作区干净 → 终止（若本轮要求归档且步骤 0 已完成且无进一步变更，可直接结束）

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
- 单文件取功能名，多文件归纳目的；若本轮含归档路径变更，可在 body 或同一批拆出的 `docs:` commit 中体现归档
- **禁止** `Co-Authored-By:` 或任何 AI 归因尾注

**粒度**：一个 commit ≈ 一子任务；宜 1–8 文件、50–300 行。超过约 500 行或明显多主题 → **拆成多个 commit**（仍自动，按主题拆；归档文档可单独 `docs: 归档 <feature>`，但仍在**同一次**推送前完成，禁止「推完再归档再推」）；每个须能编译或过项目既有验证。

### 4. 拉取（自动）

```bash
git pull origin <当前分支>
```

冲突 → 显示冲突文件、暂停、等用户解决。

### 5. 推送（自动）

```bash
git push origin <当前分支>
```

---

## 特殊情况

- **无变更**：未要求归档 → 提示终止；要求归档 → 仅步骤 0，有文件移动则继续 1–5 一次推完
- **分支不存在**：列可用分支，询问选择

---

## 快捷用法

| 用户输入 | 执行 |
|---------|------|
| `/devflow-publish` | 仅 git（不归档） |
| 同上且提到归档 | **先归档** + 再 git（一次推送） |
| `git flow` / 明确提交或推送代码 | 同左；提示词含归档则**先**归档 |
| `提交，推送，归档` / `归档，提交，推送` | 同序：**归档 → 提交 → 推送** |

---

## 注意事项

- 在 git 仓库下执行；需有远程
- 与 `devflow` 一致：本 skill **不暗含归档**
- 入口：`/devflow-publish` 或 Skill(`devflow-publish`)
