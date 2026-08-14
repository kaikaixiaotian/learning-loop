# Upgrade Protocol

`/learning-loop upgrade` does two things: (1) **pulls the latest skill from GitHub** (`git pull` in the install dir + syncs the command file itself), then (2) **migrates workspaces** per this document. This file covers step (2): marking existing workspaces so **subsequently generated content** follows the current skill spec, while **leaving existing files untouched**. Read this when the user runs upgrade.

## Design principle

Old workspaces were created by earlier skill versions — their quiz HTMLs may lack `quizKey`, `restoreData`, restore JS, or `feedback` slots. **Upgrade does NOT patch those existing files.** They keep working as-is (the AI falls back to reading questions when grading a quiz without quizKey; the form just won't auto-refill if restoreData is absent — annoying but not broken). Upgrade's only job is to ensure **everything generated from now on** follows the current spec.

Why not patch old files: patching is risky (could corrupt a working HTML), non-idempotent if done wrong, and unnecessary — old files still function. New chapters/quizzes are where the improvements matter, and those are generated fresh by the current skill, which already emits the full feature set.

## What upgrade does

1. For each `*-learning/` workspace in the current directory, read its `meta.json`.
2. Write two fields to `meta.json`:
   - `"schema_version": "<current>"` — marks this workspace as upgraded. Current value: the date or version tag of the skill at upgrade time (e.g. `"2026-08-13"`).
   - `"upgraded_at": "<ISO timestamp>"` — when the upgrade happened.
3. Append a `history` event: `{ "ts": "...", "event": "upgraded", "detail": "schema_version=2026-08-13; existing files untouched; subsequent generation follows current spec" }`.
4. Print a one-line summary per workspace: "<workspace> upgraded. Existing files unchanged. New chapters/quizzes will use the current format (quizKey + restoreData + restore JS + feedback slots)."

That's it. No file scanning, no patching, no HTML editing.

## What changes after upgrade

Nothing about existing files. But the AI's behavior for **new** content in this workspace now follows the current skill spec automatically (the skill files themselves are already the latest version) — and this is now **enforced, not just promised**: every generated chapter/quiz must carry a `<!-- learning-loop skeleton: ... -->` signature that the main agent greps for before shipping, so new content can no longer silently inherit an old sibling's visual skeleton:

- New chapter docs → read-mode HTML per **spec 2.0**: 知识点清单 + 考点断言 inventory (each KP with 3–6 testable assertions), six-element concepts with **② 直观演示** (embedded interactive demo + 观察要点 — **analogies are banned**), per-concept **检查点** (`<details>` self-test), anti-wall-of-text formatting (① per-claim list items, ⑤ case `<ul>`, ⑥ comparison table).
- New demos → **default-on per concept** (expect 5–8 per chapter; waiver only for pure-recall KPs with a recorded reason), meeting the "真正的演示" quality bar: mechanism itself visible per step, boundary-case branch coverage, user-operable.
- New quizzes → quiz-form HTML with `quizKey` (+ per-question `assert` field) + `restoreData` slot + restore JS + per-question `feedback` slots + `gradingSummary`; every `data-assert` must resolve to the chapter's 断言清单 at generation time.
- New stage-total → web-research-grounded, six types, citations.
- New stage transitions → stage-handoff (fresh session).
- Extra delivery gates: assertion-coverage check, formatting gate, and a `--vscode-`/`icube-` contamination guard (a generated chapter once shipped with ~1900 lines of accidentally-pasted IDE CSS).

The `schema_version` marker exists so the AI knows, on resume, that this workspace has been upgraded — it should NOT attempt to "fix" old files (they're intentionally left as-is), and should generate new content per the current spec.

## Grading old quizzes (those without quizKey)

When the AI grades an **old** quiz HTML that lacks `quizKey` (because it predates the feature and upgrade didn't patch it), it falls back gracefully:
- Read the quiz HTML, extract each question's `data-qid`/`data-kp`/`data-type`/`data-points` from the `<fieldset>` attributes (these exist even in older HTMLs).
- Read the question text + options to determine the correct answer (objective) or scoring dimensions (subjective).
- Grade as usual against the user's answers.json.
- Write grading back: grading.json + per-question feedback slots IF they exist in the HTML; if the old HTML lacks feedback slots, just write grading.json and report the breakdown in chat.

This fallback is less stable than reading quizKey, but it works. The user can choose to regenerate a specific quiz (via the normal rebuild flow) if they want the full feature set on that chapter — but upgrade never forces it.

## What upgrade does NOT do

- Does NOT scan or modify existing quiz/chapter HTMLs.
- Does NOT rebuild quizKey for old quizzes.
- Does NOT migrate old markdown files.
- Does NOT change the learning plan, stages, progress, or any grading/answers data.
- Does NOT enter the learning flow (user runs `/learning-loop` separately to continue).
