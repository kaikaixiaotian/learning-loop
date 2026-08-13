# Subagent Protocol

Three jobs in the learning loop are delegated to subagents via the Agent tool. Subagents run isolated — they cannot see this conversation, so every dispatch must be **self-contained**: include the templates, the wiki input, and the exact output path.

## When to use a subagent (and when not)

| Job | Subagent? | Why |
|-----|-----------|-----|
| Write chapter wiki after pass | ✅ yes | Isolated summarization; keeps main thread lean |
| Plan + draft next chapter + quiz | ✅ yes | Adapts to wiki; substantial generation |
| Plan a whole new stage | ✅ yes | Larger planning task |
| **Fetch authoritative web data for stage-total** | ✅ yes | Volume + factual rigor required; isolated fetch keeps main thread lean |
| **Fetch canonical learning path + per-chapter cards (curriculum)** | ✅ yes | Grounding the master plan in real curricula; the biggest anti-"messy plan" fix |
| Grade a quiz | ❌ no | Needs conversational context + user's answers; do inline |
| Run the plan-quiz | ❌ no | It's a live back-and-forth; do inline |
| Rebuild a failed chapter | ❌ no | Needs the specific failure analysis from the just-graded quiz; do inline |

The rebuild case is intentional: the main thread just saw exactly what the user got wrong, so it's best positioned to rewrite the chapter. Subagents are for forward planning, not error response.

## Common dispatch rules (all jobs)

1. **Inline the templates and rules.** Subagents start cold with no conversation context. Inline the relevant template sections (from `references/templates.md`), the quiz-type floor rules (from `references/quiz-types.md`), and any grading rules they need (from `references/grading.md`) directly in the prompt. Do not tell a subagent to "read references/templates.md" — point it at files *inside the learning workspace* (which do exist on disk) only for content it must absorb, and even then prefer pasting that content inline for reliability.
2. **Inline the wiki.** If the job depends on prior learning state, paste the contents of the relevant `wiki/*.md` files inline.
3. **Specify the output path(s) absolutely.** Tell the subagent exactly which file(s) to write and their full paths.
4. **Specify the format.** "Return the written file path and a 3-line summary of what you produced. Do not dump the full content back."
5. **Give the calibration inputs.** `baseline_score`, `target_level`, current stage/chapter, and the user's known weak spots from the wiki.
6. **One job per subagent.** Don't ask one subagent to both write the wiki and plan the next chapter — the wiki must exist *before* the next chapter is planned, so sequence them.

## Job 1: Wiki writer (after a chapter passes)

**Input to pass:** the chapter doc path, the graded chapter quiz, the plan-quiz Q&A and score, and the topic/stage/chapter identifiers.

**Prompt skeleton:**

```
You are recording a learning-progress wiki for an AI tutoring system.

Context:
- Topic: <topic>
- Stage <N>, Chapter <XX>: <title>
- Chapter doc: <path>  [USE THE Read TOOL ON THIS PATH]
- Chapter quiz (graded): <paste inline>
- Plan-quiz Q&A and combined score: <paste inline>

Use the Read tool on the chapter doc path above, then WRITE the file
<abs path to wiki/stageN-chXX-wiki.md> with EXACTLY this structure:
<paste the wiki schema from references/wiki-schema.md inline here>

Be specific and honest — this file guides how the NEXT chapter is planned.
Mention concrete misconceptions, not vague "needs practice".

Return only: the file path you wrote and a 3-line summary.
```

After it returns, **append a one-line entry to `wiki/progress.md`** and update `meta.json` (chapter status, plan-quiz score).

## Job 2: Next-chapter planner (after wiki is written)

**Input to pass:** the master plan (so it knows which chapter is next), the just-written chapter wiki, the user's `baseline_score` and accumulated weak spots, and the absolute output paths.

**Prompt skeleton:**

```
You are planning the next chapter of an AI tutoring system.

Context:
- Topic: <topic>, target level: <level>
- User baseline score: <x> (0..1)
- Master plan path: <path to plan/master-plan.html>  [USE THE Read TOOL ON THIS]
- Previous chapter's wiki (READ THIS WITH THE Read TOOL — adapt to what it says): <path>
- Known weak spots across all chapters so far (inline): <bulleted list distilled from wikis>

The NEXT chapter to build is Stage <N> Chapter <XX>: <slug> — <objective>.

Produce TWO files (BOTH are HTML — see references/html-format.md):
1. <abs path to chapters/stageN-chXX-<slug>.html> — the chapter doc (read-mode HTML)
2. <abs path to quizzes/stageN-chXX-quiz.html> — its quiz (quiz-form HTML)

Chapter doc template (follow exactly):
<paste the read-mode HTML skeleton from references/templates.md inline — note the
NEW 知识点清单 (KP list) section and the six-element structure for 核心概念>

TEACHING DEPTH RULES (non-negotiable):
- The chapter MUST start with a 知识点清单: 4–8 knowledge points (KP1, KP2, …),
  each a one-liner, mapping to a 核心概念 subsection. This list is the SOLE
  basis for what the quiz may test.
- Every 核心概念 subsection MUST have all six elements: ①精确定义
  (formula/signature/syntax — verifiable, not analogy), ②直觉解释 (analogy),
  ③最小例子, ④推导或代码, ⑤边界条件, ⑥与相关概念对比. Analogies never
  substitute for the precise definition. If you find yourself writing a vivid
  analogy but no precise definition, STOP and add the definition.
- The 实战演示 must be reproducible (commands/expected output, or full
  derivation).

Quiz rules (NON-NEGOTIABLE) — paste inline:
<paste the floor rules from references/quiz-types.md — all 6 types, ≥1 each,
选择+填空 ≤50%, ≥1 实战, ≥1 综合>

COVERAGE SELF-CHECK (mandatory before returning):
- Every quiz question MUST carry a [考点: KP-x] tag.
- For each question, verify KP-x exists in the 知识点清单 AND is substantively
  taught in 核心概念 (six elements present). A KP listed but thin = out-of-scope
  for deep questions.
- If a question's concept isn't actually taught, REWRITE the question to test a
  taught KP — do not ship out-of-scope questions. Catching them here is far
  better than the user discovering them at grading time.
- Include one line in your return summary: "Coverage: all N questions map to
  KPs {list}; no out-of-scope items."

VISUALIZATION (optional, per-KP judgment):
- For each KP, evaluate against the signal table in references/visualization.md
  (paste the 6-signal table inline). Visualize if ≥2 signals fire; skip if <2.
- For KPs you decide to visualize, generate a STANDALONE interactive HTML file at:
  <abs path to chapters/viz/stageN-chXX-<kp-slug>.html>
  Requirements (non-negotiable): vanilla HTML/CSS/JS, all inline, no CDN/external
  deps, 'use strict' + IIFE, at least one visible interaction (button/slider/click)
  that changes the stage, a reset control, Chinese labels. Follow the html skeleton
  in references/templates.md. Use the render()-from-state pattern.
- In the chapter doc HTML, link each viz inline at the END of its 核心概念 subsection
  (after the ⑥对比 element) using this exact format (HTML anchor, not markdown):
  <a class="viz-link" href="./viz/stageN-chXX-<kp-slug>.html">🖼️ 交互演示：<一句话名> — <用户看到/做到什么></a>
- DO NOT add viz for KPs that don't warrant it. Zero viz is fine.
- After writing each html file, SELF-VERIFY before returning (the main agent re-verifies):
  (a) extract the <script> content and confirm no syntax errors;
  (b) confirm every getElementById('x') has a matching id="x" in the HTML;
  (c) confirm required elements exist — for the quiz: <form id="quizForm">,
      <button id="submitBtn">, <pre id="answerOutput">, <div id="gradingSummary">,
      and each fieldset has <div class="feedback" id="fb-qN"></div>;
      for viz: the controls; for read-mode: titled sections;
  (d) for the quiz: every <fieldset data-qid="qN"> has a form control whose
      name (radio/checkbox) or id (text/textarea) equals "qN".

QUIZ HTML RULES (the quiz is now a form, not md — see references/html-format.md):
- Use the quiz-form html skeleton from references/templates.md verbatim structure.
- Every question is <fieldset class="question" data-qid="qN" data-kp="KP-x"
  data-type="选择|填空|实战|模拟|算法|综合" data-points="N">.
- Radio name="qN" value="A/B/C/D"; checkbox name="qN" value="A/B/C/D";
  text/textarea id="qN".
- Copy the canonical submit JS from references/html-format.md verbatim — do NOT
  rewrite it.
- **FILL the `<script id="quizKey">` tag with the correct answers** (mandatory — this is what the grader reads instead of regex-parsing the HTML). Use the schema from references/html-format.md: for each question, include `qid`/`type`/`kp`/`points` + either `answer` (objective types) or `rubric` (subjective types). Every qid must match a `<fieldset data-qid>` 1:1. The quizKey must be valid JSON — verify with JSON.parse before returning.

Include in your return summary a visualization_decisions block:
  visualization_decisions:
    KP1: skip (0 signals)
    KP3: visualize (signals: data-flow, multi-step) → viz/stageN-chXX-<slug>.html
    KP4: skip (1 signal)

Calibration: <adapt difficulty based on baseline_score, target_level, and weak spots — be specific>

Return only: the file paths (chapter doc, quiz, any viz files) and a summary including the coverage line + the visualization_decisions block.
```

After it returns: **two spot-checks before handing to the user.**
1. Coverage: open the chapter doc's KP list and the quiz's 考点 tags — fix any mismatch inline.
2. Visualization verification: for EVERY html file returned, run the JS static checks in `references/visualization.md` yourself (syntax check via `node --check` on the extracted script; element-existence; undefined-reference). A failing demo is NOT shipped — re-dispatch the planner to fix the specific failure, or drop the demo and replace its chapter-doc link with a prose note. Do not trust the subagent's self-verify alone; re-run the checks.
Then set `phase: "learn"`, advance `current_chapter`, tell the user the next chapter is ready.

## Job 3: Stage planner (when advancing to a new stage)

**Input to pass:** the topic, all prior stage wikis (so stage N+1 builds on what was actually learned, not just the original plan), `baseline_score`, and the master plan.

**Prompt skeleton:**

```
You are planning the next STAGE of an AI tutoring system.

Context:
- Topic: <topic>, target level: <level>
- Stage <N> just passed. Planning Stage <N+1>.
- Master plan path: <path>  [USE THE Read TOOL ON THIS]
- Prior stage wikis (USE THE Read TOOL ON EACH — stage N+1 must build on
  demonstrated mastery and explicitly reinforce prior weak spots):
  <list paths>

Produce (all HTML — see references/html-format.md):
1. Update <abs path to plan/master-plan.html> — fill in stage <N+1>'s chapter
   list if not already detailed (title + objective + type emphasis each).
2. <abs path to chapters/stage<N+1>-ch01-<slug>.html> — first chapter doc (read-mode)
3. <abs path to quizzes/stage<N+1>-ch01-quiz.html> — its quiz (quiz-form)

Templates and quiz floor rules (paste inline):
<paste read-mode HTML skeleton + quiz-form HTML skeleton + quiz floor rules>

QUIZ HTML RULES (same as Job 2 — the quiz is a form):
- Every question is <fieldset class="question" data-qid="qN" data-kp="KP-x"
  data-type="..." data-points="N">.
- Radio/checkbox name="qN"; text/textarea id="qN".
- Copy the canonical submit JS from references/html-format.md VERBATIM (including
  the empty-skip: `if (el.id && el.value.trim())`). Do NOT rewrite the JS.
- <form id="quizForm">, <button id="submitBtn" type="button">,
  <pre id="answerOutput" style="display:none;">, <div id="gradingSummary" style="display:none;">,
  and each fieldset has <div class="feedback" id="fb-qN"></div>.
- **FILL the `<script id="quizKey">` tag** with the correct answers (same rules as Job 2 above — mandatory, use the schema from references/html-format.md).

COVERAGE SELF-CHECK (mandatory before returning):
- Every quiz question's data-kp maps to a KP in the chapter doc's 知识点清单.
- Rewrite any question whose KP isn't taught — do not ship out-of-scope items.
- Include: "Coverage: all N questions map to KPs {list}; no out-of-scope."

SELF-VERIFY each HTML file before returning:
(a) extract <script> and confirm no syntax errors;
(b) every getElementById('x') has matching id="x";
(c) required elements present (quizForm/submitBtn/answerOutput/gradingSummary + each fieldset's fb-qN slot for quiz);
(d) every data-qid="qN" has a control with name/id = "qN".

Calibration: this is stage <N+1>, so difficulty steps up. But honor the
weak spots from prior wikis — reinforce before extending.

Return only: paths written + summary including the coverage line + self-verify result.
```

After it returns: **re-verify the HTML yourself** (do not trust the subagent's self-verify alone) — run the JS static checks from `references/html-format.md` on every generated HTML (syntax, element existence, qid↔control matching). Fix any failure inline, or degrade that artifact to md. Only then advance `current_stage`, reset `current_chapter` to 1, set `phase: "learn"`.

## Job 4: Web researcher (mandatory before every stage-total quiz)

**Purpose:** ground the stage-total quiz in real facts from authoritative sources, so its larger volume (≥12 questions) doesn't drift into AI-confident-but-wrong territory.

**Input to pass:** the stage's full scope — all chapter topics + the cumulative weak spots from chapter wikis. The researcher does NOT write the quiz; it returns a brief that the stage-total planner composes into questions.

**Critical tools note:** This subagent needs network access. It must use the `WebSearch` and `WebFetch` tools. When dispatching, remind it explicitly: "Use the WebSearch tool to find official sources, then WebFetch to read them. Do NOT fabricate sources."

**Prompt skeleton:**

```
You are researching authoritative material for a stage-total quiz in an AI tutoring system.

TOOLS: Use WebSearch and WebFetch. Every fact you return MUST come with a real URL you actually fetched. Do NOT invent or guess URLs. If you cannot find an authoritative source for a fact, omit it and note the gap.

Context:
- Topic: <topic>, target level: <level>
- Stage <N>: <stage name>
- Chapters in scope (cover each):
  1. <chapter title> — <objective>
  2. ...
- Cumulative weak spots to re-probe (from chapter wikis):
  - <weak spot 1>
  - ...

WHITELIST — fetch ONLY from these (paste the whitelist table from
references/web-research.md inline here). Anything else requires noting
as unverified.

For each chapter's concepts, find:
- Core facts (API signatures, exact defaults, semantic rules) from official docs.
- Real gotchas / common mistakes from the official repo (issues/README) or
  authoritative docs.
- Anything version-sensitive (note the version + recency).

Return EXACTLY this format (the brief the quiz planner will consume):

TOPIC: <stage topic>
REQUESTED SCOPE: <chapters covered>

FINDINGS:
## Concept: <name>
- Fact: <statement> [出处: <url>]
- Fact: <statement> [出处: <url>, <url2>]
- Gotcha: <statement> [出处: <url>]
- Recency: <version/date note, or "unknown">

## Concept: <next>
...

DEGRADATION NOTES:
- <concepts with no whitelist source — mark unverified>
- <if network failed entirely, say so>

sources:
  - url: <url>
    type: official-doc | primary-source-repo | rfc | ...
    accessed: <date>
    recency: <note>
    used_for: <which fact>

AUTHENTICITY RULES (non-negotiable):
- Source gate: whitelist only.
- Corroboration gate: behavioral claims need ≥1 official OR ≥2 independent
  sources (≥1 whitelist).
- If a fetched page contradicts common belief, TRUST THE PAGE. Note the
  contradiction in the brief.
- If network fails or returns nothing for a concept, do NOT substitute AI
  memory — leave that concept out of FINDINGS and note it in DEGRADATION NOTES.

Return only the brief. Do not write any quiz file.
```

**After it returns:**
- If `DEGRADATION NOTES` says full network failure → follow `references/web-research.md` graceful degradation: generate stage-total from AI alone + add the ⚠️ 降级 banner.
- Otherwise → hand the brief to the stage-total generation step (inline the whole brief into that step's context). Compose questions from `FINDINGS`, carry each `[出处]` into the question.

**Do not skip this job for stage-totals.** If you find yourself about to generate a stage-total without a research brief, stop and dispatch the researcher first. The only exception is documented network failure (degradation path).

## Job 5: Curriculum researcher (mandatory before master-plan & new stages)

**Purpose:** pull a canonical learning path from authoritative sources so the master plan reflects how the field is actually taught, not AI's invented structure. This is the fix for "messy plans". Read `references/curriculum-research.md` for the full spec.

**Critical tools note:** needs network access — must use `WebSearch` and `WebFetch`. Remind it explicitly: "Use WebSearch to find official learning paths/roadmaps, then WebFetch to read them. Do NOT fabricate sources or invent a path from memory."

**Input to pass:** the topic, the user's target_level (as a depth hint), and the absolute output directory for the per-chapter cards (`plan/sources/`).

**Prompt skeleton:**

```
You are researching the canonical learning path for an AI tutoring system's
master plan. You do NOT write the plan — you return a skeleton + per-chapter
material cards that the planner will adapt.

TOOLS: Use WebSearch and WebFetch. Every structural claim (stage/chapter
ordering) MUST come from a real fetched URL. Do NOT invent a path from memory.

Context:
- Topic: <topic>
- Target level hint: <aware|practitioner|expert>  (filters how deep to go)

WHITELIST — fetch the path skeleton from (paste the curriculum whitelist table
from references/curriculum-research.md inline): official learning paths,
official docs TOC, canonical roadmaps (roadmap.sh etc.), authoritative book
TOCs, standard curricula. Avoid blog 学习路线 listicles.

Find ≥2 independent whitelist sources for the topic. Extract the CONSENSUS
structure (where they disagree, note it and prefer the official-doc ordering,
recording why).

Return the skeleton in EXACTLY this format (from curriculum-research.md):

TOPIC: <topic>
TARGET LEVEL HINT: <level>

SOURCES USED (≥2):
- url: <official learning path>
  type: official-learning-path | roadmap | book-toc | official-doc-toc
  accessed: <date>
  what_it_gave: <the spine>
- url: <second source>
  ...

CANONICAL PATH:
Stage 1: <name>  [source: <url>]
  - Chapter: <title> — <objective> [source: <url>]
  - Chapter: ...
Stage 2: <name>  [source: <url>]
  - Chapter: ...
...

PREREQUISITE NOTES:
- <X before Y because...>  [source: <url>]

COMMON GOTCHAS IN TEACHING THIS FIELD:
- <learners struggle with W; teach via V>  [source or consensus]

DEGRADATION NOTES:
- <topics with no whitelist curriculum — flag for AI fill, marked unverified>

ALSO: for EACH chapter in the canonical path, write a per-chapter material card
to <abs path>/plan/sources/stageN-chXX.md using this format (paste the card
template from references/curriculum-research.md). These cards let later
chapter-generation draw on authoritative material instead of re-fetching.
Create the plan/sources/ directory if it doesn't exist.

AUTHENTICITY RULES:
- Whitelist only for the skeleton structure.
- If sources disagree sharply, note it and pick official-doc preference.
- If network fails entirely, say so in DEGRADATION NOTES and return an empty
  skeleton (the planner will fall back to AI + banner) — do NOT substitute
  AI memory for a missing external path.

Return only: the skeleton brief + a list of card file paths written + a 3-line
summary (sources used, # stages/# chapters, any degradation).
```

**After it returns:**
- If `DEGRADATION NOTES` says full network failure → follow graceful degradation in `references/curriculum-research.md`: generate plan from AI + add the ⚠️ banner to master-plan.md.
- Otherwise → take the skeleton, apply the allowed-adaptations table to fit the user's baseline/target_level, and write `master-plan.html` (read-mode HTML, with source citations + an adaptations section). The subagent gives you the structure; YOU do the user-specific adaptation (that needs the baseline profile, which the subagent doesn't have).

**Do not skip this job before writing master-plan.html.** If you find yourself about to invent stages/chapters from memory, stop and dispatch the curriculum researcher first. The only exception is documented network failure.

When a chapter passes, the correct order is:

1. (inline) Teach the wrong answers.
2. Dispatch **wiki writer** → wait.
3. Append to `progress.md`, update `meta.json`.
4. Dispatch **next-chapter planner** with the just-written wiki → wait.
5. Tell the user the next chapter is ready.

Steps 2 and 4 are sequential, not parallel — the planner needs the wiki as input. Don't try to batch them.

## What to do if a subagent output is weak

If a returned chapter doc or quiz violates the floor rules (e.g. missing a type, or 选择-heavy), **do not hand it to the user as-is**. Either:
- Re-dispatch with a sharper prompt naming the specific defect, or
- Fix the specific defect inline yourself.

The user never sees a quiz that breaks the six-type rule.
