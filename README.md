# Claude Code Skill 安装指南（中国大陆网络环境）

> GitHub HTTPS 被墙时，Claude Code 安装 Skill 的标准流程。3 分钟搞定，不用反复踩坑。

## 一句话

**GitHub 443 端口不通，但 22 端口（SSH）能通。把 git 协议切到 SSH，一切照旧。**

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

> 以上 4 步只需做一次，以后装任何 Skill 都不用再管网络问题。

---

## 安装 Skill（三种方式）

### 方式一：一键安装（推荐）

```bash
npx skills add <owner/repo> -y

# 大仓库加超时（默认 300 秒不够用）
SKILLS_CLONE_TIMEOUT_MS=600000 npx skills add <owner/repo> -y
```

### 方式二：手动 clone + 安装

```bash
cd /tmp
gh repo clone <owner/repo> -- --depth 1
npx skills add /tmp/<repo-name> -y
```

### 方式三：纯手动（什么工具都不依赖）

```bash
cd /tmp
gh repo clone <owner/repo> -- --depth 1
cp -r /tmp/<repo-name> ~/.claude/skills/<repo-name>
```

---

## 常见坑

| 问题 | 原因 | 解决 |
|------|------|------|
| `Failed to connect to github.com port 443` | 走了 HTTPS 协议 | `gh config set git_protocol ssh` |
| `Permission denied (publickey)` | SSH Key 没配或没加到 GitHub | `gh ssh-key add ~/.ssh/id_ed25519.pub` |
| Clone 超时 | 仓库太大 | `SKILLS_CLONE_TIMEOUT_MS=600000` |
| `shallow.lock: File exists` | 上次 git 进程僵死 | `taskkill /F /IM git.exe` 后再来 |
| 下载很慢 | SSH 带宽有限 | **不要并行 3 个以上 clone**，逐个来反而更快 |
| `npx skills add` 也超时 | 默认 300s 对大仓库不够 | 设更大超时或先手动 `gh repo clone` |

## 为什么 HTTPS 不通但 SSH 能通

GFW 对 GitHub 的封锁策略通常是 **SNI 封锁 + IP 阻断**，针对的是 TLS/HTTPS 流量。SSH 走 22 端口，不在封锁范围内。所以：

- ❌ `https://github.com/...` → 超时
- ✅ `git@github.com:...` → 能通

---

## 实测可用的 Skill 推荐

| Skill | 仓库 | 用途 |
|-------|------|------|
| huashu-design | alchaincyf/huashu-design | 全能设计：HTML 幻灯片 + PPTX + MP4 动画 + 设计评审 |
| html-ppt-skill | lewislulu/html-ppt-skill | 36 主题 × 31 布局网页 PPT |
| guizang-ppt-skill | op7418/guizang-ppt-skill | 杂志风 / 瑞士国际主义风网页 PPT |
| ppt-master | hugohe3/ppt-master | 原生 .pptx 生成（大仓库 ~150MB） |
| frontend-slides | zarazhangrui/frontend-slides | 零依赖 HTML 幻灯片 + PPT 转换 |
| axi-front-design | Diddysister/axi-front-design | 反 AI 俗套的设计 Skill |

---

## 参考

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [GitHub CLI 文档](https://cli.github.com/manual/)
- 本指南基于 Windows 11 + Git Bash 环境实测，macOS/Linux 同理。

## License

MIT — 随便用，随便改，随便转。
