# Document Templates

Every file the learning-loop system writes follows one of these templates. Keep them consistent so the user (and future subagents) can always find the same sections in the same place.

## FORMAT NOTE — user-facing files are HTML (read `references/html-format.md` first)

All **user-facing** artifacts are now standalone **HTML** files, not markdown:
- **Chapter doc** → `chapters/stageN-chXX-<slug>.html` (read-mode)
- **Master plan** → `plan/master-plan.html` (read-mode)
- **Baseline, chapter-quiz, stage-total-quiz** → `.html` quiz-forms (radio/checkbox/textarea + submit button → answers.json)

The two authoritative HTML skeletons are at the **end of this file** (sections "read-mode html skeleton" and "quiz-form html skeleton"). Generate from those.

The markdown templates below (baseline-assessment.md, chapter doc, chapter-quiz, stage-total-quiz) are the **degradation fallback** — use them ONLY when an HTML file repeatedly fails JS verification (see `references/html-format.md` graceful degradation). They also document the *content structure* (sections, six-element concepts, KP list, question types) that the HTML skeletons must preserve — so read them for "what content goes where", then render that content into the HTML skeleton.

**Unchanged (stay markdown — internal AI use, not user-facing):** per-chapter material card (`plan/sources/*.md`), plan-quiz receipt (`stageN-chXX-plan-quiz.md`), wiki files, meta.json.

## baseline-assessment.md

Located at `00-baseline/baseline-assessment.md`. The user fills this in and uploads it.

```markdown
# 基础测评 — <topic>

> 说明：本测评用于定位你当前的起点，**不影响通过与否**。请尽力作答；完全不会的题写「不知道」即可。所有题型都将贯穿你后续的学习，请熟悉它们的格式。

## 个人背景
- 你之前接触过 <topic> 吗？到什么程度？（自学/课程/工作/完全没有）
- 你学这个的目标是什么？（如：能解决实际问题 / 通过面试 / 能教别人）
- 每周可投入学习时间？

## 一、选择题（选择 / 多选）
1. (单选) ……？
   - A. …
   - B. …
   - C. …
   - D. …
   - **你的答案：**

## 二、填空题
1. ……中的关键概念是 _____。
   - **你的答案：**

## 三、实战题
1. （给定具体任务/数据/代码场景）请写出你的解决步骤或代码。
   - **你的答案：**

## 四、模拟题
1. （场景角色扮演，如「假设你是 X，面对 Y 情况，你会如何处理？」）
   - **你的答案：**

## 五、算法 / 推导题
1. （推导一个结论 / 设计一个流程 / 写出算法步骤）
   - **你的答案：**

## 六、高难度综合题
1. （跨章节综合、开放性、需要多步推理）
   - **你的答案：**
```

Baseline should have ~15–25 questions total, spread so easy and hard both appear. The goal is to locate the user's level, not to grade harshly.

## master-plan.md (总目录)

Located at `plan/master-plan.md`. The roadmap for the whole topic. **Now grounded in an external canonical learning path** (pulled by the curriculum-research subagent), then adapted to the user's baseline — NOT invented from AI memory. See `references/curriculum-research.md`.

```markdown
# 学习计划总目录 — <topic>

- **目标水平**：<target level>
- **基线测评得分**：<score> / 1.0
- **基线画像**：<one-paragraph strengths & gaps>
- **规划依据**：<which external canonical path this follows, e.g. "基于 react.dev/learn 官方学习路径 + roadmap.sh/frontend">
- **数据增强状态**：✅ 已基于外部路径 / ⚠️ 降级（网络不可达，纯 AI 推断，结构与公认曲线可能不符）
- **生成时间**：<ts>

## 规划来源（外部学习路径）
- [1] <url> — <来源类型：官方学习路径/路线图/书目录> · 提供了<主骨架/补充>
- [2] <url> — …

## 适配说明（AI 对外部骨架做的调整 + 理由）
- 拆分/合并：将官方第 X 章拆为两章，因为<基线/目标水平理由>
- 补强：新增「<章节>」章（标记 补强，不在原始路径），因为基线显示用户缺<前置知识>
- 跳过：target_level=aware，跳过最深章节「<章节>」
- 顺序：保持官方顺序（或：调整 X 至 Y 前，因为<前置逻辑>）
（无偏离时写：完全遵循外部路径，未做结构性调整）

## 阶段 1 — <stage name>  [出处: <url>]
- **目标**：<stage objective>
- **章节**：
  1. `<slug>` — <one-line objective>（题型侧重：<types>） [出处: <url> 或 AI推断]
  2. …
- **资料卡**：见 `plan/sources/stage1-chXX.md`
- **阶段测验**：覆盖以上全部章节的综合测验

## 阶段 2 — <stage name>  [出处: <url>]
…

## 进度总览
| 阶段 | 章节 | 状态 | 章节测验 | 计划测验 | 文档版本 |
|------|------|------|----------|----------|----------|
| 1 | 1 | ⏳进行中 | — | — | v1 |
```

## per-chapter material card — `plan/sources/stageN-chXX.md`

Written by the curriculum-research subagent during initialization (one per chapter). NOT teaching prose — a reference summary the chapter-generation subagent Reads to ground its content. See `references/curriculum-research.md`.

```markdown
# 章节资料卡 — 阶段< N >·章节< XX > <title>

> 来源：<url> (<type>) · 抓取于 <date>
> 本卡供章节生成子 agent 参考，不是教学正文。

## 该主题的官方要点
- <key point 1, from source>
- <key point 2>
- …

## 官方强调的易错点
- …

## 建议的教学顺序（来自官方教程）
- 先讲 X，再讲 Y，因为 …

## 引用
- <url>
```

## chapter doc — `chapters/stageN-chXX-<slug>.md`

The actual teaching material. Rebuilt versions get `_v2`, `_v3`.

```markdown
# 阶段< N > · 章节< XX > — <title>

> 版本 v< version > · 前置：<prev chapter or none> · 预计学习时间：<X>min
> 本章节目标：学完后你应当能 <1–3 条可验证的能力>

## 引入
（一个真实问题或反直觉现象，2–4 句话勾起为什么需要学这个）

## 知识点清单（本章覆盖度基准）
> 本节是**章节测验出题范围的唯一基准**。测验的每个考点必须能映射回这里的一项；不在此清单的内容不得作为计分题（见 `references/grading.md` 超纲规则）。生成测验前先核对这份清单，生成后再次核对覆盖度。
- **KP1**：<一句话知识点>（关联概念节：§核心概念.1）
- **KP2**：<一句话知识点>（关联：§核心概念.2）
- **KP3**：<…>
- **KP4**：<…>
（建议 4–8 个知识点；粒度以"能出一道独立小题"为准。）

## 核心概念
### 1. <concept>
**① 精确定义**（技术层面：公式 / 函数签名 / 语法 / 语义规则。不能只用类比搪塞——类比放②，定义必须有可查证的精确表述。）
**② 直觉解释**（类比 / 心智模型，帮助理解而非替代定义）
**③ 最小例子**（能跑/能验证的最小用例，标注输入输出）
**④ 推导或代码**（这个概念如何落地：算法逐步推导，或代码逐行注释，或状态流转图。不能只给结论。）
**⑤ 边界条件**（什么时候适用 / 什么时候失效 / 极端输入会怎样 / 性能边界）
**⑥ 与相关概念对比**（和易混淆的 X 有什么区别？何时该用本概念而非 X？）

### 2. <concept>
（同样六要素。每个核心概念都必须六要素齐全——不允许只给类比不给定义，也不允许只给定义不给边界。）
（**可选**：若该概念适合图形化演示，在⑥对比后插入 🖼️ 交互演示链接，见下方「图形化演示嵌入」说明。）

## 图形化演示嵌入（可选，按概念判定）
> 不是每个概念都要画。对每个 KP 用 `references/visualization.md` 的判定准则（≥2 信号才画）决定是否生成。决定要画的，存为 `chapters/viz/stageN-chXX-<kp-slug>.html`（独立文件，可交互，vanilla JS，无外部依赖），在对应概念节内嵌链接：
>
> ```markdown
> > 🖼️ **交互演示**：[<一句话演示名>](./viz/stageN-chXX-<kp-slug>.html) — <用户能看到/做到什么>。浏览器打开即可。
> ```
>
> 决定不画的 KP 不加占位、不加链接——静默即可。零演示的章节完全正常。
> 主 agent 生成后必须按 `references/visualization.md` 跑 JS 静态检查，不过的要么重生成要么丢弃改文字。

## 实战演示
（端到端走一个例子，展示核心概念如何协同落地。如果章节偏算法，这里给出完整推导/代码；如果偏实操，给出可复现的命令序列 + 预期输出。这一节要能让用户照着做一遍。）

## 常见陷阱 & 易错点
- …（每条对应一个知识点 KP，标注关联）
- …

## 小结 & 自查
- 三个关键 takeaway（对应最重要的 3 个 KP）
- 自测：对照「知识点清单」，你能否对每一项给出定义+例子？答不上来的回去看对应概念节。

## 下一步
学完填写 `quizzes/stageN-chXX-quiz.md` 并上传。
```

**六要素是非妥协项**：每个核心概念必须六项齐全。常犯的偏差是"类比写得很生动，但精确定义缺失或含糊"——这会让用户在测验的填空/算法题上失分（题要的是精确值/签名，脑子里只有类比）。另一个偏差是"只给定义不给边界"，导致实战题失分。六要素逐一覆盖这两类失分。

Calibration rule: if `baseline_score` is low, this doc leans harder on ②直觉 + ③例子，把步骤拆得更细，jargon 首次出现必给定义；但 ①精确定义 和 ⑤边界条件 仍然不可省略——只是用更通俗的话重述，不能跳过。If high, it can be denser and assume prior vocabulary.

## chapter-quiz.md — `quizzes/stageN-chXX-quiz.md`

```markdown
# 章节测验 — 阶段< N >·章节< XX > <title>

> 通过线：与计划测验合并 ≥80%。填写后上传本文件。
> 覆盖六大题型，请逐题作答。
> 每题标注「考点: KP-x」——KP 编号对应章节文档的「知识点清单」。出题前 AI 已核对所有考点都在清单内；若你发现某题考点不在清单，按超纲规则不计分（见批阅区说明）。

## 一、选择题
1. (选择题, 1分) [考点: KP-2] …
   - A. … / B. … / C. … / D. …
   - **你的答案：**
   - **理由：**（简述）

## 二、填空题
1. (填空题, 1分) [考点: KP-1] … _____ …
   - **你的答案：**

## 三、实战题
1. (实战题, 4分) [考点: KP-3] （明确任务 + 输入 + 期望输出格式）
   - **你的答案：**

## 四、模拟题
1. (模拟题, 4分) [考点: KP-4] （场景：…… 你会如何 ……）
   - **你的答案：**

## 五、算法 / 推导题
1. (算法题, 5分) [考点: KP-2] （请推导/设计 ……）
   - **你的答案：**

## 六、高难度综合题
1. (综合题, 6分) [考点: KP-1, KP-3] （综合 …… 与 …… 解决 ……）
   - **你的答案：**

---
*AI 批阅区（用户请勿填写）*
| 题号 | 类型 | 考点 | 正确？ | 计分？ | 失分点 |
|------|------|------|--------|--------|--------|
| 1 | 选择 | KP-2 |  | 是 |  |
| 2 | 填空 | KP-1 |  | 是 |  |
| 3 | 实战 | KP-3 |  | 是 |  |
| … | | | | | |
（"计分？"列：是=正常计分；**超纲=不计分**——考点不在章节知识点清单内时填此项，该题从分母中剔除，见 grading.md）
**章节测验得分：0.XX（X/Y 分，Y=计分题总分）**（若含超纲题，注明：已剔除 N 道超纲题，另见补讲补考说明）
```

## plan-quiz.md — `quizzes/stageN-chXX-plan-quiz.md`

The plan-quiz is the **transfer check**: it asks questions *deliberately different* from the chapter quiz (new scenarios, edge cases, cross-links) to verify the user can actually *use* what was taught, not just recall it.

**Format**: still live one-at-a-time in chat — the user answers in conversation, NOT by filling a file first. The back-and-forth experience is preserved (asking all questions upfront would defeat the transfer check). But after grading, you **must write the whole round back** to `quizzes/stageN-chXX-plan-quiz.md` so the user has a durable record alongside their chapter quiz.

The file is AI-authored post-hoc (after the live round finishes), not handed to the user to fill. Structure:

```markdown
# 计划测验（迁移检查）— 阶段< N >·章节< XX > <title>

> 本测验在章节测验之后进行，题目与章节测验**完全不同**（新场景/边界/跨概念）。
> 现场逐题问答，本文件为问答结束后 AI 整理写回的记录。

## 现场问答记录
### Q1 (实战题, 4分)
**题目：** <paste the question you asked live>
**你的回答：** <paste the user's answer verbatim>
**AI 讲评：** <one-line verdict + the correct reasoning if they missed>

### Q2 (模拟题, 4分)
…

---
*AI 批阅区*
| 题号 | 类型 | 正确？ | 失分点 |
|------|------|--------|--------|
| Q1 | 实战 | △ | 正确性✓ 过程✗(漏 X) 边界✗ |
| Q2 | 模拟 | ✗ | 判断✗(选了不可辩护的方案) 取舍✗ 论证✗ |
…
**计划测验得分：0.XX（X/Y 分）** · **合并分：0.45×章节 + 0.55×本测验 = 0.XX · 判定：通过 / 未通过**
```

The live round is the experience; this file is the receipt. Both matter — chat scrolls away, the file is what the user reviews when prepping for the stage-total quiz. Also append a `plan_quiz` event (score + dimension breakdowns) to `meta.json` history.

## stage-total-quiz.md — `quizzes/stageN-total-quiz.md`

Comprehensive quiz covering **every chapter** in the stage. Generated when the stage's last chapter passes (not at stage start). **Volume is the point here** — unlike the small chapter quizzes, the stage-total must be substantial because it gates stage advancement. **Must be web-research-augmented** (see `references/web-research.md`): the stage-total planner dispatches a web-research subagent first, then composes the quiz from the returned brief.

**Volume rule** (the user's explicit requirement):
- **Minimum: 2–3 questions per chapter** in the stage, spread across the six types. A 4-chapter stage → ≥12 questions; a 6-chapter stage → ≥15.
- **Each of the six types must have ≥2 questions** (not just ≥1 like chapter quizzes).
- **≥2 综合** that each span ≥2 chapters.
- Questions should lean into the stage's cumulative weak spots recorded in chapter wikis.

```markdown
# 阶段总测验 — 阶段< N > <stage name>

> 覆盖本阶段全部 <M> 个章节。通过线 ≥80%。填写后上传本文件。
> 六大题型齐全且每类 ≥2 题；本测验经网络权威源数据增强，每题标注出处。

> [数据增强状态]  ← 由 web-research 子agent 的结果决定，三选一：
>   ✅ 已增强：全部题目附 [出处: url]
>   ⚠️ 部分增强：标注的题目已验证，[未验证] 的题目需核对官方文档
>   ⚠️ 降级：网络不可达，本测验未经外部验证，事实准确性可能偏低

## 一、选择题（≥2 题）
1. (选择题, 1分) …… [出处: <url>]
   - A. … / B. … / C. … / D. …
   - **你的答案：**
   - **理由：**
2. (选择题, 1分) …… [出处: <url>]
   - **你的答案：**

## 二、填空题（≥2 题）
1. (填空题, 2分) …… [出处: <url>]
   - **你的答案：**
2. (填空题, 2分) …… [未验证]   ← 仅在降级路径出现
   - **你的答案：**

## 三、实战题（≥2 题，基于真实场景/真实 API/真实数据集）
1. (实战题, 5分) ……（综合多个章节的实操任务） [出处: <url>]
   - **你的答案：**
2. (实战题, 5分) …… [出处: <url>]
   - **你的答案：**

## 四、模拟题（≥2 题，跨章节场景）
1. (模拟题, 5分) …… [出处: <url>]
   - **你的答案：**
2. (模拟题, 5分) …… [出处: <url>]
   - **你的答案：**

## 五、算法 / 推导题（≥2 题）
1. (算法题, 6分) …… [出处: <url>]
   - **你的答案：**
2. (算法题, 6分) …… [出处: <url>]
   - **你的答案：**

## 六、高难度综合题（≥2 题，每题跨 ≥2 章节）
1. (综合题, 8分) ……（必须跨 ≥2 个章节综合） [出处: <url>, <url2>]
   - **你的答案：**
2. (综合题, 8分) …… [出处: <url>]
   - **你的答案：**

---
## 引用源（用户可点击核对）
- [1] <url> — <一句话说明此处用于哪题/什么事实>
- [2] <url> — …
（每个用到的 url 都列在此处，便于复查）

---
*AI 批阅区（用户请勿填写）*
| 题号 | 类型 | 正确？ | 出处/验证状态 | 失分点 |
|------|------|--------|--------------|--------|
| 1 | 选择 |  | [出处: url] |  |
| 2 | 填空 |  | [未验证] |  |
| 3 | 实战 |  | [出处: url] |  |
…
**阶段总测验得分：0.XX（X/Y 分）** · **判定：通过 / 未通过**
```

Passing = ≥0.80. On fail, identify the weakest chapter from the 失分点 rows and loop the rebuild flow for that chapter; after it re-passes, issue `stageN-total-quiz_v2.md` (re-research if the failure was on a [出处]-tagged factual question — the source may have been misread).

---

## chapter-quiz.md (filled example slice)

For reference, here's what a graded chapter quiz's AI section looks like. Subjective rows carry per-dimension breakdowns so the user sees *why* they lost points:

```markdown
---
*AI 批阅区*
| 题号 | 类型 | 正确？ | 失分点 |
|------|------|--------|--------|
| 1 | 选择 | ✓ | — |
| 2 | 填空 | ✗ | 混淆了 A 与 B 的定义 |
| 3 | 实战 | △ | 正确性✓ 过程✗(用了 X 而非 Y) 边界✗(漏判空输入) |
| 4 | 模拟 | △ | 判断✓ 取舍✗(没提 trade-off) 论证✗(结论对但理由空) |
| 5 | 综合 | △ | 子问题(a)✓ (b)△(方向对细节错) (c)✗ |
…
**章节测验得分：0.70**（未达单测线，但以合并分判定）
```

The 失分点 column is the part the user actually reviews against. Keep it specific and conceptual — "混淆 A/B" beats "wrong"; "正确性✓ 过程✗" beats "partial".

---

## visualization html skeleton — `chapters/viz/stageN-chXX-<kp-slug>.html`

Standalone, double-click-to-open, vanilla JS, no external deps. Follow this skeleton; the body/interaction adapts to the concept. Full rules in `references/visualization.md`.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title><KP 概念名> — 交互演示</title>
<style>
  body { font-family: -apple-system, "Segoe UI", "Microsoft YaHei", sans-serif; max-width: 760px; margin: 24px auto; padding: 0 16px; color: #1a1a1a; }
  h1 { font-size: 1.3rem; }
  .stage { /* 主可视化区域 */ min-height: 240px; border: 1px solid #ddd; border-radius: 8px; padding: 16px; margin: 12px 0; background: #fafafa; }
  .controls { margin: 12px 0; }
  .controls button, .controls input[type="range"] { padding: 6px 12px; margin-right: 8px; }
  .legend { font-size: 0.85rem; color: #666; margin-top: 8px; }
  /* 状态用颜色区分：新鲜/过期等 */
  .fresh { background: #d4edda; } .stale { background: #f8d7da; }
</style>
</head>
<body>
<h1><KP 概念名> — 交互演示</h1>
<p class="legend"><一句话说明：这个演示让你看到什么、做什么操作></p>

<div class="stage" id="stage">
  <!-- 可视化主体：div/svg/canvas，按概念选择 -->
</div>

<div class="controls">
  <!-- 至少一种交互：button（step/play/reset）、slider（参数）、或 click 高亮 -->
  <button id="btn-next">下一步 ▶</button>
  <button id="btn-reset">重置 ↺</button>
  <label>max-age: <input type="range" id="param" min="0" max="100" value="60"></label>
</div>

<div class="legend" id="status">当前状态：<动态更新这里></div>

<script>
(function(){
  'use strict';
  // —— 状态 ——
  let state = { step: 0, param: 60 };
  const stage = document.getElementById('stage');
  const statusEl = document.getElementById('status');

  // —— 渲染函数：根据 state 重绘 stage + 更新 status ——
  function render() {
    // TODO: 按 state 把可视化画进 stage
    // TODO: 根据 state 更新 statusEl.textContent
    statusEl.textContent = '当前状态：第 ' + state.step + ' 步';
  }

  // —— 交互绑定 ——
  document.getElementById('btn-next').addEventListener('click', function(){
    state.step += 1;
    render();
  });
  document.getElementById('btn-reset').addEventListener('click', function(){
    state.step = 0;
    render();
  });
  document.getElementById('param').addEventListener('input', function(e){
    state.param = parseInt(e.target.value, 10);
    render();
  });

  // —— 初始渲染 ——
  render();
})();
</script>
</body>
</html>
```

**Skeleton non-negotiables** (mirror `visualization.md`):
- All CSS/JS inline; no `<script src>`, no `<link>` to CDN.
- `'use strict'` and an IIFE wrapping the script (no globals leaking).
- At least one bound interaction (button/slider/click) that visibly changes the stage.
- A reset control.
- Chinese labels matching chapter terminology.
- The `render()` pattern: one function that reads `state` and repaints everything; interactions only mutate `state` then call `render()`. This avoids partial-update bugs.

---

## read-mode html skeleton — `chapters/stageN-chXX-<slug>.html` / `plan/master-plan.html`

Standalone, double-click-to-open, vanilla, no deps. For chapter docs and master plan (user reads only, no form). Content structure mirrors the md chapter-doc template (引入/知识点清单/核心概念六要素/🖼️演示/实战/陷阱/小结) — render that content into HTML. Full rules in `references/html-format.md`.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>阶段< N > · 章节< XX > — <title></title>
<style>
  body { font-family: -apple-system, "Segoe UI", "Microsoft YaHei", sans-serif; max-width: 820px; margin: 24px auto; padding: 0 20px; color: #1a1a1a; line-height: 1.7; }
  h1 { font-size: 1.5rem; border-bottom: 2px solid #eee; padding-bottom: 8px; }
  h2 { font-size: 1.2rem; margin-top: 28px; border-left: 4px solid #4a90d9; padding-left: 10px; }
  h3 { font-size: 1.05rem; margin-top: 20px; color: #2c3e50; }
  .meta { color: #666; font-size: 0.9rem; margin-bottom: 16px; }
  .objectives { background: #f0f7ff; border-left: 4px solid #4a90d9; padding: 10px 14px; margin: 12px 0; }
  .kp-list { background: #fffbe6; border: 1px solid #ffe58f; border-radius: 6px; padding: 12px 16px; margin: 12px 0; }
  .kp-list ul { margin: 6px 0; padding-left: 20px; }
  .concept-element { margin: 4px 0; }
  .concept-element strong { color: #4a90d9; }
  pre, code { background: #f5f5f5; border-radius: 4px; }
  pre { padding: 10px; overflow-x: auto; }
  code { padding: 1px 5px; font-size: 0.9em; }
  .viz-link { display: inline-block; background: #e8f5e9; border: 1px solid #81c784; border-radius: 6px; padding: 6px 12px; margin: 8px 0; text-decoration: none; color: #2e7d32; }
  .viz-link:hover { background: #c8e6c9; }
  .pitfall { background: #fff4e5; border-left: 4px solid #ff9800; padding: 8px 12px; margin: 8px 0; }
  .summary { background: #f3e5f5; border-radius: 6px; padding: 12px 16px; margin-top: 20px; }
  details summary { cursor: pointer; font-weight: 600; margin: 12px 0 6px; }
</style>
</head>
<body>
<h1>阶段< N > · 章节< XX > — <title></h1>
<div class="meta">版本 v&lt;version&gt; · 前置：&lt;prev&gt; · 预计学习时间：&lt;X&gt;min</div>
<div class="objectives">
  <strong>本章节目标：</strong>学完后你应当能——
  <ul><li>&lt;capability 1&gt;</li><li>&lt;capability 2&gt;</li></ul>
</div>

<h2>引入</h2>
<p>&lt;一个真实问题或反直觉现象，2-4 句话&gt;</p>

<div class="kp-list">
  <strong>📋 知识点清单（本章覆盖度基准）</strong>
  <p style="font-size:0.85rem;color:#666;">测验出题范围的唯一基准。每个考点必须映射回这里的一项。</p>
  <ul>
    <li><strong>KP1</strong>：&lt;一句话知识点&gt;</li>
    <li><strong>KP2</strong>：&lt;一句话知识点&gt;</li>
    <li><strong>KP3</strong>：…</li>
  </ul>
</div>

<h2>核心概念</h2>
<h3>1. &lt;concept&gt;</h3>
<p class="concept-element"><strong>① 精确定义：</strong>&lt;公式/签名/语法&gt;</p>
<p class="concept-element"><strong>② 直觉解释：</strong>&lt;类比/心智模型&gt;</p>
<p class="concept-element"><strong>③ 最小例子：</strong>&lt;输入输出&gt;</p>
<p class="concept-element"><strong>④ 推导或代码：</strong>&lt;逐步推导/逐行注释&gt;</p>
<p class="concept-element"><strong>⑤ 边界条件：</strong>&lt;何时适用/失效&gt;</p>
<p class="concept-element"><strong>⑥ 与相关概念对比：</strong>&lt;和 X 的区别&gt;</p>
<!-- 可选：交互演示。决定要画的 KP 才加，不画的不加占位 -->
<a class="viz-link" href="./viz/stageN-chXX-<kp-slug>.html">🖼️ 交互演示：&lt;一句话名&gt; — &lt;用户能看到/做到什么&gt;</a>

<h3>2. &lt;concept&gt;</h3>
<p>&lt;同样六要素&gt;</p>

<h2>实战演示</h2>
<p>&lt;端到端例子，可复现命令/推导&gt;</p>
<pre><code>&lt;代码或命令序列 + 预期输出&gt;</code></pre>

<h2>常见陷阱 &amp; 易错点</h2>
<div class="pitfall">&lt;陷阱 1，关联 KP-x&gt;</div>
<div class="pitfall">&lt;陷阱 2&gt;</div>

<div class="summary">
  <strong>小结 &amp; 自查</strong>
  <ul>
    <li>&lt;takeaway 1&gt;</li>
    <li>&lt;takeaway 2&gt;</li>
  </ul>
  <p style="font-size:0.9rem;">自测：对照知识点清单，你能否对每一项给出定义+例子？</p>
</div>

<p style="margin-top:24px;color:#666;">学完请打开对应的 <code>*-quiz.html</code> 测验作答。</p>
</body>
</html>
```

---

## quiz-form html skeleton — `quizzes/*.html` (baseline / chapter-quiz / stage-total-quiz)

Standalone, double-click-to-open, vanilla, no deps. User fills the form, clicks 提交, answers download as `<slug>-answers.json`. Full rules + submit JS in `references/html-format.md`.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>章节测验 — 阶段< N >·章节< XX ></title>
<style>
  body { font-family: -apple-system, "Segoe UI", "Microsoft YaHei", sans-serif; max-width: 820px; margin: 24px auto; padding: 0 20px; color: #1a1a1a; line-height: 1.7; }
  h1 { font-size: 1.4rem; border-bottom: 2px solid #eee; padding-bottom: 8px; }
  h2 { font-size: 1.15rem; margin-top: 28px; border-left: 4px solid #4a90d9; padding-left: 10px; }
  .info { background: #f0f7ff; border-left: 4px solid #4a90d9; padding: 10px 14px; margin: 12px 0; font-size: 0.92rem; }
  fieldset.question { border: 1px solid #e0e0e0; border-radius: 8px; padding: 14px 18px; margin: 14px 0; background: #fafafa; }
  fieldset.question legend { font-weight: 600; color: #2c3e50; }
  .qmeta { font-size: 0.82rem; color: #888; margin-bottom: 8px; }
  label.option { display: block; padding: 4px 0; cursor: pointer; }
  label.option:hover { background: #e3f2fd; border-radius: 4px; padding-left: 6px; }
  textarea { width: 100%; min-height: 100px; padding: 8px; border: 1px solid #ccc; border-radius: 4px; font-family: inherit; font-size: 0.95rem; resize: vertical; }
  input[type=text] { width: 100%; padding: 6px 8px; border: 1px solid #ccc; border-radius: 4px; font-family: inherit; }
  .controls { position: sticky; bottom: 0; background: #fff; padding: 14px 0; border-top: 1px solid #eee; text-align: center; }
  button { padding: 10px 28px; margin: 0 8px; border: none; border-radius: 6px; font-size: 1rem; cursor: pointer; font-weight: 600; }
  #submitBtn { background: #4caf50; color: #fff; }
  #submitBtn:hover { background: #43a047; }
  button[type=reset] { background: #e0e0e0; color: #333; }
  #answerOutput { background: #263238; color: #80cbc4; padding: 12px; border-radius: 6px; font-size: 0.85rem; white-space: pre-wrap; word-break: break-all; margin-top: 16px; }
  /* 逐题批注位（每题 fieldset 内部，AI 批改后填充） */
  .feedback { margin-top: 10px; padding: 8px 12px; border-radius: 6px; font-size: 0.88rem; display: none; }
  .feedback.shown { display: block; }
  .feedback.correct { background: #e8f5e9; border-left: 4px solid #43a047; }
  .feedback.wrong { background: #ffebee; border-left: 4px solid #e53935; }
  .feedback.partial { background: #fff8e1; border-left: 4px solid #ffb300; }
  .feedback.out-of-scope { background: #f3e5f5; border-left: 4px solid #8e24aa; }
  .feedback .verdict { font-weight: 700; }
  .feedback.correct .verdict { color: #2e7d32; }
  .feedback.wrong .verdict { color: #c62828; }
  .feedback.partial .verdict { color: #f57f17; }
  .feedback.out-of-scope .verdict { color: #8e24aa; }
  /* 总分汇总条（批改后显示在提交按钮下方） */
  #gradingSummary { margin-top: 20px; padding: 14px 18px; border-radius: 8px; background: #e3f2fd; font-size: 1.05rem; font-weight: 600; }
  .src-tag { font-size: 0.78rem; color: #1976d2; }
</style>
</head>
<body data-quiz="stageN-chXX-quiz">

<h1>章节测验 — 阶段< N >·章节< XX > &lt;title&gt;</h1>
<div class="info">
  通过线：与计划测验合并 ≥80%。每题标注考点 KP。<br>
  <strong>作答方式</strong>：选择题点选项，问答/实战题在输入框作答。完成后点底部「提交答案」，会自动下载 <code>&lt;quiz&gt;-answers.json</code>，然后在聊天里告诉 AI「做好了」。
</div>

<form id="quizForm" action="">

  <h2>一、选择题</h2>
  <fieldset class="question" data-qid="q1" data-kp="KP-2" data-type="选择" data-points="1">
    <legend>1. &lt;题干&gt;</legend>
    <div class="qmeta">[考点: KP-2] · (选择题, 1分)</div>
    <label class="option"><input type="radio" name="q1" value="A"> A. &lt;option&gt;</label>
    <label class="option"><input type="radio" name="q1" value="B"> B. &lt;option&gt;</label>
    <label class="option"><input type="radio" name="q1" value="C"> C. &lt;option&gt;</label>
    <label class="option"><input type="radio" name="q1" value="D"> D. &lt;option&gt;</label>
    <div class="feedback" id="fb-q1"></div>
  </fieldset>

  <fieldset class="question" data-qid="q2" data-kp="KP-3" data-type="多选" data-points="2">
    <legend>2. &lt;题干&gt;（多选）</legend>
    <div class="qmeta">[考点: KP-3] · (选择题[多选], 2分)</div>
    <label class="option"><input type="checkbox" name="q2" value="A"> A. &lt;option&gt;</label>
    <label class="option"><input type="checkbox" name="q2" value="B"> B. &lt;option&gt;</label>
    <label class="option"><input type="checkbox" name="q2" value="C"> C. &lt;option&gt;</label>
    <label class="option"><input type="checkbox" name="q2" value="D"> D. &lt;option&gt;</label>
    <div class="feedback" id="fb-q2"></div>
  </fieldset>

  <h2>二、填空题</h2>
  <fieldset class="question" data-qid="q3" data-kp="KP-1" data-type="填空" data-points="1">
    <legend>3. &lt;题干&gt; _____</legend>
    <div class="qmeta">[考点: KP-1] · (填空题, 1分)</div>
    <input type="text" id="q3" placeholder="你的答案">
    <div class="feedback" id="fb-q3"></div>
  </fieldset>

  <h2>三、实战题</h2>
  <fieldset class="question" data-qid="q4" data-kp="KP-3" data-type="实战" data-points="4">
    <legend>4. &lt;题干：明确任务+输入+期望输出&gt;</legend>
    <div class="qmeta">[考点: KP-3] · (实战题, 4分)</div>
    <textarea id="q4" placeholder="在此作答（可换行）"></textarea>
    <div class="feedback" id="fb-q4"></div>
  </fieldset>

  <h2>四、模拟题</h2>
  <fieldset class="question" data-qid="q5" data-kp="KP-4" data-type="模拟" data-points="4">
    <legend>5. &lt;场景：… 你会如何 …&gt;</legend>
    <div class="qmeta">[考点: KP-4] · (模拟题, 4分)</div>
    <textarea id="q5" placeholder="在此作答"></textarea>
    <div class="feedback" id="fb-q5"></div>
  </fieldset>

  <h2>五、算法 / 推导题</h2>
  <fieldset class="question" data-qid="q6" data-kp="KP-2" data-type="算法" data-points="5">
    <legend>6. &lt;题干：请推导/设计 …&gt;</legend>
    <div class="qmeta">[考点: KP-2] · (算法题, 5分)</div>
    <textarea id="q6" placeholder="在此作答"></textarea>
    <div class="feedback" id="fb-q6"></div>
  </fieldset>

  <h2>六、高难度综合题</h2>
  <fieldset class="question" data-qid="q7" data-kp="KP-1,KP-3" data-type="综合" data-points="6">
    <legend>7. &lt;题干：综合 … 与 … 解决 …&gt;</legend>
    <div class="qmeta">[考点: KP-1, KP-3] · (综合题, 6分)</div>
    <textarea id="q7" placeholder="在此作答"></textarea>
    <div class="feedback" id="fb-q7"></div>
  </fieldset>

  <div class="controls">
    <button type="button" id="submitBtn">提交答案</button>
    <button type="reset">重置</button>
  </div>
</form>

<pre id="answerOutput" style="display:none;"></pre>

<!-- 总分汇总：批改后由 AI 填充（逐题批注已内联在各题的 .feedback 位） -->
<script id="restoreData" type="application/json" style="display:none;"></script>
<script id="quizKey" type="application/json" style="display:none;"></script>
<div id="gradingSummary" style="display:none;"></div>

<script>
(function () {
  'use strict';
  var form = document.getElementById('quizForm');
  var btn = document.getElementById('submitBtn');
  var out = document.getElementById('answerOutput');

  function collect() {
    var answers = {};
    var groups = {};
    Array.prototype.forEach.call(form.elements, function (el) {
      if (!el.name) return;
      if (el.type === 'radio' && el.checked) {
        answers[el.name] = el.value;
      } else if (el.type === 'checkbox') {
        if (!groups[el.name]) groups[el.name] = [];
        if (el.checked) groups[el.name].push(el.value);
      }
    });
    Object.keys(groups).forEach(function (n) { answers[n] = groups[n]; });
    Array.prototype.forEach.call(form.querySelectorAll('input[type=text], textarea'), function (el) {
      if (el.id && el.value.trim()) answers[el.id] = el.value;
    });
    return answers;
  }

  btn.addEventListener('click', function () {
    var payload = {
      quiz: document.body.getAttribute('data-quiz'),
      submitted_at: new Date().toISOString(),
      answers: collect()
    };
    var json = JSON.stringify(payload, null, 2);
    if (out) { out.textContent = json; out.style.display = 'block'; }
    var blob = new Blob([json], { type: 'application/json' });
    var url = URL.createObjectURL(blob);
    var a = document.createElement('a');
    a.href = url;
    a.download = document.body.getAttribute('data-quiz') + '-answers.json';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    try { localStorage.setItem('ll-answers-' + payload.quiz, json); } catch (e) {}
  });

  // ---- restore-on-load: refill form so refresh isn't blank ----
  function applyAnswers(answers) {
    Object.keys(answers).forEach(function (qid) {
      var val = answers[qid];
      Array.prototype.forEach.call(form.querySelectorAll('input[type=radio][name="' + qid + '"]'), function (el) {
        el.checked = (el.value === val);
      });
      if (Array.isArray(val)) {
        Array.prototype.forEach.call(form.querySelectorAll('input[type=checkbox][name="' + qid + '"]'), function (el) {
          el.checked = val.indexOf(el.value) !== -1;
        });
      }
      var txt = form.querySelector('#' + qid);
      if (txt && typeof val === 'string') txt.value = val;
    });
  }
  function restore() {
    // Priority 0: inline data script (AI-injected at grading time — always works, no fetch/CORS needed)
    var dataEl = document.getElementById('restoreData');
    if (dataEl && dataEl.textContent.trim()) {
      try {
        var d = JSON.parse(dataEl.textContent);
        if (d && d.answers) { applyAnswers(d.answers); return; }
      } catch (e) {}
    }
    // Priority 1: fetch sibling answers.json (placed by user next to the html)
    var slug = document.body.getAttribute('data-quiz');
    fetch('./' + slug + '-answers.json')
      .then(function (r) { if (!r.ok) throw new Error('nf'); return r.json(); })
      .then(function (data) {
        if (data && data.answers) {
          applyAnswers(data.answers);
          try { localStorage.setItem('ll-answers-' + slug, JSON.stringify(data)); } catch (e) {}
        }
      })
      .catch(function () {
        // Priority 2: localStorage cache (submit-time backup)
        try {
          var cached = localStorage.getItem('ll-answers-' + slug);
          if (cached) { var d = JSON.parse(cached); if (d && d.answers) applyAnswers(d.answers); }
        } catch (e) {}
      });
  }
  restore();
})();
</script>
</body>
</html>
```

**Quiz-form non-negotiables:**
- `<body data-quiz="<slug>">` carries the slug used in the download filename.
- Every question is a `<fieldset class="question" data-qid="qN" data-kp="..." data-type="..." data-points="...">` — the AI reads these data-* attrs when grading (this replaces the inline `[考点: KP-x]` md tags).
- Radio/checkbox `name` MUST equal the qid (`q1`); text/textarea `id` MUST equal the qid (`q3`). The submit JS relies on this exact mapping.
- `<form id="quizForm">`, `<button id="submitBtn" type="button">`, `<pre id="answerOutput">`, `<script id="restoreData" ...>`, `<script id="quizKey" ...>`, `<div id="gradingSummary">` all required. Every question's `<fieldset>` MUST contain a `<div class="feedback" id="fb-qN">` slot (empty initially).
- The submit JS is the canonical version from `references/html-format.md` — copy verbatim, do not rewrite. It includes the **restore-on-load** logic: on page load, `fetch('./<quiz>-answers.json')` to refill the form (so refresh isn't blank once the user placed the downloaded json next to the html), falling back to a `localStorage` cache if fetch is CORS-blocked (Chrome on file://) or the file is absent. Both the submit handler (which writes localStorage) and the `restore()` call at the end are mandatory parts of the canonical JS.
- Stage-total-quiz uses the SAME skeleton; just add more questions (≥2 per type, ≥2 综合) and `[出处: url]` spans inside the qmeta div for web-research citations.