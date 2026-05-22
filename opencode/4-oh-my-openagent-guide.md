# Oh My OpenAgent 使用指南

> 原名 `oh-my-opencode`，现更名为 `oh-my-openagent`。本文档基于你的本地配置编写。

## 一、概述

Oh My OpenAgent（简称 OmO）是一个 OpenCode 插件，将单个 AI 编程助手升级为**多 Agent 开发团队**。

**核心功能：**
- 将 OpenCode 变为由 10+ 个专业 Agent 组成的开发团队
- 支持多 Agent 并行工作
- 智能调度：根据任务类型自动分配合适的 Agent

## 二、安装

### 自动安装

```bash
# 使用 bun（推荐）
bunx oh-my-openagent install

# 使用 npm
npx oh-my-openagent install

# 无交互模式安装
npx oh-my-openagent install --no-tui --claude=no --openai=no --gemini=no --copilot=no
```

### 订阅选项

| 选项 | 说明 |
|------|------|
| `--claude` | Claude Pro/Max 订阅 (yes/no/max20) |
| `--openai` | OpenAI/ChatGPT Plus 订阅 |
| `--gemini` | Google Gemini 模型 |
| `--copilot` | GitHub Copilot 订阅 |
| `--opencode-zen` | OpenCode Zen 订阅 |
| `--opencode-go` | OpenCode Go 订阅 |
| `--kimi-for-coding` | Kimi for Coding 订阅 |

### 验证安装

```bash
npx oh-my-openagent doctor
```

## 三、文件结构

安装后生成以下文件：

| 文件路径 | 说明 |
|----------|------|
| `~/.config/opencode/opencode.json` | 主配置，plugins 数组中添加了 `oh-my-openagent@latest` |
| `~/.config/opencode/oh-my-openagent.json` | 插件配置，定义各 Agent 使用的模型 |

## 四、核心概念

### 1. Agent 角色

| Agent | 角色 | 说明 |
|-------|------|------|
| **Sisyphus** | 主工作 Agent | 主要编码任务执行者 |
| **Prometheus** | 战略规划者 | 任务规划和架构设计 |
| **Atlas** | Todo 编排者 | 管理任务列表和进度 |
| **Hephaestus** | 深度自主工作 | 高级代码生成（GPT native） |
| **Oracle** | 架构/调试 | 架构设计和问题诊断 |
| **Metis** | 计划审查 | 审核 Prometheus 的计划 |
| **Momus** | 高精度审查 | 代码审查和质量验证 |
| **Explore** | 快速搜索 | 代码库快速检索 |
| **Librarian** | 文档搜索 | 文档和代码搜索 |
| **Multimodal Looker** | 视觉/截图 | 图片和截图分析 |

### 2. 任务分类 (Categories)

| Category | 说明 |
|----------|------|
| `visual-engineering` | 视觉/前端任务 |
| `ultrabrain` | 深度思考任务 |
| `deep` | 深度分析任务 |
| `artistry` | 创造性任务 |
| `quick` | 快速简单任务 |
| `unspecified-low` | 低优先级未分类 |
| `unspecified-high` | 高优先级未分类 |
| `writing` | 写作任务 |

### 3. 模型家族

**Claude-like 模型：**
- Claude Opus 4.7 / Sonnet 4.6 / Haiku 4.5
- Kimi K2.6 / K2.5
- GLM-5 / GLM-5.1

**GPT 模型：**
- GPT-5.5 / GPT-5.4 / GPT-5.3-codex
- GPT-5-Nano

**快速模型：**
- Grok Code Fast 1
- MiniMax M2.7 / M2.7-highspeed
- Claude Haiku 4.5

## 五、使用方法

### 1. 基础使用 - ultrawork 模式

在 OpenCode 对话中输入：

```
ultrawork
```

或简写：

```
ulw
```

这会启用完整的多 Agent 并行工作模式。

### 2. 精确模式 - Tab 切换

1. 按 **Tab** 键切换到 Prometheus（规划者）模式
2. 通过访谈流程创建工作计划
3. 输入 `/start-work` 开始执行

### 3. 自定义 Agent 调用

在配置文件中可以指定特定 Agent 使用的模型：

```json
{
  "agents": {
    "sisyphus": {
      "model": "opencode/minimax-m2.7-highspeed"
    }
  }
}
```

## 六、配置说明

### 你的当前配置

```json
{
  "agents": {
    "sisyphus": { "model": "opencode/minimax-m2.7-highspeed" },
    "hephaestus": { "model": "opencode/minimax-m2.7-highspeed" },
    "oracle": { "model": "opencode/minimax-m2.7-highspeed" },
    "librarian": { "model": "opencode/minimax-m2.7-highspeed" },
    "explore": { "model": "opencode/minimax-m2.7-highspeed" },
    "multimodal-looker": { "model": "opencode/minimax-m2.7-highspeed" },
    "prometheus": { "model": "opencode/minimax-m2.7-highspeed" },
    "metis": { "model": "opencode/minimax-m2.7-highspeed" },
    "momus": { "model": "opencode/minimax-m2.7-highspeed" },
    "atlas": { "model": "opencode/minimax-m2.7-highspeed" },
    "sisyphus-junior": { "model": "opencode/minimax-m2.7-highspeed" }
  }
}
```

### 可用的模型格式

| Provider | 格式 | 说明 |
|----------|------|------|
| OpenCode 内置 | `opencode/minimax-m2.7-highspeed` | 你的当前配置 |
| OpenAI | `openai/gpt-5.5` | 需要 OpenAI API |
| Anthropic | `anthropic/claude-opus-4-7` | 需要 Claude 订阅 |
| GitHub Copilot | `github-copilot/claude-opus-4.7` | 需要 Copilot |

## 七、常用命令

| 命令 | 说明 |
|------|------|
| `ultrawork` / `ulw` | 启用多 Agent 工作模式 |
| `/start-work` | 开始执行 Prometheus 制定的计划 |
| `npx oh-my-openagent doctor` | 诊断安装状态 |
| `npx oh-my-openagent install` | 重新安装/更新插件 |

## 八、推荐工作流

### 1. 快速任务流（ultrawork）

**适用场景：** 简单修改、bug 修复、单一文件改动

```
输入: "修复 src/auth.ts 中的登录验证问题"
```

**流程：**
- Sisyphus 直接执行
- 自动调度 Explore 搜索相关代码
- 完成后自动验证

---

### 2. 精确规划流（Prometheus → Atlas）

**适用场景：** 新功能开发、重构、多文件改动、复杂任务

**Step 1 - 规划阶段：**
```
按 Tab 切换到 Prometheus 模式
输入: "我们需要在项目中添加用户权限管理系统"
```

Prometheus 会像真正的工程师一样：
- 询问具体需求和范围
- 识别模糊点
- 创建详细计划

**Step 2 - 执行阶段：**
```
输入: /start-work
```

Atlas 接管：
- 将任务分配给专门的 subagent
- 并行执行多个子任务
- 验证每个完成的任务
- 跨任务累积学习

---

### 3. 专项咨询流（@ 提及）

**适用场景：** 需要特定专家协助

| 需求 | 命令 |
|------|------|
| 代码库搜索 | `@explore 搜索登录相关的代码` |
| 文档查询 | `@librarian 查找 React Hooks 最佳实践` |
| 架构咨询 | `@oracle 这个微服务设计合理吗` |
| 代码审查 | `@metis 审查这个 PR 的计划` |
| 视觉分析 | `@multimodal-looker 分析这个截图` |

---

### 4. 深度工作流（Hephaestus）

**适用场景：** 复杂的跨文件重构、深度调试、架构级问题

```
@hephaestus 需要重构整个认证模块，从 JWT 迁移到 OAuth2
```

**注意：** Hephaestus 需要 GPT-5.5，当前配置无法使用。

---

## 九、各 Agent 使用场景速查

| 任务类型 | 推荐 Agent | 说明 |
|----------|-----------|------|
| 简单修改/修复 | `ultrawork` / `ulw` | Sisyphus 自动处理 |
| 新功能开发 | Tab → Prometheus → `/start-work` | 完整规划+执行 |
| 代码搜索 | `@explore` | 快速 grep |
| 文档查找 | `@librarian` | 外部文档搜索 |
| 架构决策 | `@oracle` | 咨询模式 |
| Bug 调试 | `@oracle` | 复杂问题诊断 |
| 计划审查 | `@metis` | 审核 Prometheus 计划 |
| 代码审查 | `@momus` | 质量验证 |
| 前端/UI | `visual-engineering` category | 视觉任务 |
| 深度研究 | `deep` category | 复杂逻辑 |

## 十、模型选择建议

### 你的当前配置限制

当前所有 Agent 都使用 `opencode/minimax-m2.7-highspeed`，这是**不理想**的配置：

| Agent | 理想模型 | 问题 |
|--------|----------|------|
| Sisyphus | Claude Opus / Kimi K2.6 | MiniMax 编排能力有限 |
| Hephaestus | GPT-5.5 | **完全不可用** |
| Oracle | GPT-5.5 | MiniMax 深度推理不足 |
| Explore | Grok Code Fast / Haiku | 浪费高端模型 |

### 理想订阅组合

| 订阅 | 费用 | 覆盖 |
|------|------|------|
| OpenCode Go | $10/月 | Kimi K2.6, GLM-5, MiniMax |
| OpenAI Plus | $20/月 | GPT-5.5, Hephaestus 可用 |

### 临时解决方案

对于简单任务，当前配置仍可使用。但对于复杂任务，效果会打折扣。

## 十一、注意事项

1. **Sisyphus Agent 最佳效果**：推荐使用 Claude Opus 4.5+ 或 Kimi K2.6，其他模型可能降低编排质量
2. **快速任务不要浪费**：Explore 和 Librarian 使用快速模型，不要升级到 Opus 级别，浪费 tokens
3. **Hephaestus 需要 GPT**：当前配置下 Hephaestus 无法正常工作
4. **MiniMax 适合工具类任务**：Explore、Librarian 等 Utility 任务用 MiniMax 是合理的

## 十二、卸载

如需卸载插件：

1. 从 `~/.config/opencode/opencode.json` 中移除 `"oh-my-openagent@latest"`
2. 删除 `~/.config/opencode/oh-my-openagent.json`

---

**官方文档：**
- GitHub: https://github.com/code-yeongyu/oh-my-openagent
- 官网: https://ohmyopenagent.com/docs
