# Visualization Protocol

The learning-loop skill can generate **interactive HTML visualizations** for concepts that benefit from them. This file defines *when* to visualize, *how* to embed, and *how to verify* the HTML actually works. Read before any chapter that might warrant a visualization, and before dispatching a visualization-generating subagent.

## Why visualize at all

Some concepts are inherently spatial, temporal, or stateful — words undersell them. A state machine drawn as clickable nodes; a sorting algorithm stepped through; a git three-area model showing files flowing between zones. For these, an interactive demo teaches in 30 seconds what paragraphs cannot. But not every concept benefits — forcing a visualization onto a definition-heavy concept adds noise. So the decision is **per-KP, with a judgment rule**.

## When to visualize — the judgment rule

For each KP in a chapter, the planner subagent evaluates it against these signals. **Visualize if ≥2 signals fire.** Do not visualize if 0–1 fire (the prose + six-element explanation is enough; a forced viz wastes effort and distracts).

| Signal | Meaning | Example |
|--------|---------|---------|
| **Stateful / multi-step** | The concept involves transitions over time or steps | HTTP cache fresh→stale→revalidate; Promise pending→fulfilled |
| **Spatial / structural** | Relationships between parts matter (trees, graphs, layers) | DOM tree; git object graph (blob→tree→commit) |
| **Data-flow** | Things move between zones/actors | request→cache→origin; working tree→index→repo |
| **Parameter-sensitive** | Behavior changes visibly as an input changes | `max-age` slider changing freshness; recursion depth |
| **Counter-intuitive when static** | Reading it wrong is easy until you see it move | event loop; backpressure |
| **Algorithmic** | A procedure with discrete steps | merge sort passes; BFS queue evolution |

Counter-examples (do NOT visualize): pure definitions ("什么是闭包"的一句话定义), syntax memorization, historical context, philosophical distinctions. If the concept is "remember this term means X", prose wins.

The planner records its decision per KP in a `visualization_decisions` block (see `references/subagent-protocol.md`):

```
visualization_decisions:
  KP1 (cache definition): skip — 0 signals (pure definition)
  KP2 (max-age): visualize — signals: parameter-sensitive, stateful
  KP3 (强制缓存流程): visualize — signals: data-flow, multi-step, counter-intuitive
  KP4 (no-cache vs no-store): skip — 1 signal (contrast); prose + table is clearer
```

A chapter typically warrants 0–3 visualizations. Zero is fine — do not manufacture one to "look complete".

## How to embed in the chapter doc

The chapter doc is now itself an HTML file (read-mode, per `references/html-format.md`), so each visualization is **embedded inline** as a component via an `<iframe>` — not a link that opens a new tab. The visualization is still saved as a standalone `.html` file: it opens fine on its own AND is loaded into the chapter page, so the user interacts with it in-place while reading.

**File location:** `<workspace>/chapters/viz/stageN-chXX-<kp-slug>.html` (one file per visualized KP; `viz/` subfolder keeps them organized). The file remains double-click-openable and reusable; the iframe just loads it inline.

**Embed format in the chapter doc HTML** — place the `<figure class="viz">` inline within the relevant 核心概念 subsection, right after that concept's six-element `<ol class="elements">`, so the user reaches it at the moment of learning that concept:

```html
<h3>2. max-age 相对新鲜期</h3>
<ol class="elements">
  <li><span class="el-label">① 精确定义</span><div class="el-body">…</div></li>
  …
  <li><span class="el-label">⑥ 与相关概念对比</span><div class="el-body">…</div></li>
</ol>
<figure class="viz">
  <figcaption>🖼️ 交互演示：max-age 新鲜期滑块</figcaption>
  <iframe src="./viz/stage1-ch01-max-age.html" loading="lazy" title="max-age 新鲜期滑块演示"></iframe>
  <a class="viz-open" href="./viz/stage1-ch01-max-age.html" target="_blank">在新标签页打开 ↗</a>
</figure>
```

The `<figcaption>` carries the 🖼️ + demo name (scannable); the borderless `<iframe>` loads the demo inline; the `.viz-open` link is a fallback in case the iframe is ever blocked. `<figure class="viz">` is styled in the read-mode HTML skeleton (caption bar + seamless iframe + fallback link). Only render the `<figure>` for KPs you actually visualise — never add an empty placeholder.

If a chapter has zero visualizations, do NOT add a placeholder — just omit. Silence is correct (it means the concepts didn't warrant it).

## Interaction requirements (per user's confirmed choice: 可交互演示)

Visualizations must be **genuinely interactive**, not static diagrams. Minimum bar:

- At least one of: button (step/play/reset), slider (parameter), or click (highlight node / reveal). A pure static SVG with no controls does NOT count — that's a static diagram, skip the viz and use prose instead.
- State changes must be **visible** (color/position/size change), not just logged to console.
- Self-contained: single `.html` file, all CSS/JS inline, no external dependencies (no CDN — the user may open it offline). Vanilla JS or inline `<script>` only.
- Clear labels in Chinese matching the chapter's terminology.
- A "重置" (reset) control so the user can replay.

Encouraged but not required: play/pause for animations, step counters, before/after state comparison.

**Tech constraints:** Vanilla HTML/CSS/JS only. No frameworks (React/Vue), no build step, no npm. Use `<canvas>` or SVG or styled `<div>`s as fits the concept. File should open by double-click — zero setup.

## Quality verification (per user's confirmed choice: JS 静态检查)

The main agent **must verify** every generated HTML before handing the chapter to the user. Visualizations that don't render or have JS errors are worse than no visualization — they erode trust. Verification is a fast static check, not a full browser run:

**Run this check after the planner subagent returns a viz file:**

1. **Syntax check:** extract the `<script>` content and run it through Node's parser to confirm no syntax errors:
   ```bash
   node --check <extracted-script.js>
   ```
   Or use `node -e` with the script wrapped. A syntax error here means the file is broken — reject and have the subagent regenerate.

2. **Element existence check:** grep the HTML for the controls the concept requires — if it's a slider viz, `<input type="range">` must exist; if it's a step viz, the step button's id must exist. Missing required elements = incomplete viz.

3. **No undefined references:** grep for common bug patterns — `getElementById('x')` where `x` isn't in the HTML; function calls to functions not defined in the script.

If any check fails: **do not hand the broken viz to the user.** Either (a) re-dispatch the subagent with the specific failure pointed out, or (b) drop the visualization for that KP and replace the link in the chapter doc with a prose note ("本概念建议自行画状态图理解"). Never ship a viz that errors on open.

If all checks pass: the viz is good. The link stays in the chapter doc.

**Why not browser-screenshot verification (the option not chosen):** it's heavier and slower for every chapter; static checks catch the vast majority of "won't render" failures (syntax errors, missing elements) at a fraction of the cost. Visual/layout issues that slip through static checks are acceptable — the user will report them and we fix.

## What the visualization subagent returns

The planner subagent generates the HTML directly (it's the same agent that knows the concept). It returns, per visualized KP:

```
visualization_decisions:
  <per-KP decision block as above>

viz_files_written:
  - path: <abs path>/chapters/viz/stageN-chXX-<kp-slug>.html
    kp: KP3
    concept: 强制缓存流程
    interaction: step buttons (next/prev/reset) + auto-play
    verified: <true|false — set true ONLY if the subagent self-ran the static checks; main agent re-verifies anyway>
```

The main agent then runs the verification itself (don't trust the subagent's self-check alone) and either keeps or drops each viz.
