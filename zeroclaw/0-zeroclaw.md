# ZeroClaw 上手：安装与连接

ZeroClaw 是跑在你自己设备上的个人 AI 助手，可通过 Telegram、Slack、Discord 等频道与你交互，并提供网关与网页控制台。**唯一可信源码与发布以官方仓库为准：** [github.com/zeroclaw-labs/zeroclaw](https://github.com/zeroclaw-labs/zeroclaw)。请勿信任非该组织仓库或可疑域名提供的二进制与教程。

---

## 一、环境要求

- **运行时**：Rust stable（从源码或 `cargo install` 时需要）；Homebrew 安装一般为预编译二进制，无需自行装 Rust。
- **资源**：二进制约数 MB；**编译**时建议约 2GB 内存、约 6GB 磁盘。
- **平台**：Linux / macOS 为一级支持；**Windows** 推荐 **WSL2** 按 Linux 流程使用；原生 Windows 也可构建，部分功能受限。

---

## 二、安装方式（任选其一）

### 1. Homebrew（macOS / Linuxbrew，推荐）

```bash
brew install zeroclaw
zeroclaw --version
```

升级：`brew upgrade zeroclaw`。

### 2. 克隆仓库 + 安装脚本（含引导）

仓库内文档提供一键安装路径；克隆后进入目录执行安装脚本（具体脚本名以仓库当前版本为准，常见为 `install.sh` 或统一的 `bootstrap.sh`）：

```bash
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
./install.sh
```

部分版本使用 `./bootstrap.sh`，可带参数（如 `--install-rust`、`--prefer-prebuilt` 等），详见仓库内说明。

### 3. 从源码构建并安装（开发或自定义）

```bash
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
cargo build --release --locked
cargo install --path . --force --locked
```

不全局安装时可在仓库内：`cargo run --release -- <子命令>`。

### 4. 仅已有 Rust 时：crates.io

```bash
cargo install zeroclaw
```

可按需加 `--features`（如硬件、某频道、浏览器自动化等），详见官方安装文档。

### 5. Debian / Ubuntu（APT + 源码安装）

Debian 及其衍生版无官方 `.deb` 单包时，按下面装依赖、Rust，再克隆仓库用脚本或 Cargo 构建（与官方 Installation 文档中 Debian/Ubuntu 一致）。

**1）系统依赖**

```bash
sudo apt-get update
sudo apt-get install -y build-essential pkg-config libssl-dev git curl
```

**2）Rust（若尚未安装）**

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```

**3）安装 ZeroClaw（任选）**

- 推荐：克隆后用仓库脚本（脚本名以当前仓库为准，常见为 `./bootstrap.sh` 或 `./install.sh`）：

```bash
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
./bootstrap.sh
```

- 机器内存/磁盘紧张时，可优先尝试预编译（若脚本支持）：`./bootstrap.sh --prefer-prebuilt`。

- 已装好 Rust 也可在克隆后直接：`cargo build --release --locked` 与 `cargo install --path . --force --locked`，或全局 `cargo install zeroclaw`。

**可选功能依赖（按需）**

```bash
# USB / 设备等（配合对应 Cargo feature）
sudo apt-get install -y libudev-dev

# 浏览器自动化（包名因发行版而异；Debian 常见 chromium + chromium-driver）
sudo apt-get install -y chromium chromium-driver
```

若已安装 **Linuxbrew**，也可在 Debian/Ubuntu 上直接使用上文「Homebrew」方式，避免本机编译。

### 6. Windows 建议路径

- **优先**：安装 WSL2（如 Ubuntu），在 WSL 内按 **Linux / Homebrew 或 bootstrap** 安装。
- **原生 Windows**：安装 Visual Studio C++ 构建工具与 [rustup](https://rustup.rs/)，克隆仓库后 `cargo build --release` 与 `cargo install --path . --force`。

### 7. Docker（可选）

仓库支持通过 `./bootstrap.sh --docker` 等在容器内构建与运行；数据可挂载到卷（如 `~/.zeroclaw`），便于隔离部署。具体端口与环境变量以官方文档为准。

---

## 三、连接与配置（核心流程）

### 1. 引导配置（首次必做）

官方推荐用交互向导完成网关、工作区、频道与模型提供者的配置：

```bash
zeroclaw onboard
```

也可在安装脚本阶段传入 API 等参数（示例，以当前 CLI 为准）：

```bash
./install.sh --api-key "你的密钥" --provider openrouter
```

提供者支持包括 OpenAI、Anthropic、Gemini、OpenRouter 等；订阅类认证（如部分 OAuth）在向导或文档中说明。

### 2. 启动「连接面」：网关（Gateway）

网关提供 Webhook 与网页控制台等控制平面，默认监听示例（以版本为准）：

```bash
zeroclaw gateway
# 常见默认：127.0.0.1:42617；随机端口可加强本地安全：
zeroclaw gateway --port 0
```

需在对应聊天平台（如 Telegram Bot）里把 Webhook 或长轮询地址配到该网关可达的 URL（若公网访问，需正确暴露端口或反代）。

### 3. 完整常驻：守护进程

需要网关 + 频道 + 定时任务等一并运行时：

```bash
zeroclaw daemon
```

### 4. 命令行直接与 Agent 对话（不依赖频道）

```bash
zeroclaw agent -m "Hello, ZeroClaw!"
zeroclaw agent   # 交互模式
```

### 5. 频道与健康检查

- 频道列表、检查等：`zeroclaw channel list`、`zeroclaw channel doctor`（子命令以 `zeroclaw --help` 为准）。
- 全局诊断：`zeroclaw doctor`
- 状态：`zeroclaw status`

### 6. 私信安全与「配对」

默认对未知私聊发送者有配对机制：需你执行批准命令后才会处理其消息，例如：

```bash
zeroclaw pairing approve <配对相关信息>
```

公共入站 DM 需在配置中显式允许；务必阅读官方 `SECURITY.md` 与 `zeroclaw doctor` 提示。

---

## 四、安装后自检

```bash
zeroclaw --version
zeroclaw doctor
```

升级软件包后若行为异常，可再次运行 `zeroclaw doctor`。

---

## 五、从 OpenClaw 迁移（可选）

```bash
zeroclaw migrate openclaw --dry-run   # 仅预览
zeroclaw migrate openclaw             # 执行迁移
```

数据通常从 `~/.openclaw/` 迁到 `~/.zeroclaw/`。

---

## 六、开机自启与后台常驻

ZeroClaw 的“完整常驻”入口是：

```bash
zeroclaw daemon
```

要做到开机自启且稳定后台运行，建议使用系统服务管理器，而不是普通终端常驻。

### 1. Linux / WSL2（推荐）：`systemd --user`

> 适用于 Linux 主机或已启用 systemd 的 WSL2。

1）先确认可前台正常运行：

```bash
zeroclaw daemon
```

2）创建用户级服务文件：

```bash
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/zeroclaw.service <<'EOF'
[Unit]
Description=ZeroClaw Daemon
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/zeroclaw daemon
Restart=always
RestartSec=5
WorkingDirectory=%h
Environment=HOME=%h

[Install]
WantedBy=default.target
EOF
```

3）重载并启用开机自启：

```bash
systemctl --user daemon-reload
systemctl --user enable --now zeroclaw
```

4）如需“未登录仍运行”（服务器/长期运行场景）：

```bash
loginctl enable-linger "$USER"
```

5）运维检查：

```bash
systemctl --user status zeroclaw
journalctl --user -u zeroclaw -f
```

### 2. Windows 原生：注册为 Windows 服务

Windows 原生建议把 `zeroclaw daemon` 注册为系统服务（如使用 NSSM 或 WinSW），并设置：

- Startup type：`Automatic`
- 服务失败恢复：`Restart the Service`
- 工作目录与配置目录指向当前用户可读写路径（如 `%USERPROFILE%\.zeroclaw`）

不建议仅放到“启动文件夹”，稳定性与故障恢复都弱于服务管理器。

### 3. 配置与路径注意事项

- `ExecStart`/服务启动命令请使用 `zeroclaw` 的绝对路径（可通过 `which zeroclaw` / `where zeroclaw` 查询）。
- 若你通过 `zeroclaw onboard` 生成了配置，确认服务运行用户与该配置目录一致。
- 修改端口、网关绑定或渠道配置后，建议重启服务再验证。

### 4. 快速排障清单

```bash
zeroclaw --version
zeroclaw doctor
zeroclaw status
zeroclaw channel doctor
```

若服务反复重启，优先看日志，并先回到前台执行 `zeroclaw daemon` 定位具体报错。

---

## 七、参考链接

- 官方仓库：[https://github.com/zeroclaw-labs/zeroclaw](https://github.com/zeroclaw-labs/zeroclaw)
- 中文 README（仓库内）：`docs/i18n/zh-CN/README.md`
- 安装与平台说明：官方文档站点中的 Installation / 入门指南（以仓库 README 导航为准）
- 服务管理参考：`systemd.service(5)`、Windows Service 官方文档

文档整理日期：2026-04-14（内容随上游版本变化，请以官方仓库为准）。
