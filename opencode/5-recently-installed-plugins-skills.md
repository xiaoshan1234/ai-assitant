# 最近安装的 OpenCode Skills 和 Plugins 使用指南

> 安装日期：2026-05-23

---

## 1. oh-my-openagent

**类型**：OpenCode Agent 引擎插件
**仓库**：https://github.com/code-yeongyu/oh-my-openagent
**安装方式**：npm 全局安装

### 作用
为 OpenCode 提供增强的 Agent 能力，包含多个专业 Agent（Sisyphus、Oracle、Librarian 等）。

### 使用方式
无需手动调用，已作为 OpenCode 的 Agent 引擎运行。

---

## 2. using-superpowers

**类型**：Global Skill
**仓库**：obra/using-superpowers
**安装位置**：
- 全局：`~/.agents/skills/using-superpowers/`
- 项目级：`.agents/skills/using-superpowers/`

### 作用
Superpowers 技能集成，提供增强的工作流能力。

### 使用方式
```bash
# 在 opencode 中使用
npx skills use using-superpowers
```

---

## 3. subtask2

**类型**：Plugin
**仓库**：@spoons-and-mirrors/subtask2
**安装方式**：npm 全局安装

### 作用
子任务管理插件，用于分解和管理复杂任务。

### 使用方式
在 opencode 中自动激活，或查看插件文档了解具体用法。

---

## 4. planning-with-files

**类型**：Personal/Project Skill
**仓库**：https://github.com/OthmanAdi/planning-with-files
**安装位置**：`~/.config/opencode/skills/planning-with-files/`

### 作用
规划文件技能，通过文件进行任务规划和进度追踪。创建 `task_plan.md`、`progress.md`、`findings.md` 等文件来管理复杂任务。

### 使用方式

**方式一：告诉 Agent 使用**
```
Use the planning-with-files skill for this task
```

**方式二：手动调用**
```
Create a task plan for [your task]
/planning-with-files
```

**核心文件**：
- `task_plan.md` - 任务计划
- `progress.md` - 进度追踪
- `findings.md` - 发现记录

### 局限性
OpenCode 的 session 存储格式与 Claude Code 不同（OpenCode 使用 `.json` 文件在 `~/.local/share/opencode/storage/session/`），因此 `/clear` 后的 session catchup 功能有限。需要手动阅读 `task_plan.md`、`progress.md`、`findings.md` 来恢复上下文。

---

## 安装汇总

| 名称 | 类型 | 安装位置 |
|------|------|----------|
| oh-my-openagent | Plugin | npm global |
| using-superpowers | Skill | `~/.agents/skills/` |
| subtask2 | Plugin | npm global |
| planning-with-files | Skill | `~/.config/opencode/skills/` |

---

## 相关配置文件

- `~/.config/opencode/opencode.json` - 主配置
- `~/.config/opencode/oh-my-openagent.json` - Agent 配置（含 skills 配置）
