# Hermes Agent 安装与最简使用教程

> 官方仓库：<https://github.com/NousResearch/hermes-agent>  
> 官方文档：<https://hermes-agent.nousresearch.com/docs/>

---

## 一、先说结论（2 分钟上手）

Hermes Agent 官方推荐一键安装：

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

安装完成后：

```bash
source ~/.bashrc   # 或 source ~/.zshrc
hermes
```

如果你是 Windows 用户：**请在 WSL2 中安装和运行**（原生 Windows 目前不支持）。

---

## 二、安装前准备

### 1) 系统支持

- Linux
- macOS
- WSL2（Windows 推荐路径）
- Android Termux（可用同一安装脚本）

### 2) 最小前置条件

官方要求基本只需要 `git` 可用：

```bash
git --version
```

其余依赖（如 Python/Node/ffmpeg/ripgrep 等）由安装脚本自动处理。

---

## 三、推荐安装方式（自动安装）

执行：

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

然后重载 shell：

```bash
source ~/.bashrc
# 或
source ~/.zshrc
```

验证是否安装成功：

```bash
hermes version
hermes doctor
```

---

## 四、手动安装（开发者可选）

如果你想完全掌控安装过程，用手动方式：

```bash
git clone --recurse-submodules https://github.com/NousResearch/hermes-agent.git
cd hermes-agent

curl -LsSf https://astral.sh/uv/install.sh | sh
uv venv venv --python 3.11
export VIRTUAL_ENV="$(pwd)/venv"
uv pip install -e ".[all]"
```

将 `hermes` 命令加入 PATH（示例）：

```bash
mkdir -p ~/.local/bin
ln -sf "$(pwd)/venv/bin/hermes" ~/.local/bin/hermes
```

---

## 五、首次配置（最关键）

安装完成后，至少做一件事：配置模型提供商。

最简单方式：

```bash
hermes model
```

也可以一次性走完整向导：

```bash
hermes setup
```

如果使用 API Key 提供商（例如 OpenRouter），按提示填写对应 Key 即可。

---

## 六、最简使用流程（新手照抄）

### Step 1：启动对话

```bash
hermes
```

### Step 2：先试一条自然语言请求

例如输入：

```text
帮我查看当前磁盘占用，并列出最大的 5 个目录
```

Hermes 会调用工具执行并返回结果。

### Step 3：会话中常用命令

- `/help`：查看帮助
- `/tools`：查看可用工具
- `/model`：切换模型
- `/save`：保存会话
- `Ctrl+C`：中断当前任务

### Step 4：退出后恢复会话

```bash
hermes --continue
# 或简写
hermes -c
```

---

## 七、常用命令速查

```bash
hermes                 # 进入交互模式
hermes model           # 选择/切换模型提供商
hermes tools           # 配置工具开关
hermes setup           # 一次性完整配置
hermes doctor          # 环境诊断
hermes update          # 升级
hermes gateway         # 启动消息网关（Telegram/Discord 等）
```

---

## 八、可选：接入消息平台（手机聊天）

如果你想在 Telegram/Discord/Slack 等平台和 Hermes 对话：

```bash
hermes gateway setup
hermes gateway start
```

完成平台 token 配置后，就可以在对应 IM 里直接和 Hermes 聊天。

---

## 九、常见问题

### 1) `hermes: command not found`

- 先执行 `source ~/.bashrc` 或 `source ~/.zshrc`
- 再检查 `~/.local/bin` 是否在 PATH

### 2) 模型不可用 / API key 未设置

- 执行 `hermes model` 重新配置
- 或用 `hermes setup` 重跑向导

### 3) Windows 上直接装失败

- 这是预期，官方建议使用 WSL2
- 在 WSL2 里执行安装命令

---

## 十、建议的入门路径

1. 先自动安装  
2. 跑 `hermes model` 完成模型配置  
3. 用 `hermes` 聊天测试  
4. 用 `hermes doctor` 排查环境  
5. 再按需配置 `gateway`、skills、MCP

这样成功率最高，也最省时间。

---

文档整理日期：2026-04-16。
