# ZeroClaw 插件：安装、使用与原生 Rust 开发

> 参考基线：`zeroclaw` 官方 README、`docs/reference/cli/commands-reference.md`、`CONTRIBUTING.md`（截至 2026-04 可见内容）。  
> 说明：ZeroClaw 迭代快，命令和目录可能变化；动手前请先执行 `zeroclaw --help`、`zeroclaw skills --help`、`zeroclaw config schema` 以本机版本为准。

---

## 一、先说结论：ZeroClaw 的“插件”有 3 种

在 ZeroClaw 语境里，大家常把“插件”混着叫。实际建议区分为：

1. **Skills（最成熟、最易安装）**
   - 通过 `zeroclaw skills ...` 管理
   - 适合快速扩展行为与提示策略
2. **MCP 服务器扩展**
   - 通过配置接入外部工具生态
   - 适合对接三方系统/内网服务
3. **原生 Rust 扩展（源码级）**
   - 在 ZeroClaw 仓库中实现 trait 并注册
   - 适合深度定制（新 Tool/Provider/Channel/Memory 等）

如果你问“能不能安装插件”，最直接答案是：**能，优先用 Skills**。  
如果你问“怎么开发原生 Rust 插件”，本质是：**按 trait 扩展 + 工厂注册 + 编译发布**。

---

## 二、如何安装插件（Skills 路线）

官方命令组是 `skills`：

```bash
zeroclaw skills list
zeroclaw skills audit <git-url-or-local-path>
zeroclaw skills install <git-url-or-local-path>
zeroclaw skills remove <skill-name>
```

### 1. 推荐安装流程

1）先审计：

```bash
zeroclaw skills audit https://github.com/<you>/<skill-repo>.git
```

2）再安装：

```bash
zeroclaw skills install https://github.com/<you>/<skill-repo>.git
```

3）检查是否生效：

```bash
zeroclaw skills list
zeroclaw status
```

### 2. 安全机制（官方文档已明确）

`skills install` 内置静态安全审计，默认会拦截高风险内容，例如：

- symlink
- 脚本文件（`.sh/.ps1/.bat/.cmd` 等）
- 高风险 shell 片段
- 越界或远程 markdown 链接

这也是为什么建议先 `skills audit`，再 `skills install`。

---

## 三、如何开发“可安装插件”（Skill）

如果你追求“像插件一样安装给别人用”，建议先从 Skill 做：

1. 新建仓库（或本地目录），放 `SKILL.md` / `SKILL.toml`
2. 写清触发条件、执行步骤、约束边界
3. 本地 `skills audit` + `skills install` 验证
4. 提供 git 地址给他人安装

这条路线不需要改 ZeroClaw 内核，迭代最快。

---

## 四、MCP 扩展（插件化接入外部工具）

MCP 适合接入“外部工具能力”，比如内部 API、SaaS、自动化平台。

建议流程：

1. 先保证 MCP Server 本身可用（stdio/http）
2. 在 `config.toml` 配置对应服务器（字段以 `config schema` 为准）
3. `zeroclaw doctor` 检查
4. 在对话中验证工具是否可调用

MCP 更像“工具层插件”，Skill 更像“行为层插件”。

---

## 五、重点：如何开发原生 Rust 插件（源码级）

这一部分是“真正的内核扩展”。  
官方 `CONTRIBUTING.md` 给出的核心思想是：

- ZeroClaw 是 **trait-based pluggability**
- 扩展方式是：**实现 trait -> 注册到工厂**

### 1. 原生 Rust 扩展适用场景

适合以下需求：

- 新增一个内置工具（Tool）
- 新增模型提供商（Provider）
- 新增消息渠道（Channel）
- 新增记忆后端（Memory）
- 新增观测后端（Observer）

如果只是提示词策略或流程编排，不建议上来就做原生 Rust 扩展。

### 2. 开发前准备

```bash
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
cargo build --release --locked
```

调试命令（无需全局安装）：

```bash
cargo run --release -- status
cargo run --release -- doctor
```

### 3. 架构定位（按 trait 选扩展点）

官方文档给出的目录语义：

- `src/providers/` -> `Provider trait`
- `src/channels/` -> `Channel trait`
- `src/tools/` -> `Tool trait`
- `src/memory/` -> `Memory trait`
- `src/observability/` -> `Observer trait`

建议先明确你要扩展哪一层，再落代码，避免跨层耦合。

### 4. 示例：新增一个原生 Tool（最常见）

#### 第一步：创建工具实现文件

新建 `src/tools/your_tool.rs`（命名用 snake_case）：

```rust
use async_trait::async_trait;
use anyhow::Result;
use serde_json::{json, Value};
use crate::tools::traits::{Tool, ToolResult};

pub struct YourTool;

#[async_trait]
impl Tool for YourTool {
    fn name(&self) -> &str { "your_tool" }

    fn description(&self) -> &str { "Do something useful safely" }

    fn parameters_schema(&self) -> Value {
        json!({
            "type": "object",
            "properties": {
                "input": { "type": "string", "description": "Input text" }
            },
            "required": ["input"]
        })
    }

    async fn execute(&self, args: Value) -> Result<ToolResult> {
        let input = args["input"]
            .as_str()
            .ok_or_else(|| anyhow::anyhow!("Missing 'input'"))?;

        Ok(ToolResult {
            success: true,
            output: format!("Processed: {input}"),
            error: None,
        })
    }
}
```

#### 第二步：注册到工厂

在 `src/tools/mod.rs`（或项目当前的工具注册位置）中：

1. `mod your_tool;`
2. 在工具构建分支里加入 `"your_tool"` 的映射

伪代码示意：

```rust
"your_tool" => Ok(Box::new(your_tool::YourTool)),
```

> 关键点：**只实现 trait 还不够，必须注册工厂映射，否则不会被系统发现。**

#### 第三步：补配置（可选）

如果你的工具需要开关、token、endpoint：

- 在配置结构体里新增字段
- 需要保密的字段用 `#[secret]`
- 需要 CLI 可配置时，确保能被 `config` 子系统识别

然后验证：

```bash
zeroclaw config list --filter <your-prefix>
zeroclaw config schema
```

#### 第四步：写测试

至少覆盖：

- 参数缺失/类型错误
- 正常输入输出
- 风险边界（路径、域名、命令等）
- 配置缺失时的报错行为

建议同时跑：

```bash
cargo test --locked
./scripts/ci/rust_quality_gate.sh
```

### 5. 新增 Provider / Channel 的套路（同理）

官方给了标准骨架：

- Provider：实现 `Provider trait`，注册到 `src/providers/mod.rs`
- Channel：实现 `Channel trait`，注册到 channel factory

共同模式都是：

1. 定义结构体 + `new()`
2. 实现 trait 的核心方法
3. 工厂注册
4. 配置接线
5. 测试与文档

### 6. 配置字段与 Secret 字段（非常重要）

如果你加了敏感字段（token/password/api key）：

- 使用 `#[secret]` 标注
- 走 `config` 命令读写，而不是硬编码
- 默认值与兼容性要考虑旧配置加载

避免把密钥写在代码、示例、日志和测试快照里。

### 7. 发布与部署

原生 Rust 扩展完成后，一般有两种交付方式：

1. **提交上游仓库**（PR 合并后由上游发布）
2. **自维护分支/私有发行版**

本地安装构建产物：

```bash
cargo install --path . --force --locked
zeroclaw --version
zeroclaw doctor
```

若已做服务化运行，重启服务：

```bash
zeroclaw service restart
zeroclaw service status
```

---

## 六、原生 Rust 插件开发的最佳实践

1. **先 Skill/MCP，后 Rust 内核**
   - 能用配置和组合解决的，不先改内核
2. **trait 边界清晰**
   - Provider 不直接侵入 Channel 内部逻辑
3. **默认安全**
   - 最小权限、allowlist、显式开关
4. **错误可诊断**
   - 错误信息包含上下文，不暴露敏感值
5. **文档与回滚**
   - 改配置结构时写迁移说明和回滚路径

---

## 七、常见问题（FAQ）

### Q1：我执行了代码但工具不出现？

大概率是没在 factory 注册，或 feature/config 没启用。

### Q2：为什么我写了新字段但 `config` 看不到？

检查配置结构 derive、prefix/nested、schema 导出链路是否完整。

### Q3：原生 Rust 插件和 Skill 哪个更“正确”？

没有绝对。  
经验是：**业务策略/流程 -> Skill；系统能力扩展 -> Rust。**

### Q4：有没有 `zeroclaw plugins install`？

以当前官方命令参考为准，公开稳定命令是 `zeroclaw skills ...`。  
插件生态能力如有新增，请以 `zeroclaw --help` 和 release note 为准。

---

## 八、最小实践路径（建议照着做）

1. 先写一个 Skill 并成功 `skills install`
2. 再写一个最小 `Tool trait` 原生扩展
3. 完成：注册、配置、测试、`doctor` 验证
4. 最后再考虑 Provider/Channel 级别扩展

这样学习曲线最平滑，也最不容易把系统改坏。

---

文档整理日期：2026-04-16。
