---
name: commit-project-111
description: Use when the user asks to commit and push the current project to the 111 repository, e.g. "提交项目", "commit 并 push", "推送到 gitee", "同步到 111", or "推送到 yao-mingtao1/111". Commits all current changes and pushes to the Gitee remote git@gitee.com:yao-mingtao1/111.git (SSH).
---

# 提交项目

将当前项目通过 `git commit` 并 `git push` 提交到 Gitee 仓库：
`git@gitee.com:yao-mingtao1/111.git`（SSH 方式）

## 前置检查

1. 确认是 git 仓库：
   ```
   git rev-parse --is-inside-work-tree
   ```
   若不是仓库，先执行 `git init`。

2. 确认远程 `origin` 指向目标仓库：
   ```
   git remote -v
   ```
   - 若没有 `origin`：`git remote add origin git@gitee.com:yao-mingtao1/111.git`
   - 若存在但地址不符：`git remote set-url origin git@gitee.com:yao-mingtao1/111.git`

## 提交步骤

3. 查看改动，确认要提交的内容：
   ```
   git status
   git diff
   ```

4. 暂存并提交（提交信息根据实际改动概括，不要空提交）：
   ```
   git add -A
   git commit -m "<简明描述本次改动>"
   ```
   若没有任何改动可提交，告知用户无需提交并停止。

5. 确认当前分支（默认 `master`，若远程使用 `main` 则切换或重命名）：
   ```
   git branch --show-current
   ```

6. 推送到远程：
   ```
   git push -u origin <当前分支>
   ```

## 注意事项

- 仅在用户明确要求提交时才执行本流程；不要自动提交。
- 绝不提交密钥、密码或敏感信息。
- 推送前若远程有新提交导致冲突，先 `git pull --rebase origin <分支>` 再推送。
- 推送遇到认证失败时，提醒用户配置 Gitee SSH key（首次可执行 `ssh -T git@gitee.com` 验证）。
- 完成后向用户报告：提交信息、分支名、推送结果。
