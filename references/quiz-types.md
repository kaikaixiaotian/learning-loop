# Quiz Types & Question Composition

The user explicitly required that **pre-filled quizzes must not pile up long-form written questions** — answering time belongs to learning, not to form-filling. As of v1.5.0, every **form quiz** (baseline, chapter quiz, stage-total) is composed of **objective items: 选择 + 填空**, with ONE exception — the stage-total may include at most **1** written 综合 question. Subjective depth questions are NOT gone; they moved to where answering is cheap: the **plan-quiz** (live chat, one question at a time).

## Two arenas, two rule sets

| Arena | Format | Allowed types |
|-------|--------|---------------|
| **Form quizzes** — baseline / chapter quiz / stage-total | HTML form (`quiz-form`) | 选择（单选/多选）+ 填空 only；stage-total 额外允许 ≤1 道 textarea 文字综合题 |
| **Plan-quiz** — live transfer check after each chapter quiz | chat, one at a time | 全六题型照旧（实战/模拟/算法/综合为主体，见文末） |

## Form-quiz item repertoire

### 1. 选择题 (single / multi select)
- **Form**: options A–D (or more); multi-select marked 多选 with checkbox semantics.
- **Tests**: recognition, discrimination between similar concepts, scenario judgment, code reading, cross-concept combination.
- Rotate these sub-patterns so the section isn't monotone:
  - **概念辨析**：「以下哪个说法正确」— classic definition checking.
  - **情境判断**（模拟题的客观化）：describe a messy scenario under constraints, ask「最佳做法是？」. Distractors = defensible-but-suboptimal or commonly-chosen-wrong paths.
  - **代码阅读**（实战题的客观化）：show a snippet → 问输出是什么 / 哪一行有 bug / 哪段实现正确 / 哪条命令序列能达到目标。
  - **组合判断**（综合题的客观化）：a multi-select whose correct set spans ≥2 KPs or chapters:「下列哪些说法的组合能同时满足 X 与 Y」。
  - **关键步骤定位**（算法/推导题的客观化）：问推导或流程的「第一步应该是什么 / 决定性的判断点在哪」，选项为候选步骤。

### 2. 填空题
- **Form**: a sentence/statement with one blank (multi-blank allowed for compound facts).
- **Tests**: recall precision of terms, values, formulas, parameters.
- **Rules**: keep blanks unambiguous — avoid free-text where many phrasings are defensible; when near-synonyms exist, list them in the quizKey's `accept` array.

## 客观化转换指南 (keeping depth without textareas)

The old 实战/模拟/算法/综合 types tested abilities that objective items CAN still test — by moving the reasoning into the distractor design:

| 原主观题考什么 | 客观化后怎么考 | 必须保留的深度信号 |
|----------------|----------------|--------------------|
| 实战 — 能不能做出来 | 给真实代码/配置/数据片段：选输出、找 bug 行、辨析哪种实现满足全部约束 | 片段要来自该章 ③④⑤ 出现过的机制，错误实现各对应一个真实踩坑点 |
| 模拟 — 判断与取舍 | 情境决策单选：最佳做法是？ | 讲评里展开 trade-off（为何次优选项次优）；选项须都可辩护，只是优劣不同 |
| 算法/推导 — 推理过程 | 选推导结果 / 选第一步或关键分支 / 关键中间值与复杂度填空 | 干扰项覆盖典型中途走偏的路；答案讲评补全完整推导链 |
| 综合 — 跨章整合 | 跨 KP/跨章的组合多选题 | 正确组合 ≥2 章/KP 的断言；错误项为跨章迁移时的典型混淆 |

**Distractor quality bar (hard rules):**
- Every wrong option must correspond to a *real, common misconception* or a real trap taught in the chapter's ⑤边界条件 — never an absurd filler.
- No giveaway options: if an option can be eliminated without understanding the concept, rewrite it.
- For scenario items, ALL options should look defensible at first glance; correctness comes from constraint analysis. This is how 判断力 survives objectification.
- The 失分点 you write at grading time replaces the old rubric feedback: name the misconception behind the chosen wrong option.

## Coverage rule — replaces the old type quota

There is **no fixed question count and no type quota** anymore. The driver is coverage:

- Read the chapter doc's 断言清单 (or, for baseline/stage-total, the domain's breadth). Generate enough objective items to **cover every assertion worth testing**, with easy → hard ladder inside each sub-pattern. Fewer questions ≠ less coverage — if the list demands it, generate more questions rather than drop topics.
- Re-probe past weak spots (from chapter wikis) from a new angle — as new objective items in later chapter quizzes and stage-totals.
- User-level adaptation lives in difficulty now: low level → clean statements, short snippets, direct recall; high level → longer snippets, subtler misconceptions, scenario items where several options are plausible.
- State point value per question so grading stays transparent.

### Floor rules (non-negotiable)

Regardless of adaptation, every form quiz must satisfy:

- Baseline & chapter quiz contain **ONLY 选择 + 填空**. Zero textarea 主观大题.
- Stage-total: 选择 + 填空 make up essentially all the points; **AT MOST 1** textarea 文字综合题 (cross-chapter; skipping it entirely is also fine).
- 题量由断言清单全覆盖决定 — 不设固定区间。宁可多几道快速客观题，也不出一道让用户写十分钟的大题。
- Every question maps its assertions via `data-assert`/quizKey to the 断言清单 (unchanged gate).

If you find yourself drafting a long-form written question for a form quiz — **stop**: either convert it into an objective item with misconception-driven distractors, or move its spirit into the next plan-quiz.

## Difficulty ladder within a type

Inside any single quiz, order questions easy → hard within each pattern, and weight points accordingly (选择 1pt, 多选 2pt, 填空 1–2pt, stage-total 文字综合题 5–8pt). State the point value next to each question so grading is transparent.

**Point-notation format is fixed** — use it for EVERY question so points can be summed reliably by both humans and tooling:

```
3. (选择题[多选], 2分) …
```

Rules for the notation:
- Half-width parentheses `(` `)` and half-width comma `,` — NOT full-width `（`，`）`. Mixed-width brackets break automated point-summing.
- Format exactly `(题型, X分)`. Enum: 选择题 / 填空题 remain the workhorses of form quizzes (`多选` noted inside brackets when applicable); 实战题 / 模拟题 / 算法题 / 综合题 survive only in plan-quizzes and (综合题 only) in the stage-total's single optional written item. Legacy quizzes using the full enum still grade normally.
- Place it immediately after the question number, before the question text.

Every question in every quiz uses this exact notation. A quiz with any unmarked question is not finished — regenerate or fix before handing to the user.

## Plan-quiz specifically — six-type depth lives HERE (unchanged spec)

The plan-quiz is **live** (asked in chat, one at a time) and must use **different** questions from the chapter quiz — new scenarios, edge cases, cross-concept links. Its purpose is transfer, not recall. Because the user answers conversationally one question at a time, long-form questions are cheap here — this is the home of 实战/模拟/算法/高难度.

Recommended mix for a plan-quiz of ~6 questions:

- 0–1 选择 or 填空（warm-up）
- 1–2 实战
- 1 模拟
- 1 算法/推导
- 1 高难度综合

Ask one question, wait for the answer, then the next. **Before scoring each answer, run the completeness check (漏答追问):** a live answer must address every point the question asked — sub-questions, 小问, required dimensions. If the reply skipped a point, follow up naming it and restating that part — 「第 X 题里的『某某点』你还没有作答——题目问的是……，请补充」 — and wait; the supplement joins the answer and grades normally. One follow-up per question is the cap: if the user explicitly passes (不会/跳过) or still misses the point after that follow-up, record the sub-point as 未作答 — 0 on its share, noted in the 失分点 + 批阅区, re-taught in the 讲评 — and move on. Every asked point must end resolved (answered or explicitly 未作答); never silently drop one from grading. After the last, score and combine with the chapter quiz for the pass/fail gate (combined = 0.45 × chapter + 0.55 × plan, ≥0.80 passes — see `grading.md`).

Dynamic mixing inside the plan-quiz still adapts to signals: chapter nature (skill chapters lean 实战), user level (high → heavier 算法/综合), past weak spots (re-probe them from a new angle).

## Stage-total specifically

Covers all chapters in the stage. **Volume follows content breadth** — roughly 2–4 objective questions per chapter is typical — but the driver is covering every chapter's key assertions plus the stage's cumulative weak spots, not hitting a quota. Composition:

- Sections 一 (选择题) + 二 (填空题), drawn across all chapters; include **≥2 组合多选题 whose correct sets span ≥2 chapters' assertions** (this replaces the old cross-chapter 综合 quota objectively).
- End with **at most 1** textarea 文字综合题 spanning multiple chapters (5–8 分; rubric-graded; may be omitted).
- Questions re-probe the weak spots recorded in chapter wikis. Same ≥0.80 gate as before.

## Drill-mode question rules (刷题模式出题规则)

刷题模式是独立于表单测验/plan-quiz 的第三套出题场：题目存进 `题库/`（数据源），每轮随机选一题放入新建的 `题目-NNN/` 文件夹让用户作答。出题规则如下（状态机与选题优先级见 SKILL.md「Drill-mode flow」）。

### 题型与作答方式（四类）

| 类型 | 作答方式 | 批阅依据 |
|------|----------|----------|
| **选择题**（单/多选） | 用户直接写入 `题目.md` 作答区（字母） | 题库存档的答案 |
| **填空题** | 用户直接写入作答区（按空作答） | 答案 + `accept` 同义列表 |
| **应用题**（情境应用，须说明准确原因） | 用户直接写入作答区（结论 + 原因，可含代码/命令） | **结论与原因都正确才算对**；缺原因→追问一轮（漏答追问模式） |
| **算法题**（编程主题 stage 3 的形态） | 用户在同文件夹的代码文件（`solution.<ext>`）里实现 | 尽量实际运行最小用例验证；无运行时则静态推演并注明 |

四类都来自题库存档（`题库/Q-xxx.md`），作答文件里**不得**出现答案或解析；每轮用哪一类由所属知识点的 stage 决定（见下节难度阶梯）。

**点评必附正确代码示例**：每轮批阅（无论对错）的讲评必须附一段正确示例——算法题为参考实现（答对给对照/更优写法，答错给修复代码），选择/填空/应用题为体现正确答案的最小可运行示例（代码/命令/配置；应用题的示例须同时印证正确原因）。该示例在题目入库时就要备好（存于题库存档的 答案 区），AI 自主出题与链接采集题同样执行。

### 题目来源与选题（知识点中心）

- 选题先选**知识点**，再按该知识点的 stage 出对应题型——题目服务于知识掌握，同一知识点可换题型、换角度反复考。选题优先级与掌握判定状态机见 SKILL.md「Drill-mode flow」。
- **真题**（用户粘贴）/ **链接采集**（Job 6 子代理抓取）——刷题的核心素材，优先级最高。
- **AI 自主出题**（`source: "AI"`）——题库为空时的启动素材、以及真题覆盖不了的知识面补充；用户补充真题后真题优先。
- **变种**（`source: "变种"`）——知识点进入 stage 4 或掌握后的复习加深时，基于真题生成的加难题目。

### AI 自主出题质量要求（与客观化转换指南同源）

- 干扰项硬规则沿用上文：每个错误选项必须对应一个**真实常见误解**或该主题的边界陷阱，禁止凑数选项；情境题所有选项都要初看可辩护。
- 填空题空位必须答案唯一；存在同义写法时在题库 `accept` 列表里列全。
- 应用题必须明确要求写出**结论与准确原因**；题库存档的答案区同时给出参考结论与参考原因。
- 算法题必须**可运行验证**：输入/输出/约束明确 + 文件头里给最小自测用例（含一个边界情形）；不出无法批阅的题。
- 初始 AI 题组（用户暂无真题时）：约 5–8 题，覆盖主题主要知识点，**每题标注唯一所属知识点**，每个知识点的首题为选择题（stage 1）；宁可小而准，题库随后随真题补充生长。

### 题型难度阶梯（知识点 stage 1→4）

每个知识点独立爬梯，新知识点（含补充真题引入的）一律从 stage 1 起步：

| stage | 题型 | 考什么 |
|-------|------|--------|
| 1 | 选择 | 识别 |
| 2 | 填空 | 精确回忆 |
| 3 | 应用（编程主题=算法实现） | 情境应用 + **说出准确原因** |
| 4 | 变种/组合 | 迁移与综合，难度随 `variant_level` 递增 |

晋级/降级由 AI 判断：答对且理由充分 → 晋级（表现强可跳级）；答错 → 停留当前 stage 并随机排期复现；复习答错 → 退回重爬（起点由 AI 定，通常 stage 2）。掌握判定规则见 SKILL.md「Drill-mode flow」。

### 变种难度阶梯（variant_level）

知识点进入 stage 4（或掌握后的复习加深）时，从该知识点的真题取材生成变种，每提升一档 `variant_level` 难度上一级：

1. **难度 1 — 改条件**：换数值/换场景/加一个约束，考同一条结论的迁移。
2. **难度 2 — 换角度**：逆问（给结论推条件）、对比辨析（与易混淆概念混合出题）、边界加深。
3. **难度 3 — 组合**：合并两道及以上真题的考点成一题（组合选择/综合填空/复合算法题）。

每个变种必须在题库存档中标注基于哪道真题（`变种（基于Q-00x · 难度档N）`）及其所属知识点，并纳入该知识点的 stage/掌握跟踪。刷题空间无上限：真题可随时补充（优先），变种档位可无限递增。
