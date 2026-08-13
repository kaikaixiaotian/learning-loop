---
description: AI 闭环学习——初始化/续学、查进度、升级（拉取最新 skill + 迁移工作区）
argument-hint: "[主题 | status | upgrade]  留空=自动判断；写主题=直接初始化；status=只查进度；upgrade=升级 skill 与工作区"
skills: learning-loop
---

使用 `learning-loop` skill 处理本次学习请求。

$ARGUMENTS

## 按参数分支执行

根据上面的参数（可能为空、是一个主题、`status`、或 `upgrade`），选择对应分支：

### 分支 A：参数为空（`/learning-loop`）
按 skill 的「First run vs. resume」逻辑自动判断：
- 当前目录已有 `*-learning/meta.json` → 续学，报告进度并继续。
- 没有 → 询问用户想学什么，进入初始化流程。

### 分支 B：参数是一个主题（`/learning-loop React`）
1. 跳过"想学什么"的提问，直接以该主题初始化。
2. 仍要询问目标水平（aware/practitioner/expert，默认 practitioner）。
3. 创建 `<主题slug>-learning/` 工作区，生成基线测评，按 skill 的上传约定停下等用户填写。
4. 例外：如果当前目录已存在**同名主题**的 workspace，转为续学而非重建。

### 分支 C：参数是 `status`（`/learning-loop status`）
只读不进入学习：
1. 扫描当前目录的所有 `*-learning/meta.json`。
2. 列出每个 workspace 的：主题、当前阶段/章节、phase、最近一次章节得分。
3. 不修改任何文件，不进入学习流程。

### 分支 D：参数是 `upgrade`（`/learning-loop upgrade`）
**升级 skill 本体 + 迁移工作区**，一步到位。只做"拉取 + 复制 + 标记"，绝不破坏用户数据、不进入学习流程。

**第 1 步：拉取最新 skill（更新工具本体）**
1. 定位 skill 安装目录 `skillDir = ~/.agents/skills/learning-loop`。
2. 判断是否 git 克隆（`$skillDir/.git` 是否存在）：
   - **是克隆**：读当前 `version`（`$skillDir/SKILL.md` frontmatter）记为 `旧版本`；在 `$skillDir` 执行 `git pull --ff-only`（远端 https://github.com/kaikaixiaotian/learning-loop.git ）。
   - **不是克隆**（旧式复制安装）：把 `$skillDir` 备份为 `$skillDir.bak.<时间戳>` 后删除，重新 `git clone https://github.com/kaikaixiaotian/learning-loop.git "$skillDir"`；`旧版本 = "(复制式安装)"`。
3. **同步命令自身**：把 `$skillDir/commands/learning-loop.md` 复制到 `~/.zcode/commands/learning-loop.md`。
4. 若 `git pull --ff-only` 因本地改动/分叉失败，**不要 `--force` 或强制覆盖**，把 git 错误原样转告用户并停止，**不要**继续第 2 步。

**第 2 步：迁移工作区（让旧工作区后续生成遵循新规范）**
读 `references/upgrade.md`：
1. 扫描当前目录所有 `*-learning/meta.json`。
2. 对每个写入 `schema_version`（取更新后的 skill `version`）+ `upgraded_at` + history 事件。
3. **不改任何已有文件**（旧测验 HTML 缺 quizKey/restoreData 等也维持原样；批改时 AI 回退读题判断，仍能工作）。
4. 后续新生成的章节/测验自动按当前规范（quizKey + restoreData + restore JS + feedback 槽位）。

**第 3 步：报告**
- skill：读更新后的 `version` 记为 `新版本`；`新版本 != 旧版本` → 「✅ skill 已从 v旧版本 升级到 v新版本」，相同 → 「✅ skill 已是最新 v新版本」。
- 工作区：列出迁移的 workspace 数（或「当前目录无学习工作区，仅升级 skill 本体」）。
- 提示「请新开一个会话以加载最新 skill」。

## 通用约束（所有分支都要遵守）

- 严格遵循 `learning-loop` skill 的全部规则：六题型齐全、选择+填空 ≤50%、评分用 grading.md rubric、失败最多 v3、章节通过后派子 agent 写 wiki + 规划下一章。
- 每次状态变更后用一行汇报当前阶段/章节/phase 和下一步动作，不要长篇。
- 学习材料（`*-learning/` 工作区）始终建在**当前工作目录**下，而非用户主目录。
- `upgrade` 分支是维护操作：只拉取/复制/标记，不进入学习流程、不修改已有学习文件。
