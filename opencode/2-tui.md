# OpenCode TUI：功能与快捷键

> 参考：[TUI 文档](https://opencode.ai/docs/tui/)、[快捷键（Keybinds）](https://opencode.ai/docs/keybinds/)

---

## 一、TUI 是什么、如何进入

OpenCode 提供**终端交互界面（TUI）**，在当前目录下与 LLM 协作开发项目。

- 启动（当前目录）：`opencode`
- 指定工作目录：`opencode /path/to/project`

进入后可像聊天一样输入提示；多数操作还支持 `@` 引用、`!` 执行命令、`/` 斜杠命令，以及下文中的**前导键（Leader）+ 第二键**组合。

---

## 二、TUI 支持的核心交互方式

| 方式 | 说明 |
|------|------|
| **普通提问** | 在输入区直接输入自然语言提示并发送。 |
| **`@` 文件引用** | 输入 `@` 可对当前工作目录做**模糊文件搜索**，选中后文件内容会加入对话上下文。也可在句子里写 `@路径/文件` 提问。 |
| **`!` Shell** | 消息以 `!` 开头会作为 Shell 执行（如 `!ls -la`），输出作为工具结果进入对话。 |
| **`/` 斜杠命令** | 输入 `/` 加命令名快速执行动作（如 `/help`）；多数命令对应下文「Leader + 键」快捷键。 |
| **命令面板** | `Ctrl+P` 打开命令列表（默认，可在 `tui.json` 中改为 `command_list`）。 |
| **模式切换** | `Tab` / `Shift+Tab` 在 Agent 间循环（默认 `agent_cycle` / `agent_cycle_reverse`）。 |
| **模型变体** | `Ctrl+T` 循环模型变体（如是否带扩展推理等，`variant_cycle`）。 |
| **最近模型** | `F2` / `Shift+F2` 在 recently used 模型间切换。 |

---

## 三、斜杠命令与默认快捷键（Leader 默认 `Ctrl+X`）

文档中 **`<leader>` 表示先按 Leader，再按后面的键**。默认 Leader 为 **`Ctrl+X`**。

| 斜杠命令 | 作用 | 默认快捷键 |
|----------|------|------------|
| `/connect` | 添加提供商并配置 API 密钥 | — |
| `/compact`（`/summarize`） | 压缩当前会话 | `Ctrl+X` `C` |
| `/details` | 切换工具执行详情显示 | `Ctrl+X` `D` |
| `/editor` | 用外部编辑器（`EDITOR`）编写消息 | `Ctrl+X` `E` |
| `/exit`（`/quit` `/q`） | 退出 OpenCode | `Ctrl+X` `Q`；也可 `Ctrl+C` / `Ctrl+D` |
| `/export` | 导出当前对话为 Markdown 并用默认编辑器打开 | `Ctrl+X` `X` |
| `/help` | 帮助对话框 | `Ctrl+X` `H` |
| `/init` | 引导创建或更新 `AGENTS.md` | `Ctrl+X` `I` |
| `/models` | 列出可用模型 | `Ctrl+X` `M` |
| `/new`（`/clear`） | 新建会话 | `Ctrl+X` `N` |
| `/redo` | 重做已撤销的消息（需 Git 仓库以恢复文件） | `Ctrl+X` `R` |
| `/sessions`（`/resume` `/continue`） | 列出并切换会话 | `Ctrl+X` `L` |
| `/share` | 分享当前会话 | 文档记为 `Ctrl+X` `S`；默认 `tui.json` 里 `session_share` 常为 `none`，同组合可能绑定 **状态视图**（`status_view`），以 `/help` 与本地配置为准 |
| `/themes` | 主题列表 | `Ctrl+X` `T` |
| `/thinking` | 切换是否显示模型的 thinking/reasoning 块（仅显示，不开关模型推理能力） | — |
| `/undo` | 撤销上一轮对话并回滚相关文件变更（需 Git 仓库） | `Ctrl+X` `U` |
| `/unshare` | 取消分享当前会话 | — |

---

## 四、默认键位分类速查（摘自官方默认 `tui.json`）

以下可在项目或用户级 **`tui.json` / `tui.jsonc`** 中覆盖；`<leader>` 默认 `Ctrl+X`。

### 应用与会话

| 动作（配置键名） | 默认 |
|------------------|------|
| 退出 | `Ctrl+C`、`Ctrl+D`、`<leader> Q` |
| 打开外部编辑器 | `<leader> E` |
| 主题列表 | `<leader> T` |
| 侧栏开关 | `<leader> B` |
| 新建会话 | `<leader> N` |
| 会话列表 | `<leader> L` |
| 会话时间线 | `<leader> G` |
| 压缩会话 | `<leader> C` |
| 导出会话（Markdown） | `<leader> X` |
| 中断当前生成 | `Escape` |
| 子会话：进入第一个子会话 | `<leader> Down` |
| 子会话：下一个 / 上一个 | `Right` / `Left` |
| 返回父会话 | `Up` |

### 消息区浏览

| 动作 | 默认 |
|------|------|
| 上/下翻页 | `PageUp` / `PageDown`，或 `Ctrl+Alt+B` / `Ctrl+Alt+F` |
| 按行上/下 | `Ctrl+Alt+Y` / `Ctrl+Alt+E` |
| 半页上/下 | `Ctrl+Alt+U` / `Ctrl+Alt+D` |
| 跳到首条 / 末条消息 | `Ctrl+G` / `Home`，`Ctrl+Alt+G` / `End` |
| 复制消息 | `<leader> Y` |
| 撤销 / 重做消息（对话级） | `<leader> U` / `<leader> R` |
| 切换隐藏/显示（conceal 等） | `<leader> H`（与 tips 等可能共用，以配置为准） |

### 输入框（类 Emacs/Readline，节选）

| 动作 | 默认 |
|------|------|
| 提交 | `Enter` |
| 换行（不发送） | `Shift+Enter`、`Ctrl+Enter`、`Alt+Enter`、`Ctrl+J` |
| 行首/行尾 | `Ctrl+A` / `Ctrl+E` |
| 左/右一字 | `Left` / `Right` 或 `Ctrl+B` / `Ctrl+F` |
| 删到行尾/行首 | `Ctrl+K` / `Ctrl+U` |
| 删前一词/后一词 | `Ctrl+W` 等 / `Alt+D` 等 |
| 模型列表 | `<leader> M` |
| Agent 列表 | `<leader> A` |

将某项设为 **`"none"`** 可禁用对应快捷键。

---

## 五、外部编辑器与导出

`/editor`、`/export` 使用环境变量 **`EDITOR`**（如 `vim`、`nano`、`code --wait`）。GUI 编辑器通常需 **`--wait`** 阻塞至关闭。Windows 可在 PowerShell 中设置 `$env:EDITOR` 或系统环境变量。

---

## 六、TUI 配置（`tui.json`）

与 **`opencode.json`**（服务端/运行行为）分开，TUI 专用配置示例：

- **`theme`**：界面主题  
- **`keybinds`**：含 `leader` 及各动作键位  
- **`scroll_speed` / `scroll_acceleration`**：滚动速度或 macOS 风格加速  
- **`diff_style`**：`auto` / `stacked` 等 diff 展示  
- **`mouse`**：是否捕获鼠标（默认 `true`；关闭后更易用终端原生选区）

可通过环境变量 **`OPENCODE_TUI_CONFIG`** 指定配置文件路径。

命令面板（`Ctrl+X H` 或 `/help`）里可调部分显示选项（如用户名是否显示），部分会持久保存。

---

## 七、桌面端提示框

桌面应用输入框另有内置 **Readline/Emacs 风格** 快捷键（见官方 Keybinds 页的表格），与 `opencode.json` 无关。

---

## 八、小结

- TUI 支持：**对话、`@` 文件、`!` Shell、`/` 命令、命令面板、会话/模型/Agent 切换、主题与导出**等。  
- 多数组合键依赖 **Leader（默认 `Ctrl+X`）**；完整列表以 **[Keybinds 文档](https://opencode.ai/docs/keybinds/)** 与本地 **`/help`** 为准。  
- 自定义：编辑 **`tui.json`**，禁用某键写 **`"none"`**。
