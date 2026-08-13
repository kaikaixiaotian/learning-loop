---
description: 更新 learning-loop skill——从 GitHub 拉取最新代码并同步本地安装
argument-hint: "update"
---

`/learning-loop` 命令目前**仅用于自更新**。学习本身请直接用自然语言描述主题（如「我想学 React」），skill 会自动触发。

## 按参数分支

依据 `$ARGUMENTS`：

### 分支：参数为 `update` —— 更新本地 skill

只做"读远端 + 拉取 + 复制"，**绝不触碰用户的学习工作区 `*-learning/` 和 meta.json**。

1. **定位 skill 安装目录** `skillDir = ~/.agents/skills/learning-loop`。
2. **判断它是不是 git 克隆**（即 `$skillDir/.git` 是否存在）：
   - **是克隆**：先读当前版本（`$skillDir/SKILL.md` frontmatter 的 `version`）记为 `旧版本`；然后在 `$skillDir` 执行 `git pull --ff-only`（远端为 https://github.com/kaikaixiaotian/learning-loop.git ）。
   - **不是克隆**（旧的复制式安装）：把 `$skillDir` 备份为 `$skillDir.bak.<时间戳>` 后删除，再 `git clone https://github.com/kaikaixiaotian/learning-loop.git "$skillDir"`；`旧版本 = "(复制式安装)"`。
3. **同步命令自身**：把 `$skillDir/commands/learning-loop.md` 复制到 `~/.zcode/commands/learning-loop.md`（让本命令也保持最新）。若源文件不存在（远端尚未包含 commands/），跳过并提示用户"命令源尚未同步，本次仅更新 skill 本体"。
4. **报告版本**：读更新后的 `version` 记为 `新版本`：
   - `新版本 != 旧版本` → 「✅ 已从 v旧版本 升级到 v新版本」
   - 相同 → 「✅ 已是最新 v新版本」
   并提示「请新开一个会话以加载最新 skill」。
5. **冲突处理**：若 `git pull --ff-only` 因本地改动/分叉失败，**不要 `--force` 或强制覆盖**，把 git 的错误原样转告用户，让其自行处理。

### 分支：参数为空 或 其它值
回复一段话：本命令目前只支持 `update`（更新 skill）。要学习，请直接用自然语言说出主题，例如「我想学 Docker」「教我梯度下降」。

## 约束
- 跨平台：只用 git 通用命令；路径用 `~`（Bash/PowerShell 均展开）或 Bash 风格。
- 只读 + 拉取 + 复制；不修改 `*-learning/` 工作区、不动 meta.json、不改用户其它配置。
