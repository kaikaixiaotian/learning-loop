# Changelog

记录 `learning-loop` skill 的版本变更。版本号以 `SKILL.md` frontmatter 的 `version` 字段为**单一来源**。
参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/) 与 [语义化版本](https://semver.org/lang/zh-CN/)。

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
