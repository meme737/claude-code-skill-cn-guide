---
name: skill-install-workflow
description: Claude Code skill 安装标准流程——GitHub被墙环境下的固定操作步骤
metadata: 
  node_type: memory
  type: reference
  originSessionId: 2af39351-c8d9-4593-aaf7-4d2514091e08
---

# Skill 安装标准流程（GitHub 被墙环境）

## ⚡ 核心规则：下载慢立马换路径

**SSH/HTTPS 直连 GitHub 一旦超过 30 秒没拉完或速度 < 0.5MB/s，立刻放弃，直接走 AtomGit 镜像。不要浪费时间反复试。**

判断标准：
- `git clone` 跑了 30 秒还没到 50%，杀
- `gh release download` 速度低于 0.5MB/s，杀
- 任何方法连续失败 2 次，不再试，换路径

## 网络现状

- **HTTPS (443)**：GitHub 直连被墙，curl/git HTTPS 全部超时
- **SSH (22)**：能通但不稳定，大文件（>50MB）容易断连
- **gh CLI**：已登录 `meme737`，默认协议已切为 SSH (`gh config set git_protocol ssh`)
- **AtomGit**：国内镜像 `atomgit.com`，速度快，优先走这个

## 安装步骤（按优先级）

### 方式一：AtomGit 国内镜像（🚀 首选，大仓库唯一解）

直接搜 `atomgit.com/<owner>/<repo>` 确认镜像存在，然后：

```bash
cd /tmp/skills && rm -rf <repo-name>
git clone --depth 1 https://atomgit.com/<owner>/<repo-name>.git
cp -r /tmp/skills/<repo-name> ~/.claude/skills/<repo-name>
# 杀僵死进程：
taskkill /F /IM git.exe 2>/dev/null; taskkill /F /IM gh.exe 2>/dev/null
```

### 方式二：npx skills add（小仓库可用，大仓库必超时）

```bash
SKILLS_CLONE_TIMEOUT_MS=600000 npx skills add <owner/repo> -y
```

`npx skills add` 默认走 SSH，小仓库（<50MB）能过，大仓库必断。

### 方式三：SSH 手动 clone（备用，仅小仓库）

```bash
cd /tmp/skills && rm -rf <repo-name>
timeout 120 git clone --depth 1 git@github.com:<owner>/<repo-name>.git
# 超时或太慢 → 立刻换 AtomGit
cp -r /tmp/skills/<repo-name> ~/.claude/skills/<repo-name>
```

### 方式四：Release 下载（仅当有 Release 且镜像也没法用）

```bash
gh release download <tag> --repo <owner/repo> --pattern '*.zip' --dir /tmp/skills
# 速度 < 0.5MB/s → 杀，换 AtomGit
```

## 已知坑

1. **下载慢第一反应换 AtomGit**：别在 SSH/HTTPS 上反复试，纯浪费时间
2. **大仓库（>50MB）直接走 AtomGit**：SSH 对大文件断连概率极高
3. **残留 lock 文件**：中途失败先 `taskkill /F /IM git.exe` 再 `rm -rf`
4. **`--depth 1` 必加**：浅克隆，减少传输量
5. **不要并行 clone**：SSH 带宽有限，并行更慢
6. **AtomGit 不是所有仓库都有镜像**：先浏览器确认 `atomgit.com/<owner>/<repo>` 存在。如果没有，再退回 SSH 硬拉 + 长超时

## 当前已安装的 Skill

| Skill | 用途 | 来源 |
|-------|------|------|
| academic-report-ppt | 大学作业 .doc + 文献验证 | 已有 |
| guizang-ppt-skill | 杂志风/Swiss风网页PPT | 已有 |
| huashu-design | 全能设计 HTML+PPTX+MP4 | 已有 |
| html-ppt-skill | 36主题×31布局网页PPT | 已有 |
| ppt-master | AI 生成原生可编辑 PPTX | AtomGit 镜像，1.3GB，2026-06-20 |
