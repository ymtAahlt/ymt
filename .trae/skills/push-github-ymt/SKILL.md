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

## ⚠️ 环境约束（必读）

**本环境（国内 Windows）GitHub 的 22 端口被运营商/公司拦截**，直接用 SSH `git@github.com:...` 会得到 `Connection refused`。

实测可用的两种方案：
1. **HTTPS remote（推荐，已验证可行）**：`https://github.com/ymtAahlt/ymt.git` — Windows 凭据管理器会缓存 token
2. **SSH over 443**：用 `ssh://git@ssh.github.com:443/ymtAahlt/ymt.git` 并设置 `GIT_SSH_COMMAND="ssh -p 443"`

本项目 `upstream` 已配置为 **HTTPS**，直接 push 即可。

## 执行步骤（按顺序，必须严格遵守）

### 1. 检查仓库状态

```powershell
git status --short
git remote -v
# upstream 应该是 https://github.com/ymtAahlt/ymt.git
```

**前置条件**：
- 当前目录必须是一个 git 仓库（有 `.git/`）
- 必须有一个 remote 叫 `upstream` 指向 GitHub 仓库 HTTPS 地址
- 用户已配置好 GitHub 凭据（Windows 凭据管理器里有 GitHub token）

**前置检查失败的处理**：
- 如果 `upstream` 不存在：`git remote add upstream https://github.com/ymtAahlt/ymt.git`
- 如果 upstream 是 SSH 且 push 报 Connection refused：改成 HTTPS 再试
- 如果 HTTPS push 报 403：需要用户在 GitHub 生成一个 PAT（有 `repo` 权限）并用 `git config credential.helper store` 或 Windows 凭据管理器缓存

### 2. 添加并提交

```powershell
git add -A
git commit -m "<清晰描述本次改动的中文 commit message>"
```

Commit message 规范：
- 使用中文（项目作者母语）
- 格式示例：`feat: 新增 17-4 API 测试工具`、`fix: 修复 17-6 进度条显示`、`docs: 更新首页分组`、`chore: 新增 push-github-ymt 技能`
- 如果没有任何改动（`git status` 为空），跳过 commit，直接进入第 3 步

### 3. 推送到 GitHub

```powershell
$env:GIT_TERMINAL_PROMPT = "0"   # 禁止交互式询问凭据，避免挂起
git push upstream <当前分支名>
```

获取当前分支名：`git branch --show-current`

**必须显式指定** `upstream` 和分支名，避免推到错误的 remote（项目同时有 Gitee `origin` 和 GitHub `upstream` 两个远端）。

如果 push 挂起超过 30 秒，说明需要凭据提示，但被沙箱拦了 — 终止命令并提示用户手动 push 或配置 token。

### 4. 验证推送结果

- 从输出中确认出现 `-> upstream/<branch>` 且 exit code 为 0
- 或出现 `Everything up-to-date`（说明没有新 commit）
- 用 WebFetch 请求 `https://github.com/ymtAahlt/ymt/commits/master` 查看最新 commit hash 是否与本地 `git rev-parse --short HEAD` 一致

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
| `Connection refused` + 22 端口 | 22 被封，切 HTTPS 地址，不要用默认 SSH |
| `ssh.github.com:443` 也连不上 | 只用 HTTPS；如果 HTTPS 也挂那就是 GitHub 整体不可达 |
| `Permission denied (publickey)` | 提示用户把本机公钥加到 GitHub Settings → SSH Keys；**不要**索要私钥 |
| `403 Authentication failed` | HTTPS 需要 GitHub PAT；提示用户生成一个有 `repo` 权限的 token |
| `rejected: non-fast-forward` | 先 `git fetch upstream && git log ..upstream/<branch>` 看差异，**不要**无脑 `--force`；与用户确认后再决定 rebase/merge |
| LF/CRLF 警告 | Windows 正常现象，忽略即可，不是错误 |
| push 长时间无输出挂起 | 大概率在等凭据输入，用 StopCommand 终止，提示用户先手动 push 一次让凭据存进 Windows 凭据管理器 |

## 关键常量

| 项目 | 值 |
|---|---|
| GitHub 仓库 | `https://github.com/ymtAahlt/ymt` |
| GitHub SSH (443) | `ssh://git@ssh.github.com:443/ymtAahlt/ymt.git` |
| 本地 upstream | **HTTPS** `https://github.com/ymtAahlt/ymt.git`（已验证） |
| 本地 origin | Gitee `git@gitee.com:yao-mingtao1/111.git` |
| 默认分支 | `master` |
| GitHub Pages | `https://ymtaahlt.github.io/ymt/`（需在网页端开启，当前 404） |

## 示例：一次完整执行

```powershell
# 1. 检查
git remote -v
git status --short

# 2. 提交（有改动时）
git add -A
git commit -m "feat: 完善首页 + 新增 17 章异步专题"

# 3. 推送
$env:GIT_TERMINAL_PROMPT = "0"
git push upstream master
# 期望输出：34c4c1e..22ca267  master -> master

# 4. 验证（可选）
git rev-parse --short HEAD
# 对照 https://github.com/ymtAahlt/ymt/commits/master 上的最新 hash
```

## 已验证的推送历史

| 时间 | Commit | 说明 |
|---|---|---|
| 2026-11-02 | `22ca267` | chore: 新增 push-github-ymt 技能 |
| 2026-11-02 | `34c4c1e` | feat: 完善首页 + 新增 17 章异步专题 |
| 2026-10-15 | `e1eba86` | 新增 12-1.html CSS 变量页面及项目全部文件 |
