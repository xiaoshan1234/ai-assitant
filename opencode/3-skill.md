# OpenCode：如何「导入」与使用 Skill

> 主文档：[Agent Skills](https://opencode.ai/docs/skills/)  
> OpenCode 没有单独的「导入」数据库操作：**把符合规范的 `SKILL.md` 放到扫描目录下**，启动后会被发现；Agent 通过内置 **`skill` 工具**按需加载全文。

---

## 一、Skill 是什么

- 每个 Skill 是**一个目录**，目录内必须有 **`SKILL.md`**（文件名全大写）。
- 元数据写在 **`SKILL.md` 顶部的 YAML Frontmatter**，正文为 Markdown 说明。
- 运行时会在 `skill` 工具描述里列出名称与摘要；需要时 Agent 调用 `skill({ name: "<name>" })` 加载完整内容。

---

## 二、「导入」的两种方式

### 1. 手动放置（官方文档主推，与仓库/本机目录直接对应）

把别人提供的 Skill **整个文件夹**（内含 `SKILL.md`）拷到下面**任一**位置即可，无需额外「注册」命令。

| 范围 | 路径 |
|------|------|
| **项目内（可随 Git 提交）** | `.opencode/skills/<skill-name>/SKILL.md` |
| **用户全局** | `~/.config/opencode/skills/<skill-name>/SKILL.md` |
| **与 Claude Code 兼容（项目）** | `.claude/skills/<skill-name>/SKILL.md` |
| **与 Claude Code 兼容（全局）** | `~/.claude/skills/<skill-name>/SKILL.md` |
| **与 `.agents` 约定兼容（项目）** | `.agents/skills/<skill-name>/SKILL.md` |
| **与 `.agents` 约定兼容（全局）** | `~/.agents/skills/<skill-name>/SKILL.md` |

**发现规则简述：**

- 对**项目路径**：从当前工作目录**向上**遍历到 **Git 工作区根**，沿途所有匹配的 `skills/*/SKILL.md` 都会参与发现。
- **全局**路径下的 Skill 始终会加载。

因此「从 GitHub 下载 ZIP / `git clone` 再复制到 `.opencode/skills/某目录/`」就是最常见的导入方式。

### 2. 使用生态 CLI（可选）

社区里有面向多工具（含 OpenCode）的 Skill 安装器，例如 Vercel 维护的 **[vercel-labs/skills](https://github.com/vercel-labs/skills)**，典型用法为：

```bash
npx skills add <owner/repo>
```

具体子命令（如 `list`、`find`、`update`、`remove`）以该仓库 **README** 为准；安装目标目录通常会落到上述 **`.opencode/skills`** 或 **`~/.config/opencode/skills`** 等约定路径（以 CLI 当前行为为准）。

若未使用 CLI，**仅手动复制文件夹**同样有效；OpenCode 官方文档**不依赖**特定 `npx` 命令。

---

## 三、编写 `SKILL.md` 的硬性要求

**Frontmatter 中仅识别这些字段：**

- **`name`**（必填）
- **`description`**（必填）
- **`license`**（可选）
- **`compatibility`**（可选）
- **`metadata`**（可选，字符串键值对）

其它字段会被忽略。

**`name` 校验：**

- 长度 1–64。
- 仅小写字母、数字、单段连字符；不能以 `-` 开头或结尾；不能出现连续 `--`。
- **必须与所在目录名一致**（正则：`^[a-z0-9]+(-[a-z0-9]+)*$`）。

**`description`：** 长度 1–1024，写清楚适用场景，便于 Agent 选对 Skill。

官方示例见文档中的 `git-release` 示例（`.opencode/skills/git-release/SKILL.md`）。

---

## 四、权限（`opencode.json`）

默认可用模式匹配控制谁能加载哪些 Skill：

```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "pr-review": "allow",
      "internal-*": "deny",
      "experimental-*": "ask"
    }
  }
}
```

| 值 | 行为 |
|----|------|
| `allow` | 直接允许加载 |
| `deny` | 对 Agent 隐藏并拒绝加载 |
| `ask` | 加载前询问用户 |

可为**自定义 Agent** 在 Frontmatter 里写 `permission.skill`，或为**内置 Agent** 在 `opencode.json` 的 `agent.<name>.permission` 中覆盖（见官方文档）。

若某 Agent **完全不需要** Skill，可设 `tools.skill: false`（自定义 Agent 用 YAML Frontmatter，内置 Agent 用 `opencode.json` 的 `agent.<name>.tools`）。

---

## 五、不生效时排查

1. 文件名是否为 **`SKILL.md`**（全大写）。
2. Frontmatter 是否含 **`name`** 与 **`description`**。
3. **`name` 是否与文件夹名一致**且符合正则。
4. 多个路径下 **Skill 名是否冲突**（需唯一）。
5. **`permission.skill` 是否把该 Skill 设成了 `deny`**。

---

## 六、小结

| 问题 | 做法 |
|------|------|
| 如何导入？ | 将含 `SKILL.md` 的目录放到 `.opencode/skills/` 或 `~/.config/opencode/skills/`（或兼容的 `.claude`/`.agents` 路径）。 |
| 是否要执行导入命令？ | 官方流程**不需要**；复制/克隆到上述目录即可。可选使用 `npx skills` 等 CLI 辅助安装。 |
| 如何被使用？ | 自动出现在 `skill` 工具列表中，由 Agent 按需 `skill({ name: "..." })` 加载。 |

更细的权限与 Agent 覆盖示例见 [Agent Skills | OpenCode](https://opencode.ai/docs/skills/)。
