# Changelog

记录 `learning-loop` skill 的版本变更。版本号以 `SKILL.md` frontmatter 的 `version` 字段为**单一来源**。
参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/) 与 [语义化版本](https://semver.org/lang/zh-CN/)。

## [1.5.0] - 2026-08-27

### 变更（测验客观化：预填表单只出选择+填空，答题时间还给学习）

用户反馈：基线/章节/阶段测验中大量简答类题目让作答时间远超学习时间。旧「六题型底线规则」（每卷 ≥1 每型、选择+填空 ≤50% 分数）强制一半分数来自实战/模拟/算法/综合四类长文字 textarea 题，是耗时根源。

- **预填测验全客观化**：基线测评与章节测验**只允许选择题（单/多选）+ 填空题**，禁止任何 textarea 大题；阶段总测验在选择/填空为主体之外，允许**至多 1 道**可选文字综合题（跨章、rubric 批改、可不设）。`quiz-types.md` 底线规则整体反转——新禁令是"若发现自己在预填表单里写长文字大题，停下重写为带干扰项的客观题"。
- **客观化转换指南**（保深度不掉）：原四类主观能力的等效客观考法沉淀为对照表与硬性干扰项质量标准——实战→代码输出判断/实现辨析；模拟→情境最佳做法单选（选项皆可辩护、讲评展开 trade-off）；算法→推导结果/关键步骤选择+关键值填空；综合→跨 KP/跨章组合多选（阶段总测验要求 ≥2 道跨章组合多选，替代旧"≥2 综合"配额）。每个错误选项必须对应章节里教过的真实误解或 ⑤边界条件陷阱。
- **题量放开为覆盖驱动**：删除固定题数区间（基线 15–25、章节 6–10、阶段每章 2–3 全部废除），改由考点断言清单全覆盖决定——宁可多几道秒答的客观题，也不出一道让用户写十分钟的大题；保留易→难梯度与弱点复测。
- **plan-quiz 保持不变**：章节后的 chat 现场问答仍用原六题型（实战/模拟/算法/高难度为主体），它是主观深问的唯一阵地——一次一题、对话作答，检测真实迁移能力；合并分权重（0.45/0.55）与通过线（≥0.80）不动。
- **机制零改动**：分值记法 `(题型, X分)`、`data-type` 枚举（新增题目只用 选择|多选|填空，阶段文字题为 综合）、quizKey/answers.json 结构、超纲判定、补讲补考、重建上限 v3 全部向后兼容；旧版六题型测验照常批改。
- 同步更新：`SKILL.md`（初始化/建章/阶段流程与 non-negotiable 节、版本号 1.5.0）、`subagent-protocol.md`（Job 2/3 内联底线规则、QUIZ HTML RULES、弱产出处置）、`templates.md`（基线/章节/阶段模板重排 + quiz-form 骨架示例）、`html-format.md`（Mode B 控件映射与 rubric 使用范围）、`grading.md` 开篇口径、`commands/learning-loop.md` 通用约束、`upgrade.md` 升级后行为清单、`README.md` 特性描述。
- 遵循升级原则：不改动已生成工作区的既有文件，自下一次生成的测验起生效。

## [1.4.1] - 2026-08-14

### 修复
- **`learning-loop upgrade` 被误判为"仅工作区迁移"**：完整升级协议（先 `git pull` 更新 skill 本体 + 同步命令文件，再迁移工作区）此前只写在 slash command 文件里；用户不走斜杠命令、直接输入 `learning-loop upgrade` 时，模型只加载 SKILL.md，而 SKILL.md 的索引与 `references/upgrade.md` 都只描述工作区迁移（upgrade.md 甚至自述"本文件只覆盖第 2 步"），导致每次都跳过 skill 更新。现收敛为单一来源：`references/upgrade.md` 重写为完整三步协议（更新本体 → 迁移工作区 → 报告）；SKILL.md 的触发描述、参考索引与新增「Upgrade requests」正文路由都明确"升级 = 先更新 skill 本体，绝非仅迁移工作区"；命令文件分支 D 改为引用同一协议。
- `schema_version` 标记值从日期改为取 skill frontmatter `version`（与命令文件口径一致）。

## [1.4.0] - 2026-08-14

### 变更（教学规范 2.0：取消类比 → 全覆盖交互动画 + 断言对齐）

基于 java-backend-learning 实测证据的三个根因修复：正文要素内堆砌成段（①一段塞 4 论断、⑤内联 a)b)c)、②叠 3 个类比）；丢分集中在被埋没的边界条件/精确事实；每章 8 个概念仅 3 个演示且缺演示的概念恰是历史丢分点。

- **废除类比教学**：六要素的 ②直觉解释（类比/心智模型）移除，改为 **②直观演示**——内嵌交互动画（`<figure class="viz">` iframe 移入②要素位，骨架 CSS 使其破格全宽）+ 2–3 条「观察要点」（点什么/看什么/验证哪条断言）。全文禁止比喻类文案（"像遥控器/像仓库"句式）。
- **演示默认必做**（`visualization.md` 判定规则反转）：旧「≥2 信号才画、每章 0–3 个、零演示正常」废除；每个核心概念默认配演示（预期每章 5–8 个），仅纯记忆型 KP 可豁免且须在 `visualization_decisions` 写明理由。原 6 信号表降级为**演示类型选择器**。
- **「真正的演示」质量标准**（硬性）：演示机制本身（真实实体显式画出、逐步执行状态可见变化）；必须覆盖该 KP 关键分支**含至少一个⑤边界条件 case**；用户可改参数自行试错；下一步+重置必备；`branches_covered` 写进返回块供主 agent 抽查。
- **考点断言清单**（治"讲得浅考得深"）：知识点清单升级——每 KP 列 3–6 条可考断言（`KP1-A1…`），⑤每个 case 必须有对应断言。测验 fieldset 增加 `data-assert`，quizKey 增加 `assert` 字段；覆盖度校验与超纲判定升级为**断言级**（旧测验回退 KP 级），出题阶段即拦截超纲而不是批改才发现。补讲时新断言回填进清单。
- **排版微格式**（治正文堆砌，硬约束）：①每条独立规则一个 `<li>`（≤2 句，并列情形加粗分条）；⑤必须 `<ul>` 一 case 一条（场景→结果→一句原因），禁止内联 a)b)c)；⑥ ≥2 概念×≥2 维度必须对比表格；el-body 段落禁止内联枚举。骨架占位符改为结构化示例，新增要素内列表/表格 CSS。
- **检查点（非计分自测）**：每个概念六要素后加 `<section class="checkpoint">`，2–3 道 `<details><summary>` 展开答案的自测题（零 JS）：预测（先作答再到②演示验证）/判断+说理由/填关键值，答案含推理+回看指引。章节顶部加学习路线 pill：通读→玩演示+做检查点→回看→再开测验。
- **生成与校验链路同步**：`subagent-protocol.md` Job 2/3 prompt 内联全部新规则，自检新增格式门/演示覆盖门/污染防护；`html-format.md` 静态校验新增断言覆盖、格式门、`--vscode-`/icube 污染防护（真实事故：一次生成的章节混入约 1900 行 IDE 主题 CSS）；`SKILL.md` step 5/6 与学习/批改流程同步；`grading.md` 超纲判定断言级化。
- 遵循升级原则：不改动已生成工作区的既有文件，自下一次生成的章节起生效。

## [1.3.3] - 2026-08-13

### 修复
- **演示区自适应高度**：交互演示（viz）不再被固定高度裁切。演示文件的 `reportHeight()` 把自身实际高度 `postMessage` 给父页（load/resize/DOM 变化时上报），章节页监听后把对应 iframe 高度调到刚好（300–1200px 钳位）。`file://` 双击打开同样生效——postMessage 不受同源 DOM 限制（不像读 `contentDocument` 会被挡）。
- `templates.md`：章节骨架加自适应监听脚本（read-mode 唯一 JS，仅为演示可用性）+ iframe 默认高度 380→460；viz 骨架加 `reportHeight()` 片段；两处骨架的 non-negotiables 与 `visualization.md` 注明「必须保留该片段，禁止写死过小高度」。

## [1.3.2] - 2026-08-13

### 新增
- **补讲纳入左侧目录（可快捷跳转）**：超纲/补充教学（补讲）现在有标准骨架——`<h2 id="sec-backfill">补讲</h2>` 下每个补讲为 `<h3 id="backfill-<slug>">标题 <span class="backfill-badge">补讲</span></h3>` + 来源 meta（KP·日期·来源）+ 六要素 `<ol class="elements">`；左侧 `<aside class="toc">` 末尾按「补讲」分组追加 `<a href="#backfill-<slug>">` 跳转链接。
- `templates.md` 增补讲 CSS（琥珀徽章 / meta / toc-sub / `::before` 前缀）与注释化示例；`SKILL.md`、`grading.md` 的补讲指令补全骨架约定（id + 徽章 + 六要素 + 目录跳转 四件套）。

## [1.3.1] - 2026-08-13

### 修复
- **修复"骨架不随 skill 更新"的根因**：生成/重建章节与测验时，AI 会从已有兄弟文件复制旧骨架、而非读取最新模板（真实案例：1.3.0 升级后生成的 stage2 章节仍是旧骨架）。现强制：每次生成与每次重建都必须从当前 `references/templates.md` 取骨架 + `<style>`，**禁止**从兄弟 `chapters/*.html` / `quizzes/*.html` 复制结构或样式。
- **骨架签名 + 交付前静态校验**：模板 `<head>` 埋 `<!-- learning-loop skeleton: read-mode|quiz-form -->` 签名；交付任何章节/测验前必须 grep 该签名，缺失 = 抄了旧兄弟 = 立即按模板重生成（复用已验证的 viz 校验模式）。
- 该规则写入 `SKILL.md`（生成 / rebuild / verify 三处）、`templates.md`（两骨架顶部）、`html-format.md`、`subagent-protocol.md`、`upgrade.md`。

## [1.3.0] - 2026-08-13

### 变更
- **重新设计章节学习页（read-mode）**：精致阅读型——共享设计系统（CSS 变量 + 自动深色模式）、左侧粘性目录、六要素从挤压的 `<p>` 改为可扫读的 `ol.elements` 定义列表、四种 callout 卡片统一。
- **演示改为内嵌组件**：viz 从指向独立文件的 `<a class="viz-link">` 改为 `<figure class="viz">` 内嵌 `<iframe>`（演示文件仍是 `viz/` 下独立可复用的 `.html`），用户学习时直接在章节页内交互、不再跳转。仅对决定要画的 KP 渲染 `<figure>`。
- **重新设计答题页（quiz-form）**：仅替换 CSS 表现层（卡片化题目、`accent-color` 选项、focus ring、粘性提交条、美化批注与总分横幅），共享同一设计系统 + 自动深色；**所有评分/续学契约逐字节保留**（`body[data-quiz]`/`#quizForm`/`#submitBtn`/`#answerOutput`/`#restoreData`/`#quizKey`/`#gradingSummary`/`#fb-qN` 空形态、`name=id=qid`、submit+restore JS 原样、`.feedback.shown`+四 verdict 类）。
- 同步更新 `visualization.md`、`html-format.md`、`subagent-protocol.md`、`SKILL.md`、`upgrade.md` 中演示嵌入与六要素的描述。

## [1.2.0] - 2026-08-13

### 变更
- **合并自更新进 upgrade**：移除 `/learning-loop update`，其能力（从 GitHub 拉最新 skill + 同步命令）并入 `/learning-loop upgrade`。`upgrade` 现一步完成「拉新版本 skill → 迁移工作区」。
- **恢复学习命令分支**：`/learning-loop` 重新支持 init（空/主题）、`status`（查进度）、`upgrade`（升级）；frontmatter 恢复 `skills: learning-loop`。
- README「安装与更新」改为「命令用法」表，列全四个分支。
- `references/upgrade.md` 顶部注明 upgrade 先拉取最新 skill 再迁移工作区。

## [1.1.0] - 2026-08-13

### 新增
- **`/learning-loop update` 自更新命令**：输入该命令自动从 GitHub（kaikaixiaotian/learning-loop）拉取最新代码，更新本地 skill 并同步命令自身。
- **`commands/learning-loop.md`**：slash command 源，随仓库版本化管理。

### 变更
- 安装/更新方式改为 **git 克隆 + `/learning-loop update`**（取代 1.0.0 的"复制目录"）。安装目录 `~/.agents/skills/learning-loop/` 即 GitHub 仓库的克隆，更新 = `git pull`。
- README 新增「安装与更新」使用指导。

## [1.0.0] - 2026-08-13

首版发布。

### 能力
- **闭环学习系统**：基线测评 → 总目录规划 → 章节学习 + 六题型测验 → 批改评分 → 错题重构 → 阶段总测验 → wiki 沉淀。
- **六种必考题型**与难度阶梯（选择+填空 ≤50%），`grading.md` rubric，失败最多 v3 的重构循环。
- **网络驱动的课程路径规划**：权威源 + 三道真实性闸门 + 失败降级标注。
- **全 HTML 交付**：阅读态教材 + 表单态测验 + `answers.json` 提交 + restore-on-load 回填。

### 项目结构
- 仓库根即 skill 根：`SKILL.md` + `README.md` + `references/`（10 个参考文档），对齐 `prototype-review` 约定（纯 skill，无部署脚本、无 slash command 绑定）。
- 版本单一来源为 frontmatter `version` 字段，不再依赖独立 `VERSION` 文件；安装方式为复制目录，与 `prototype-review` 一致。
