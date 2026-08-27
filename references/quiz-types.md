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

Ask one question, wait for the answer, then the next. After the last, score and combine with the chapter quiz for the pass/fail gate (combined = 0.45 × chapter + 0.55 × plan, ≥0.80 passes — see `grading.md`).

Dynamic mixing inside the plan-quiz still adapts to signals: chapter nature (skill chapters lean 实战), user level (high → heavier 算法/综合), past weak spots (re-probe them from a new angle).

## Stage-total specifically

Covers all chapters in the stage. **Volume follows content breadth** — roughly 2–4 objective questions per chapter is typical — but the driver is covering every chapter's key assertions plus the stage's cumulative weak spots, not hitting a quota. Composition:

- Sections 一 (选择题) + 二 (填空题), drawn across all chapters; include **≥2 组合多选题 whose correct sets span ≥2 chapters' assertions** (this replaces the old cross-chapter 综合 quota objectively).
- End with **at most 1** textarea 文字综合题 spanning multiple chapters (5–8 分; rubric-graded; may be omitted).
- Questions re-probe the weak spots recorded in chapter wikis. Same ≥0.80 gate as before.
