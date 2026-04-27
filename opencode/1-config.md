# OpenCode 配置文件详解

> 主文档：[Config](https://opencode.ai/docs/config/)  
> 运行时配置 Schema：[opencode.ai/config.json](https://opencode.ai/config.json)  
> TUI Schema：[opencode.ai/tui.json](https://opencode.ai/tui.json)

OpenCode 通过 **JSON** 或 **JSONC（带注释的 JSON）** 文件进行配置。建议在文件顶部声明 `$schema`，以便编辑器校验与自动补全。

---

## 一、基本格式

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4-5",
  "autoupdate": true,
  "server": {
    "port": 4096
  }
}
```

- **运行时 / 通用配置**：`$schema` 指向 `https://opencode.ai/config.json`
- **TUI 专用**：使用 `tui.json` / `tui.jsonc`，`$schema` 指向 `https://opencode.ai/tui.json`

---

## 二、配置位置与合并规则

### 2.1 合并方式

- 多个来源的配置会 **合并**，不是整文件覆盖。
- **仅对冲突的键**，后加载的来源覆盖先加载的；**不冲突的键**会保留来自各层的设置。
- 例：全局设 `autoupdate: true`，项目设 `model: "anthropic/..."`，最终配置会 **同时包含** 这两项。

### 2.2 优先级顺序（从早到晚加载；后者在冲突时覆盖前者）

| 顺序 | 来源 | 说明 |
|------|------|------|
| 1 | **Remote**（`.well-known/opencode`） | 组织级默认；在支持该能力的提供商认证后自动拉取 |
| 2 | **全局** `~/.config/opencode/opencode.json` | 用户级偏好 |
| 3 | **自定义路径** 环境变量 `OPENCODE_CONFIG` | 指向单个配置文件 |
| 4 | **项目** 项目根目录 `opencode.json` | 在「普通配置文件」中优先级最高 |
| 5 | **`.opencode` 目录** | agents、commands、plugins 等 |
| 6 | **内联** `OPENCODE_CONFIG_CONTENT` | 运行时覆盖（JSON 内容） |
| 7 | **托管配置文件**（见下文系统路径） | 管理员控制 |
| 8 | **macOS 托管偏好**（MDM `.mobileconfig`） | 最高优先级，用户不可覆盖 |

含义简述：**项目可覆盖全局与远程**；**托管策略覆盖一切**。

### 2.3 全局配置

- 主文件：`~/.config/opencode/opencode.json`
- TUI 专用：`~/.config/opencode/tui.json`

适用于跨项目的提供商、模型、权限等。

### 2.4 项目配置

- 主文件：项目根目录 `opencode.json`
- TUI：同目录 `tui.json`

启动时会在 **当前目录向上查找**，直到最近的 **Git 根目录**。适合提交到仓库，与全局使用 **同一套 schema**。

### 2.5 自定义配置文件路径

```bash
export OPENCODE_CONFIG=/path/to/my/custom-config.json
opencode run "Hello world"
```

加载顺序上位于全局与项目之间（见上文优先级表）。

### 2.6 自定义配置目录

```bash
export OPENCODE_CONFIG_DIR=/path/to/my/config-directory
opencode run "Hello world"
```

该目录会像标准 `.opencode` 一样被搜索（agents、commands、modes、plugins 等），且 **在全局与 `.opencode` 之后** 加载，可覆盖其中内容。

### 2.7 远程组织配置

组织可通过 `.well-known/opencode` 提供默认项（例如默认关闭某些 MCP）。你在本地 `opencode.json` 里把对应项改为 `enabled: true` 即可覆盖，无需复制整份远程配置。

### 2.8 托管（Managed）配置

管理员下发、用户一般 **不能改文件** 的层级：

| 平台 | 路径 |
|------|------|
| macOS | `/Library/Application Support/opencode/` |
| Linux | `/etc/opencode/` |
| Windows | `%ProgramData%\opencode` |

在其中放置 `opencode.json` 或 `opencode.jsonc`。macOS 还可通过 MDM 下发 `ai.opencode.managed` 偏好（详见官方文档中的 `.mobileconfig` 示例）。

验证合并结果可用：

```bash
opencode debug config
```

### 2.9 `.opencode` 与目录命名

`~/.config/opencode` 与项目 `.opencode` 下子目录推荐使用复数：`agents/`、`commands/`、`modes/`、`plugins/`、`skills/`、`tools/`、`themes/`。单数形式为兼容旧版仍支持。

---

## 三、TUI 配置（`tui.json`）

与 `opencode.json` 分离，专门控制终端界面。

```json
{
  "$schema": "https://opencode.ai/tui.json",
  "scroll_speed": 3,
  "scroll_acceleration": {
    "enabled": true
  },
  "diff_style": "auto",
  "mouse": true
}
```

- 环境变量 **`OPENCODE_TUI_CONFIG`** 可指向自定义 TUI 配置文件路径。
- 旧版在 `opencode.json` 里的 `theme`、`keybinds`、`tui` 已 **弃用**，会在可能时自动迁移。

**主题**（在 `tui.json` 中）：

```json
{
  "$schema": "https://opencode.ai/tui.json",
  "theme": "tokyonight"
}
```

**快捷键**：

```json
{
  "$schema": "https://opencode.ai/tui.json",
  "keybinds": {}
}
```

详见：[Themes](https://opencode.ai/docs/themes)、[Keybinds](https://opencode.ai/docs/keybinds)。

---

## 四、服务端（`serve` / `web`）

```json
{
  "$schema": "https://opencode.ai/config.json",
  "server": {
    "port": 4096,
    "hostname": "0.0.0.0",
    "mdns": true,
    "mdnsDomain": "myproject.local",
    "cors": ["http://localhost:5173"]
  }
}
```

| 字段 | 含义 |
|------|------|
| `port` | 监听端口 |
| `hostname` | 监听地址；启用 `mdns` 且未设置时默认可为 `0.0.0.0` |
| `mdns` | 局域网 mDNS 发现 |
| `mdnsDomain` | mDNS 域名，默认 `opencode.local` |
| `cors` | 浏览器客户端跨域时允许的 **完整源**（scheme + 主机 + 可选端口） |

详见：[Server](https://opencode.ai/docs/server)。

---

## 五、模型与提供商

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {},
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "anthropic/claude-haiku-4-5"
}
```

- **`model`**：主模型。
- **`small_model`**：轻量任务（如标题生成）；默认会尽量用更便宜模型，否则回退到主模型。
- **`provider`**：可按提供商设置选项，例如：

```json
{
  "provider": {
    "anthropic": {
      "options": {
        "timeout": 600000,
        "chunkTimeout": 30000,
        "setCacheKey": true
      }
    }
  }
}
```

| 选项 | 含义 |
|------|------|
| `timeout` | 请求超时（毫秒），默认 300000；`false` 表示禁用 |
| `chunkTimeout` | 流式响应块之间无数据则中止 |
| `setCacheKey` | 为指定提供商始终设置 cache key |

**Amazon Bedrock** 额外支持 `region`、`profile`、`endpoint`（VPC Endpoint 等），详见 [Providers - Amazon Bedrock](https://opencode.ai/docs/providers#amazon-bedrock)。

本地模型见：[Models - Local](https://opencode.ai/docs/models#local)。

---

## 六、工具开关（`tools`）

限制 LLM 可调用的工具：

```json
{
  "tools": {
    "write": false,
    "bash": false
  }
}
```

详见：[Tools](https://opencode.ai/docs/tools)。

---

## 七、Agent

### 7.1 在配置中内联定义

```jsonc
{
  "agent": {
    "code-reviewer": {
      "description": "Reviews code for best practices and potential issues",
      "model": "anthropic/claude-sonnet-4-5",
      "prompt": "You are a code reviewer...",
      "tools": {
        "write": false,
        "edit": false
      }
    }
  }
}
```

也可使用 `~/.config/opencode/agents/` 或 `.opencode/agents/` 下的 Markdown 文件。详见：[Agents](https://opencode.ai/docs/agents)。

### 7.2 默认 Agent（`default_agent`）

```json
{
  "default_agent": "plan"
}
```

- 必须是 **主 Agent**（不能是 subagent）。
- 可为内置如 `"build"`、`"plan"`，或自定义 Agent；无效时会回退到 `"build"` 并警告。
- 对 TUI、CLI `opencode run`、桌面、GitHub Action 等均生效。

---

## 八、分享（`share`）

```json
{
  "share": "manual"
}
```

| 值 | 含义 |
|----|------|
| `"manual"` | 默认：需手动用 `/share` 等分享 |
| `"auto"` | 新对话自动分享 |
| `"disabled"` | 完全关闭分享 |

详见：[Share](https://opencode.ai/docs/share)。

---

## 九、自定义命令（`command`）

```jsonc
{
  "command": {
    "test": {
      "template": "Run the full test suite with coverage...",
      "description": "Run tests with coverage",
      "agent": "build",
      "model": "anthropic/claude-haiku-4-5"
    },
    "component": {
      "template": "Create a new React component named $ARGUMENTS...",
      "description": "Create a new component"
    }
  }
}
```

也可使用 `~/.config/opencode/commands/` 或 `.opencode/commands/` 下的 Markdown。详见：[Commands](https://opencode.ai/docs/commands)。

---

## 十、快照（`snapshot`）

默认开启，用于在会话内撤销 Agent 对文件的修改。大仓库可能带来索引慢、磁盘占用高。

```json
{
  "snapshot": false
}
```

关闭后 **无法** 通过界面回滚 Agent 改动。

---

## 十一、自动更新（`autoupdate`）

```json
{
  "autoupdate": false
}
```

或仅通知新版本：

```json
{
  "autoupdate": "notify"
}
```

注意：若通过 **Homebrew 等包管理器** 安装，`notify` 等行为可能不适用（以官方说明为准）。

---

## 十二、格式化（`formatter`）

```json
{
  "formatter": {
    "prettier": {
      "disabled": true
    },
    "custom-prettier": {
      "command": ["npx", "prettier", "--write", "$FILE"],
      "environment": {
        "NODE_ENV": "development"
      },
      "extensions": [".js", ".ts", ".jsx", ".tsx"]
    }
  }
}
```

详见：[Formatters](https://opencode.ai/docs/formatters)。

---

## 十三、权限（`permission`）

默认多数操作无需确认；可改为询问或拒绝：

```json
{
  "permission": {
    "edit": "ask",
    "bash": "ask"
  }
}
```

详见：[Permissions](https://opencode.ai/docs/permissions)。

---

## 十四、上下文压缩（`compaction`）

```json
{
  "compaction": {
    "auto": true,
    "prune": true,
    "reserved": 10000
  }
}
```

| 字段 | 含义 |
|------|------|
| `auto` | 上下文满时自动压缩（默认 true） |
| `prune` | 修剪旧工具输出以省 token（默认 true） |
| `reserved` | 压缩时预留 token，避免溢出 |

---

## 十五、文件监听（`watcher`）

```json
{
  "watcher": {
    "ignore": ["node_modules/**", "dist/**", ".git/**"]
  }
}
```

Glob 语法，用于排除嘈杂目录。

---

## 十六、MCP 服务（`mcp`）

```json
{
  "mcp": {}
}
```

详见：[MCP Servers](https://opencode.ai/docs/mcp-servers)。

---

## 十七、插件（`plugin`）

插件文件可放在 `.opencode/plugins/` 或 `~/.config/opencode/plugins/`，也可通过配置从 npm 加载：

```json
{
  "plugin": ["opencode-helicone-session", "@my-org/custom-plugin"]
}
```

详见：[Plugins](https://opencode.ai/docs/plugins)。

---

## 十八、说明文件 / 规则（`instructions`）

```json
{
  "instructions": ["CONTRIBUTING.md", "docs/guidelines.md", ".cursor/rules/*.md"]
}
```

路径与 glob 会作为给模型的说明来源。详见：[Rules](https://opencode.ai/docs/rules)。

---

## 十九、提供商白名单与黑名单

**禁用部分提供商**（优先级高于 `enabled_providers`）：

```json
{
  "disabled_providers": ["openai", "gemini"]
}
```

效果：不加载对应提供商、环境变量与 `/connect` 也不会启用、模型列表中不显示。

**仅允许部分提供商**：

```json
{
  "enabled_providers": ["anthropic", "openai"]
}
```

若同一提供商同时出现在两个列表中，以 **`disabled_providers` 为准**。

---

## 二十、实验性（`experimental`）

```json
{
  "experimental": {}
}
```

实验项不稳定，可能变更或删除。CLI 文档中还有大量以 `OPENCODE_EXPERIMENTAL_*` 等形式暴露的环境变量，可与官方 release 说明对照使用。

---

## 二十一、配置中的变量替换

### 21.1 环境变量 `{env:NAME}`

```json
{
  "model": "{env:OPENCODE_MODEL}",
  "provider": {
    "anthropic": {
      "models": {},
      "options": {
        "apiKey": "{env:ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

未设置的环境变量会被替换为 **空字符串**。

### 21.2 文件内容 `{file:path}`

```json
{
  "instructions": ["./custom-instructions.md"],
  "provider": {
    "openai": {
      "options": {
        "apiKey": "{file:~/.secrets/openai-key}"
      }
    }
  }
}
```

路径可为相对配置文件目录，或以 `/`、`~` 开头的绝对路径。适合把密钥与大段说明拆到独立文件。

---

## 二十二、调试合并结果

```bash
opencode debug config
```

可查看最终生效的配置（含托管项是否不可覆盖等）。

---

## 二十三、相关链接速查

| 主题 | 链接 |
|------|------|
| 配置总览 | [Config](https://opencode.ai/docs/config/) |
| Schema（运行时） | [config.json](https://opencode.ai/config.json) |
| Schema（TUI） | [tui.json](https://opencode.ai/tui.json) |
| 模型 | [Models](https://opencode.ai/docs/models) |
| 提供商 | [Providers](https://opencode.ai/docs/providers) |
| 服务端 | [Server](https://opencode.ai/docs/server) |
