---
name: git-commit
description: "创建高质量的 git commit：审查/暂存目标变更，拆分为逻辑提交，并编写清晰的 commit message（包括 Conventional Commits）。当用户请求提交、编写 commit message、暂存变更或拆分多个提交时使用。"
---

# Git Commit

## 目标
使提交易于审查且安全发布：
- 只包含目标变更
- 提交范围逻辑清晰（必要时拆分）
- commit message 描述变更内容和原因

## 需要询问的输入（如缺失）
- 单个提交还是多个提交？（不确定时：默认在有无关变更时拆分为多个小提交）
- 提交风格：需要使用 Conventional Commits。
- 任何规则：最大 subject 长度、必需的 scope。

## 工作流程（检查清单）
1) 暂存前检查工作区
   - `git pull`
   - `git status`
   - `git diff`（未暂存）
   - 如果变更较多：`git diff --stat`
2) 确定提交边界（必要时拆分）
   - 按以下维度拆分：feature vs refactor、backend vs frontend、formatting vs logic、tests vs prod code、dependency bumps vs behavior changes
   - 如果变更混合在同一文件中，计划使用 patch staging
3) 只暂存属于下一次提交的内容
   - 对于混合变更，优先使用 patch staging：`git add -p`
   - 取消暂存 hunk/文件：`git restore --staged -p` 或 `git restore --staged <path>`
4) 审查实际将要提交的内容
   - `git diff --cached`
   - 合理性检查：
     - 无 secret 或 token
     - 无意外的 debug logging
     - 无无关的 formatting 改动
5) 用 1-2 句话描述暂存的变更（编写 message 之前）
   - "变更了什么？" + "为什么？"
   - 如果无法清晰描述，提交可能太大或太混乱；返回步骤 2
6) 编写 commit message
   - 使用 Conventional Commits（必需）：
     - `type(scope): short summary`
     - 空行
     - body（what/why，不是实现日记）
     - footer（BREAKING CHANGE）如需要
   - 优先使用编辑器编写多行 message：`git commit -v --no-verify`
   - 如有帮助，使用 `references/commit-message-template.md`
7) 运行最小相关的验证
   - 在进行下一步之前，运行仓库最快的有意义的检查（unit tests、lint 或 build）
8) 重复处理下一次提交，直到工作区干净

## 交付物
提供：
- 最终的 commit message(s)
- 每次提交的简短摘要（what/why）
- 用于暂存/审查的命令（至少：`git diff --cached`，以及运行的任何测试）
