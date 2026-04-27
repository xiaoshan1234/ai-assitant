# OpenCode 入门：安装、使用与更新

> 参考：[OpenCode Download](https://opencode.ai/download)、[CLI 文档](https://opencode.ai/docs/cli/)、[Windows (WSL)](https://opencode.ai/docs/windows-wsl/)

---

## 一、安装

### 1. 通用安装脚本（Linux / macOS；Windows 多在 Git Bash / WSL 中使用）

```bash
curl -fsSL https://opencode.ai/install | bash
```

### 2. 包管理器

| 方式 | 命令 |
|------|------|
| npm | `npm i -g opencode-ai` |
| Bun | `bun add -g opencode-ai` |
| Homebrew | `brew install anomalyco/tap/opencode` |
| Arch（paru） | `paru -S opencode` |

### 3. Windows

- **npm / Bun**：在 PowerShell 或 CMD 中全局安装（同上表）。
- **桌面端（Beta）**：从 [Download](https://opencode.ai/download) 下载 Windows 安装包。
- **WSL**：在 WSL 内使用 `curl | bash` 或 npm，体验与 Linux 一致；详见 [Windows (WSL)](https://opencode.ai/docs/windows-wsl/)。

### 4. Linux

- 除脚本与 **npm / Bun** 外，Arch 可用 **paru**；也可从 [Download](https://opencode.ai/download) 获取 **.deb / .rpm** 等（以页面为准）。

### 5. 验证安装

```bash
opencode --version
```

---

## 二、使用（CLI）

默认**无参数**启动终端界面（TUI）：

```bash
opencode
```

指定项目目录：

```bash
opencode /path/to/project
```

### 常用命令

| 目的 | 示例 |
|------|------|
| 非交互执行一句提示 | `opencode run "用一句话解释 JavaScript 闭包"` |
| 配置各 AI 提供商的 API 密钥 | `opencode auth login`（凭据位于 `~/.local/share/opencode/auth.json`） |
| 查看已登录提供商 | `opencode auth list`（简写 `opencode auth ls`） |
| 列出可用模型 | `opencode models`；按提供商过滤如 `opencode models anthropic`；刷新缓存 `opencode models --refresh` |
| Web 界面 | `opencode web` |
| 仅 HTTP API（无 TUI） | `opencode serve` |
| 会话列表 / 用量统计 | `opencode session list`、`opencode stats` |

更多子命令（`agent`、`mcp`、`github` 等）见 [CLI 文档](https://opencode.ai/docs/cli/)。

---

## 三、更新

### 推荐（已安装 CLI）

```bash
opencode upgrade
```

指定版本：

```bash
opencode upgrade v0.1.48
```

若需与安装方式一致，可使用 `opencode upgrade` 的 `--method` 参数：`curl`、`npm`、`pnpm`、`bun`、`brew` 等。

### 通过 npm 安装的补充

```bash
npm update -g opencode-ai
```

### 桌面端

到 [Download](https://opencode.ai/download) 下载并安装新版本。

---

## 四、速查

- **Linux**：`curl … | bash` 或 `npm i -g opencode-ai` / `paru -S opencode` 等。
- **Windows**：**npm/Bun** 或 **Desktop**；类 Unix 环境可用 **WSL** 配合 Linux 方式。
- **使用**：`opencode` 进入 TUI；`opencode run "…"` 快速提问；`opencode auth login` 配置密钥。
- **更新**：优先 `opencode upgrade`；npm 安装可再配合 `npm update -g opencode-ai`。
