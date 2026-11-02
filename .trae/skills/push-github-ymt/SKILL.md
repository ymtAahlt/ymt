---
name: "push-github-ymt"
description: "Commits all changes and pushes to GitHub ymtAahlt/ymt (upstream). Invoke when user says 'push to github', 'submit to ymt', '发布到 github', or wants to deploy this project to the GitHub mirror."
---

# Push to GitHub `ymtAahlt/ymt`

本技能用于把当前项目的所有本地改动一次性提交并推送到 GitHub 仓库 `https://github.com/ymtAahlt/ymt`（本地 remote 名称为 `upstream`）。

## 何时调用

- 用户说"发布到 GitHub"、"推到 GitHub"、"提交到 ymt"、"push 一下"等类似意图
- 完成了一批页面/功能开发，需要同步到 GitHub 备份或部署 Pages
- 本地有未提交改动，希望一次性提交

## 执行步骤（按顺序，必须严格遵守）

### 1. 检查仓库状态

```powershell
git status --short
git remote -v
```

**前置条件**：
- 当前目录必须是一个 git 仓库（有 `.git/`）
- 必须有一个 remote 叫 `upstream` 指向 `https://github.com/ymtAahlt/ymt.git` 或 `git@github.com:ymtAahlt/ymt.git`
- 用户已配置好 GitHub 凭据（SSH 公钥已添加 或 HTTPS 凭据已缓存）

**前置检查失败的处理**：
- 如果 `upstream` 不存在：先执行 `git remote add upstream https://github.com/ymtAahlt/ymt.git`
- 如果凭据失败：提示用户重新配置 SSH key 或 GitHub token，**不要**在脚本里硬编码任何密码

### 2. 添加并提交

```powershell
git add -A
git commit -m "<清晰描述本次改动的中文 commit message>"
```

Commit message 规范：
- 使用中文（项目作者母语）
- 格式示例：`feat: 新增 17-4 API 测试工具`、`fix: 修复 17-6 进度条显示`、`docs: 更新首页分组`
- 如果没有任何改动（`git status` 为空），跳过 commit，直接进入第 3 步

### 3. 推送到 GitHub

```powershell
git push upstream <当前分支名>
```

获取当前分支名：`git branch --show-current`

**必须显式指定** `upstream` 和分支名，避免推到错误的 remote（项目同时有 Gitee `origin` 和 GitHub `upstream` 两个远端）。

### 4. 验证推送结果

- 从输出中确认出现 `-> upstream/<branch>` 且 exit code 为 0
- 可选：用 WebFetch 请求 `https://github.com/ymtAahlt/ymt` 首页看最近一次 commit hash 是否与本地一致

### 5. 向用户汇报

输出格式：
```
✅ 已推送到 GitHub
   仓库：https://github.com/ymtAahlt/ymt
   分支：<branch>
   Commit：<short hash> <message>
   Pages：https://ymtaahlt.github.io/ymt/ （如已开启）
```

## 错误处理

| 错误现象 | 处理方式 |
|---|---|
| `Connection refused` / SSH 超时 | 尝试改用 HTTPS 地址：`https://github.com/ymtAahlt/ymt.git` |
| `Permission denied (publickey)` | 提示用户把本机公钥加到 GitHub Settings → SSH Keys；**不要**在对话里索要私钥内容 |
| `rejected: non-fast-forward` | 先 `git fetch upstream && git log ..upstream/<branch>` 看差异，**不要**无脑 `--force`；与用户确认后再决定 rebase/merge |
| `403 Authentication failed` | HTTPS 模式下需要 GitHub Personal Access Token；提示用户去 GitHub 生成一个有 `repo` 权限的 token 并用 `git config credential.helper store` 缓存 |
| LF/CRLF 警告 | Windows 正常现象，忽略即可，不是错误 |

## 关键常量

| 项目 | 值 |
|---|---|
| GitHub 仓库 | `https://github.com/ymtAahlt/ymt` |
| 本地 remote 名 | `upstream` |
| Gitee remote 名 | `origin` |
| 默认分支 | `master`（若仓库后来改名为 main 则同步调整） |
| GitHub Pages 地址 | `https://ymtaahlt.github.io/ymt/` |

## 示例：一次完整执行

```powershell
git remote -v
# 确认 upstream 存在 → https://github.com/ymtAahlt/ymt.git

git add -A
git commit -m "feat: 完善首页 + 新增 17 章异步专题"
git push upstream master
# 输出：e1eba86..34c4c1e  master -> master
```
