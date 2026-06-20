# Claude Code Skill 安装指南（中国大陆网络环境）

> GitHub HTTPS 被墙时，Claude Code 安装 Skill 的标准流程。3 分钟搞定，不用反复踩坑。

## 一句话

**小仓库走 SSH（22端口能通）。大仓库（>50MB）SSH 必断，直接走 AtomGit 国内镜像。**

## 前提条件（一次性配置）

### 1. 安装 GitHub CLI 并登录

```bash
# Windows: winget install GitHub.cli
# macOS:   brew install gh

gh auth login
# 选 GitHub.com → HTTPS → 用浏览器登录
```

### 2. 生成 SSH Key 并添加到 GitHub

```bash
ssh-keygen -t ed25519 -C "your@email.com" -f ~/.ssh/id_ed25519 -N ""
gh ssh-key add ~/.ssh/id_ed25519.pub --title "Claude-Code"
```

### 3. 切到 SSH 协议

```bash
gh config set git_protocol ssh
```

### 4. 验证

```bash
ssh -T git@github.com
# 预期输出: Hi <用户名>! You've successfully authenticated...
```

> 以上 4 步只需做一次。

---

## 安装 Skill（按速度排序）

### 🚀 方式一：AtomGit 国内镜像（大仓库首选，最快最稳）

SSH 对大仓库（>50MB）极不稳定，反复断连。AtomGit 是国内代码托管平台，HTTPS 满速下载。

```bash
# 1. 先确认镜像存在：浏览器打开 https://atomgit.com/<owner>/<repo>
# 2. 浅克隆（--depth 1 只拉最新版本）
mkdir -p /tmp/skills && cd /tmp/skills && rm -rf <repo-name>
git clone --depth 1 https://atomgit.com/<owner>/<repo-name>.git
# 3. 手动安装到 Claude Code
cp -r /tmp/skills/<repo-name> ~/.claude/skills/<repo-name>
# 4. 杀僵死进程（如果中途失败过）
taskkill /F /IM git.exe 2>/dev/null; taskkill /F /IM gh.exe 2>/dev/null
```

> 实测 ppt-master（1.3GB）：SSH 反复断连 >5 次，AtomGit 一次拉完 30 秒。

### 方式二：npx skills add（仅小仓库 <50MB）

```bash
npx skills add <owner/repo> -y

# 稍大的仓库加超时
SKILLS_CLONE_TIMEOUT_MS=600000 npx skills add <owner/repo> -y
```

> `npx skills add` 默认走 SSH。小仓库够用，大仓库必超时。

### 方式三：SSH 手动 clone（备用，仅小仓库）

```bash
cd /tmp/skills && rm -rf <repo-name>
git clone --depth 1 git@github.com:<owner>/<repo-name>.git
cp -r /tmp/skills/<repo-name> ~/.claude/skills/<repo-name>
```

---

## ⚡ 核心规则：慢就换，别死磕

| 信号 | 动作 |
|------|------|
| SSH clone 超过 **30 秒** 没到 50% | 立刻杀，换 AtomGit |
| 任何方法连续 **失败 2 次** | 不再试，换路径 |
| 仓库 **>50MB** | 直接走 AtomGit，别在 SSH 上浪费时间 |
| gh release download 速度 **< 0.5MB/s** | 杀，换 AtomGit |

---

## 常见坑

| 问题 | 原因 | 解决 |
|------|------|------|
| `Failed to connect to github.com port 443` | 走了 HTTPS 协议 | `gh config set git_protocol ssh` |
| `Permission denied (publickey)` | SSH Key 没配或没加到 GitHub | `gh ssh-key add ~/.ssh/id_ed25519.pub` |
| Clone 超时 | 大仓库 SSH 扛不住 | **换 AtomGit 镜像**，别设更大超时硬撑 |
| `shallow.lock: File exists` | 上次 git 进程僵死 | `taskkill /F /IM git.exe` 后再来 |
| 下载很慢 | SSH 带宽有限 | 不要并行 3 个以上 clone |
| 大仓库 SSH 反复断连 | SSH 对大文件传输不稳定 | **直接 AtomGit**，别在 SSH 上反复试 |
| AtomGit 没有镜像 | 不是所有仓库都有 | 浏览器先确认 `atomgit.com/<owner>/<repo>`，没有就退回 SSH 硬拉 |

## 为什么 HTTPS 不通但 SSH 能通

GFW 对 GitHub 的封锁策略通常是 **SNI 封锁 + IP 阻断**，针对的是 TLS/HTTPS 流量。SSH 走 22 端口，不在封锁范围内。所以：

- ❌ `https://github.com/...` → 超时
- ✅ `git@github.com:...` → 能通

但 SSH 带宽有限 + 连接不稳定，大文件传输容易断。AtomGit 走国内 CDN，HTTPS 满速，是大仓库的最优解。

---

## 实测可用的 Skill 推荐

| Skill | 仓库 | 用途 |
|-------|------|------|
| huashu-design | alchaincyf/huashu-design | 全能设计：HTML 幻灯片 + PPTX + MP4 动画 + 设计评审 |
| html-ppt-skill | lewislulu/html-ppt-skill | 36 主题 × 31 布局网页 PPT |
| guizang-ppt-skill | op7418/guizang-ppt-skill | 杂志风 / 瑞士国际主义风网页 PPT |
| ppt-master | hugohe3/ppt-master | 原生 .pptx 生成（1.3GB，走 AtomGit） |
| frontend-slides | zarazhangrui/frontend-slides | 零依赖 HTML 幻灯片 + PPT 转换 |
| axi-front-design | Diddysister/axi-front-design | 反 AI 俗套的设计 Skill |

---

## 参考

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [GitHub CLI 文档](https://cli.github.com/manual/)
- 本指南基于 Windows 11 + Git Bash 环境实测，macOS/Linux 同理。

## License

MIT — 随便用，随便改，随便转。
