---
description: AI 闭环学习——初始化/续学（学习模式/刷题模式）、查进度、升级（拉取最新 skill + 迁移工作区）
argument-hint: "[主题 | 刷题 主题 | status | upgrade]  留空=自动判断；写主题=直接初始化；刷题 <主题>=刷题模式初始化；status=只查进度；upgrade=升级 skill 与工作区"
skills: learning-loop
---

使用 `learning-loop` skill 处理本次学习请求。

$ARGUMENTS

## 按参数分支执行

根据上面的参数（可能为空、是一个主题、`status`、或 `upgrade`），选择对应分支：

### 分支 A：参数为空（`/learning-loop`）
按 skill 的「First run vs. resume」逻辑自动判断：
- 当前目录已有 `*-learning/meta.json` 或 `*-drill/meta.json` → 按 meta.json 的 `mode` 分流：学习模式续学、刷题模式续刷，报告进度并继续。
- 没有 → 询问用户想学什么，并询问模式（学习模式默认 / 刷题模式），进入对应初始化流程。

### 分支 B：参数是一个主题（`/learning-loop React`，或 `刷题 React`）
1. 跳过"想学什么"的提问，直接以该主题初始化。参数以 `刷题` 开头（如 `/learning-loop 刷题 React`）→ 走刷题模式（第 3 步）；否则学习模式（第 2 步）。
2. 学习模式：仍要询问目标水平（aware/practitioner/expert，默认 practitioner）；创建 `<主题slug>-learning/` 工作区，生成基线测评，按 skill 的上传约定停下等用户填写。
3. 刷题模式：按 skill「Drill-mode flow」初始化——询问是否立即提供真题（粘贴文本 / 真题库链接（派 Job 6 子代理采集）/ 暂无则 AI 自主出题），创建 `<主题slug>-drill/`（仅 `meta.json` + `题库/`），随即创建 `题目-001/` 出第一题，停下等用户作答。
4. 例外：如果当前目录已存在**同名主题**的 workspace（`*-learning/` 或 `*-drill/`），转为续学/续刷而非重建。

### 分支 C：参数是 `status`（`/learning-loop status`）
只读不进入学习：
1. 扫描当前目录的所有 `*-learning/meta.json` 与 `*-drill/meta.json`。
2. 列出每个 workspace 的：主题、模式（学习/刷题）、当前阶段/章节或已刷轮数、phase、最近一次章节得分或题库掌握数。
3. 不修改任何文件，不进入学习流程。

### 分支 D：参数是 `upgrade`（`/learning-loop upgrade`）
**升级 skill 本体 + 迁移工作区**。完整协议以 skill 的 `references/upgrade.md` 为**唯一权威来源**——读它并按三步执行，不要凭记忆改写步骤：

1. **更新 skill 本体（必须先做）**：定位安装目录 `skillDir = ~/.agents/skills/learning-loop`。有 `.git` → 在 `$skillDir` 执行 `git pull --ff-only`（远端 https://github.com/kaikaixiaotian/learning-loop.git ）；无 `.git`（复制式旧安装）→ 备份 `$skillDir.bak.<时间戳>` 后重新 `git clone` 该远端。**pull 失败（本地改动/分叉）时原样转告 git 错误并停止，禁止 force/强制覆盖，不要继续第 2 步。** 随后把 `$skillDir/commands/learning-loop.md` 复制到 `~/.zcode/commands/learning-loop.md`（同步命令自身）。
2. **迁移工作区**：按 upgrade.md 给当前目录每个 `*-learning/meta.json` 与 `*-drill/meta.json` 写 `schema_version`（取更新后的 skill `version`）+ `upgraded_at` + history 事件。**不改任何已有文件**（旧测验 HTML 缺 quizKey/restoreData 等也维持原样；批改时 AI 回退读题判断，仍能工作）。
3. **报告**：读更新后的 `version` 记为 `新版本`；`新版本 != 旧版本` → 「✅ skill 已从 v旧版本 升级到 v新版本」，相同 → 「✅ skill 已是最新 v新版本」；列出迁移的 workspace 数（或「当前目录无学习/刷题工作区，仅升级 skill 本体」）；提示「请新开一个会话以加载最新 skill」。

## 通用约束（所有分支都要遵守）

- 严格遵循 `learning-loop` skill 的全部规则：预填测验客观化（基线/章节测验仅选择+填空、阶段总测验至多 1 道文字题，题量按断言清单全覆盖自定）、plan-quiz 保留深度文字问答（一次一题、漏答必追问）、评分用 grading.md rubric、失败最多 v3、章节通过后派子 agent 写 wiki + 规划下一章。
- 每次状态变更后用一行汇报当前阶段/章节/phase 和下一步动作，不要长篇。
- 学习材料（`*-learning/` 工作区）始终建在**当前工作目录**下，而非用户主目录。
- 刷题模式（`*-drill/` 工作区）遵循 skill「Drill-mode flow」：知识中心闭环——跟踪与掌握单位是**知识点**，题目只是载体；一次一题（每轮新建 `题目-NNN/`），题型按知识点 stage 爬梯：选择→填空→应用（须说出准确原因）→变种；掌握由 AI 综合判断（无固定答对次数规则），判断理由写进批阅区；答错随机排期复现，已掌握知识点随机排期防遗忘复习（原题/变种可重复出）；知识点到 stage 4 或掌握后即可基于真题出变种加难；用户可随时补充真题（粘贴直接入库，链接派 Job 6 子代理采集）；每题点评必附正确代码示例（无论对错，取自题库存档的答案区）。
- `upgrade` 分支是维护操作：只拉取/复制/标记，不进入学习流程、不修改已有学习文件。
