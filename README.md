# Claude Code Skill 安装指南（中国大陆网络环境）

> GitHub HTTPS 被墙时，Claude Code 安装 Skill 的标准流程。

## 一句话

**四种方式按速度排：浏览器下载 > AtomGit 镜像 > SSH clone > npx。Git 挂了别死磕，立刻换通道。**

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

### 🚀 方式一：浏览器下载（通用最快，绕过 GFW）

Claude Code 内置浏览器（CloakBrowser/Playwright）走不同的网络通道，能绕过 GFW 直连 GitHub。**SSH/HTTPS/git 全挂的时候，浏览器反而是最快的。**

```
# 直接在 Claude Code 对话中说：
"用浏览器打开 https://github.com/<owner>/<repo>/archive/refs/heads/main.zip"
# 文件自动保存到 ~/.cloakbrowser/playwright-mcp/<repo>-main.zip
# 然后：
unzip -q ~/.cloakbrowser/playwright-mcp/<repo>-main.zip -d /tmp/skills/
mv /tmp/skills/<repo>-main /tmp/skills/<repo-name>
cp -r /tmp/skills/<repo-name> ~/.claude/skills/<repo-name>
```

> 实测 headroom（54MB 仓库，41MB zip）：SSH/HTTPS/镜像全部失败，浏览器 30 秒拉完。

### 🚀 方式二：AtomGit 国内镜像（有镜像时极快）

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

### 方式三：SSH 手动 clone（仅小仓库 <50MB）

```bash
cd /tmp/skills && rm -rf <repo-name>
git clone --depth 1 git@github.com:<owner>/<repo-name>.git
cp -r /tmp/skills/<repo-name> ~/.claude/skills/<repo-name>
```

### 方式四：npx skills add（兜底，小仓库适用）

```bash
npx skills add <owner/repo> -y
```

---

## ⚡ 核心规则：慢就换，别死磕

| 信号 | 动作 |
|------|------|
| SSH clone 超过 **30 秒** 没到 50% | 立刻杀，换浏览器下载 |
| 任何方法连续 **失败 2 次** | 不再试，换下一优先级 |
| 仓库 **>50MB** | 跳过 SSH，直接浏览器或 AtomGit |
| gh release download 速度 **< 0.5MB/s** | 杀，换浏览器下载 |
| AtomGit 没镜像 | 直接浏览器下载，别在 SSH 上浪费时间 |

---

## 常见坑

| 问题 | 原因 | 解决 |
|------|------|------|
| `Failed to connect to github.com port 443` | 走了 HTTPS 协议 | `gh config set git_protocol ssh` |
| `Permission denied (publickey)` | SSH Key 没配或没加到 GitHub | `gh ssh-key add ~/.ssh/id_ed25519.pub` |
| Clone 超时 | 大仓库 SSH 扛不住 | **换浏览器下载或 AtomGit** |
| `shallow.lock: File exists` | 上次 git 进程僵死 | `taskkill /F /IM git.exe` 后再来 |
| 大仓库 SSH 反复断连 | SSH 对大文件传输不稳定 | **直接浏览器下载** |
| 所有方法都失败 | GFW 全面封锁 git/SSH/HTTPS | **浏览器下载走不同通道，实测能过** |

## 为什么浏览器能过但 SSH/git 不能

GFW 对 GitHub 的封锁层次不同：

- ❌ `github.com:443`（HTTPS）→ SNI 封锁 + IP 阻断
- ❌ `github.com:22`（SSH）→ 不稳定，大文件断连
- ❌ `api.github.com`（API）→ 限速极慢
- ✅ **浏览器（CloakBrowser）** → 走不同的网络栈和代理策略，能直连

---

## 实测可用的 Skill 推荐

| Skill | 仓库 | 用途 |
|-------|------|------|
| huashu-design | alchaincyf/huashu-design | 全能设计：HTML 幻灯片 + PPTX + MP4 动画 + 设计评审 |
| html-ppt-skill | lewislulu/html-ppt-skill | 36 主题 × 31 布局网页 PPT |
| guizang-ppt-skill | op7418/guizang-ppt-skill | 杂志风 / 瑞士国际主义风网页 PPT |
| ppt-master | hugohe3/ppt-master | 原生 .pptx 生成（1.3GB，走 AtomGit） |
| headroom | chopratejas/headroom | LLM token 压缩 60-95%（走浏览器下载） |
| frontend-slides | zarazhangrui/frontend-slides | 零依赖 HTML 幻灯片 + PPT 转换 |
| axi-front-design | Diddysister/axi-front-design | 反 AI 俗套的设计 Skill |

---

## 参考

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [GitHub CLI 文档](https://cli.github.com/manual/)
- 本指南基于 Windows 11 + Git Bash 环境实测，macOS/Linux 同理。

## License

MIT — 随便用，随便改，随便转。
