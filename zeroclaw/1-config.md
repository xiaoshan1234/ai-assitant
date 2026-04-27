# ZeroClaw 配置文件详解（`config.toml`）

> 参考：官方配置参考（`docs/reference/api/config-reference.md`）、`zeroclaw config` 命令文档  
> 说明：ZeroClaw 迭代较快，字段可能随版本变化；本文按 2026-02 左右官方文档整理，使用前建议执行 `zeroclaw config schema` 与 `zeroclaw config show` 核对本机版本。

---

## 一、配置文件位置与组成

首次完成引导（`zeroclaw onboard`）后，常见目录结构：

```text
~/.zeroclaw/
├── config.toml             # 主配置文件
├── .secret_key             # 本地密钥（用于敏感字段加密，严禁提交）
├── active_workspace.toml   # 当前工作区标记（可选）
├── workspace/              # 工作区目录
└── memory/                 # 记忆数据（与 backend 有关）
```

关键点：

- 主配置文件：`~/.zeroclaw/config.toml`
- 建议把 `.secret_key` 加入忽略，不要进 Git
- 某些字段可被环境变量覆盖，不写进 `config.toml` 也会生效

---

## 二、加载与优先级（很重要）

官方文档里有两种表述（命令文档与配置参考），实际可以按下面理解：

1. **命令行参数（CLI flags）**：最高优先级  
2. **环境变量（`ZEROCLAW_*` / provider 变量）**  
3. **`config.toml`**  
4. **内置默认值**

此外，配置文件路径解析常见顺序为：

1. `ZEROCLAW_WORKSPACE`（或指定 config-dir 的场景）  
2. `~/.zeroclaw/active_workspace.toml` 标记  
3. 默认 `~/.zeroclaw/config.toml`

建议用这两个命令确认“最终生效值”：

```bash
zeroclaw config show
zeroclaw config get gateway.port
```

---

## 三、`zeroclaw config` 命令（建议先会用）

```bash
zeroclaw config show               # 查看完整生效配置（敏感字段会脱敏）
zeroclaw config get <dot.path>     # 读取单个键
zeroclaw config set <dot.path> <value>   # 修改并持久化
zeroclaw config schema             # 打印 JSON Schema（最权威）
```

示例：

```bash
zeroclaw config get default_provider
zeroclaw config set gateway.port 9090
zeroclaw config set autonomy.allowed_commands '["git","ls","cat"]'
```

`config set` 会自动做类型推断：

- `true/false` -> 布尔
- 数字 -> 整数/浮点
- `[...]` / `{...}` -> 数组/对象
- 其他 -> 字符串

---

## 四、核心顶层键（最常改）

| 键 | 默认值（常见） | 作用 |
|---|---|---|
| `default_provider` | `openrouter` | 默认模型提供商 |
| `default_model` | `anthropic/claude-sonnet-4-6`（文档可能写 `4.6`） | 默认模型 |
| `default_temperature` | `0.7` | 采样温度 |
| `api_key` / `api_url` | 无 | 提供商凭据与接口地址（如本地 Ollama） |

说明：

- 最终版本以 `zeroclaw config schema` 和 `config show` 为准（文档字符串有时会出现 `4-6` / `4.6` 差异）
- 生产环境建议优先用环境变量注入密钥（如 `ANTHROPIC_API_KEY`）

---

## 五、重点配置区块（按使用频率）

### 1) `[gateway]`（网关服务）

常用键：

- `host`（默认 `127.0.0.1`）
- `port`（默认 `42617`）
- `require_pairing`（默认 `true`）
- `allow_public_bind`（默认 `false`）
- `path_prefix`（反向代理子路径部署）

建议：

- 本机单用户：`host = "127.0.0.1"` 保持默认
- 对公网暴露前必须配置反代、认证、限流，并保留 `allow_public_bind = false` 直到确认安全策略

---

### 2) `[autonomy]`（自治与执行权限）

常用键：

- `level`：`read_only` / `supervised` / `full`
- `workspace_only`：是否限制在工作区
- `allowed_commands`：shell 命令白名单（支持 `"*"` 但不建议）
- `forbidden_paths` / `allowed_roots`
- 风险控制：`require_approval_for_medium_risk`、`block_high_risk_commands`

这是“安全边界”的核心区块，建议优先配置。

---

### 3) `[agent]`（代理行为）

常用键：

- `max_tool_iterations`（默认 `10`）
- `max_history_messages`（默认 `50`）
- `compact_context`（小模型建议开）
- `parallel_tools`（并行工具调用）
- `tool_filter_groups`（按关键词动态收缩 MCP 工具 schema，节省 token）

如果出现“循环调用工具”或“响应太慢”，先看这一节。

---

### 4) `[memory]`（记忆后端）

常用键：

- `backend`：`sqlite` / `lucid` / `markdown` / `none`
- `auto_save`
- 嵌入相关：`embedding_provider`、`embedding_model`、`embedding_dimensions`

经验：

- 单机默认用 `sqlite` 即可
- 追求语义检索可开启 embedding + 向量后端

---

### 5) `[channels_config]`（多渠道接入）

顶层常用：

- `message_timeout_secs`（默认约 `300`，慢模型更需要）

子表按渠道写，例如：

- `[channels_config.telegram]`
- `[channels_config.discord]`
- `[channels_config.whatsapp]`
- `[channels_config.nostr]`
- `[channels_config.linq]`
- `[channels_config.nextcloud_talk]`

注意：

- 多数渠道的 allowlist 默认是“空即拒绝”
- 先小范围白名单测试，再逐步放开

---

### 6) `[security.otp]` 与 `[security.estop]`（安全强相关）

`[security.otp]`：

- `enabled`
- `method`（如 `totp`）
- `gated_actions` / `gated_domains` / `gated_domain_categories`

`[security.estop]`：

- `enabled`
- `state_file`
- `require_otp_to_resume`

适合高风险动作场景（shell、浏览器、敏感域名）。

---

### 7) `[reliability]`（稳定性与回退）

常用键：

- `fallback_providers`
- `model_fallbacks`
- `api_keys`（429 轮换）
- `provider_retries` / `provider_backoff_ms`

当主模型抽风、限流或超时时，这节能显著提升可用性。

---

### 8) 其他常见区块

- `[observability]`：日志、OTEL、运行时 trace
- `[skills]`：open-skills 开关、prompt 注入模式
- `[browser]` / `[http_request]` / `[google_workspace]`：工具级开关与域名限制
- `[runtime]`：如 `reasoning_enabled`
- `[[model_routes]]` / `[[embedding_routes]]`：hint 路由（便于平滑升级模型）
- `[query_classification]`：按消息内容自动选路由
- `[hardware]` / `[peripherals]`：硬件与外设

---

## 六、一个实用起步模板（偏安全）

```toml
default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4-6"
default_temperature = 0.7

[gateway]
host = "127.0.0.1"
port = 42617
require_pairing = true
allow_public_bind = false

[autonomy]
level = "supervised"
workspace_only = true
allowed_commands = ["git", "ls", "cat", "pwd"]
block_high_risk_commands = true
require_approval_for_medium_risk = true

[agent]
max_tool_iterations = 10
max_history_messages = 50
compact_context = true

[memory]
backend = "sqlite"
auto_save = true

[channels_config]
message_timeout_secs = 300

[security.otp]
enabled = false

[reliability]
provider_retries = 2
provider_backoff_ms = 500
```

---

## 七、配置修改后的验证流程

每次改完建议跑：

```bash
zeroclaw config show > /dev/null
zeroclaw doctor
zeroclaw channel doctor
zeroclaw status
```

如果你在跑 `zeroclaw channel start` / daemon：

- 部分键支持热更新（如默认 provider/model、reliability 等）
- 但涉及绑定地址、端口、部分工具初始化的改动，通常重启更稳

---

## 八、常见问题与排查

1. **改了 `config.toml` 但不生效**  
   先查是否被环境变量覆盖：`ZEROCLAW_*`、`OPENAI_API_KEY`、`ANTHROPIC_API_KEY` 等。

2. **`config set` 报类型错误**  
   你传入的是字符串但目标是数值/布尔/数组，按 JSON 形式传值。

3. **频道不回消息**  
   优先查 allowlist 是否为空（很多渠道默认拒绝全部）。

4. **HTTP/浏览器工具无法访问**  
   看 `allowed_domains` 是否放行；`http_request` 默认 deny-by-default。

5. **网关外网不可达或风险过高**  
   检查 `host`、`allow_public_bind`、反代配置和 pairing/OTP 策略。

---

## 九、建议的运维习惯

- 把密钥放环境变量，不把明文 key 提交到仓库
- 每次升级 ZeroClaw 后执行：`zeroclaw config schema` + `zeroclaw doctor`
- 用 `zeroclaw config get` 做自动化巡检（如 CI 或开机自检脚本）
- 线上环境固定一份“最小权限”基线配置，再按场景局部放开

---

## 十、参考链接

- 官方仓库：<https://github.com/zeroclaw-labs/zeroclaw>
- 配置参考（仓库）：`docs/reference/api/config-reference.md`
- 命令文档：`zeroclaw config`（show/get/set/schema）

文档整理日期：2026-04-16。
