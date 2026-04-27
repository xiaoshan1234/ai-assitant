# OpenClaw Skill：安装、使用与作用说明

本文说明 OpenClaw **Skill（技能）** 是什么、有什么作用，以及如何**下载（安装）**、**启用**和**使用** Skill。

---

## 一、Skill 是什么、有什么作用

### 1. 定义

**Skill** 是 OpenClaw 的**能力扩展单元**。每个 Skill 本质上是一个**文件夹**，里面至少包含一个带 YAML 头部和说明的 **SKILL.md** 文件，用于「教」智能体如何完成一类任务——例如调用某个工具、遵循某套步骤、或使用某条 API。

- 兼容 [AgentSkills](https://agentskills.io/) 规范。
- OpenClaw 会在启动/加载时读取这些 Skill，并根据环境、配置和依赖（如是否安装某二进制、是否配置某 API Key）决定是否启用。

### 2. 作用与意义


| 作用           | 说明                                                   |
| ------------ | ---------------------------------------------------- |
| **扩展 AI 能力** | 让智能体从「只会对话」变成「能查网页、跑命令、发邮件、管日历」等，按需安装、按需启用。          |
| **标准化与复用**   | 技能以统一格式（SKILL.md + 元数据）描述，便于在社区共享、版本管理和更新。           |
| **按场景组合**    | 不同用户可安装不同 Skill 组合，办公、开发、生活等场景各取所需。                  |
| **安全与可控**    | 可在配置里对每个 Skill 开关（enabled）、传环境变量或 API Key，并对高危能力做审批。 |


总结：**Skill 的意义 = 给 OpenClaw 装上可插拔的「技能包」，安装后通过配置启用，在对话中由 AI 按需调用。**

---

## 二、Skill 从哪里来（加载位置与优先级）

OpenClaw 从**三个位置**加载 Skill，优先级从高到低：


| 优先级   | 位置                             | 说明                                |
| ----- | ------------------------------ | --------------------------------- |
| 1（最高） | **工作区** `./skills/`            | 当前项目专用，ClawHub 默认也会安装到这里（若在工作区内）。 |
| 2     | **本地托管** `~/.openclaw/skills/` | 当前用户下所有会话可用。                      |
| 3（最低） | **内置**                         | 随 OpenClaw 安装包自带的 Skill。          |


同一名称的 Skill 若在多个位置存在，**高优先级会覆盖低优先级**。  
「下载」一个 Skill 指的就是：通过 ClawHub 或手动，把 Skill 文件夹放到上述某一位置（通常是 `./skills/` 或 `~/.openclaw/skills/`）。

---

## 三、下载（安装）Skill

推荐使用 **ClawHub**——OpenClaw 的公共 Skill 注册中心，用一条命令即可搜索并安装到本地。

### 1. 安装 ClawHub CLI（仅需一次）

任选其一：

```bash
npm i -g clawhub
# 或国内镜像
npm i -g clawhub --registry https://registry.npmmirror.com
```

```bash
pnpm add -g clawhub
```

不装全局也可用 npx：

```bash
npx clawhub@latest search "日历"
npx clawhub@latest install <skill-slug>
```

### 2. 搜索 Skill

按关键词搜索（支持自然语言/语义搜索）：

```bash
clawhub search "日历"
clawhub search "browser automation"
clawhub search "postgres backups"
```

会列出名称、描述等，记下你要安装的 **slug**（技能标识符）。

### 3. 安装（下载）到本地

```bash
clawhub install <skill-slug>
```

例如：

```bash
clawhub install browser
clawhub install calendar
```

- 默认会安装到**当前工作目录**下的 `./skills/`；若当前目录是 OpenClaw 工作区，则安装后 OpenClaw 会从该处加载。
- 若要安装到用户目录：可先 `cd ~/.openclaw` 再执行 `clawhub install <slug>`，或通过 `--dir` / `CLAWHUB_WORKDIR` 指定目录（具体见 `clawhub --help`）。

### 4. 指定版本与覆盖

```bash
clawhub install <skill-slug> --version 1.0.0
clawhub install <skill-slug> --force   # 已存在则覆盖
```

### 5. 查看已安装的 Skill（ClawHub 侧）

```bash
clawhub list
```

会读取当前工作目录下的 `.clawhub/lock.json`，列出通过 ClawHub 安装的 Skill。

### 6. 更新已安装的 Skill

```bash
clawhub update <skill-slug>      # 更新单个
clawhub update --all             # 批量更新
```

---

## 四、启用与配置 Skill（使用前的设置）

安装只是把 Skill **放到加载路径**；是否**启用**、是否**传 API Key 等**，在 OpenClaw 的配置里完成。

### 1. 配置文件位置

- **Windows**：`%USERPROFILE%\.openclaw\openclaw.json`
- **Linux / macOS**：`~/.openclaw/openclaw.json`

### 2. 启用 / 禁用某个 Skill

在 `openclaw.json` 的 `skills.entries` 里为对应 Skill 设置 `enabled`：

```json
{
  "skills": {
    "entries": {
      "browser": { "enabled": true },
      "calendar": { "enabled": true },
      "某个技能名": { "enabled": false }
    }
  }
}
```

- 未在 `entries` 里列出的 Skill，若在加载路径中存在且满足其依赖（环境变量、二进制等），一般会按该 Skill 的默认逻辑加载。
- 设为 `false` 即禁用该 Skill。

### 3. 为 Skill 配置环境变量或 API Key

部分 Skill 需要 API Key、Token 等，可在对应 entry 下写 `env` 或 `config`：

```json
{
  "skills": {
    "entries": {
      "某技能": {
        "enabled": true,
        "env": {
          "GEMINI_API_KEY": "你的密钥"
        },
        "config": {
          "endpoint": "https://api.example.com"
        }
      }
    }
  }
}
```

具体字段以该 Skill 的 SKILL.md 或文档为准。

### 4. 使配置生效

修改 `openclaw.json` 后，**重启网关**即可：

```bash
openclaw gateway restart
```

新安装或新启用的 Skill 会在下一次会话中被加载。

---

## 五、如何使用 Skill（日常使用方式）

1. **安装**：用 `clawhub install <slug>` 把 Skill 下载到 `./skills/` 或 `~/.openclaw/skills/`。
2. **启用与配置**：在 `openclaw.json` 的 `skills.entries` 里设置 `enabled: true`，并按需填写 `env` / `config`。
3. **重启网关**：`openclaw gateway restart`。
4. **在对话中使用**：在 QQ、飞书、钉钉、Telegram 或 Control UI 里**用自然语言描述任务**，AI 会根据已加载的 Skill 自动选择是否调用。例如：
  - 装了「浏览器」类 Skill 后：「打开 [https://example.com](https://example.com) 并截图」
  - 装了「日历」类 Skill 后：「帮我查一下本周的日程」
  - 装了「搜索」类 Skill 后：「查一下今天某某公司的最新新闻」

**你不需要背命令**：只要描述清楚目标，AI 会决定用哪个工具（Skill）。

---

## 六、常用 ClawHub 命令速查


| 命令                                                    | 说明                        |
| ----------------------------------------------------- | ------------------------- |
| `clawhub search "关键词"`                                | 搜索 Skill                  |
| `clawhub install <slug>`                              | 安装（下载）Skill               |
| `clawhub list`                                        | 列出已安装的 Skill（ClawHub 管理的） |
| `clawhub update <slug>` / `clawhub update --all`      | 更新 Skill                  |
| `clawhub login` / `clawhub whoami` / `clawhub logout` | 登录 / 查看身份 / 登出（发布等操作时需要）  |


---

## 七、推荐优先安装的 Skill（参考）

可根据需求选择安装，常见推荐：

- **浏览器 / Browser**：网页自动化（打开、点击、填表、截图）。
- **搜索类（如 Brave Search、web-search）**：联网搜索、实时信息。
- **Shell / Exec**：在受控环境下执行终端命令。
- **Cron / 定时**：定时任务与提醒（参见 [计划任务](../3-计划与任务/订阅早报.md)）。
- **日历 / Calendar**：日程查询与创建。
- **ClawHub**：技能市场本体，便于继续搜索和安装其他 Skill。

安装示例：

```bash
clawhub install browser
clawhub install calendar
```

具体名称以 `clawhub search "关键词"` 的搜索结果为准。

---

## 八、安全与注意

- **来源**：优先从 ClawHub 等官方/社区认可渠道安装，注意下载量、更新时间和评价。
- **权限**：涉及 Gmail、GitHub、支付等账号的 Skill，授权前务必确认用途与可信度。
- **高危能力**：浏览器、Shell 等可执行外部操作的 Skill，建议在 OpenClaw 中开启**审批/确认模式**（如 `exec.ask: "on"`），避免误操作。
- **更新**：定期执行 `clawhub update --all` 获取安全与功能更新。

---

## 九、参考链接

- [OpenClaw Skills 官方说明](https://openclawcn.com/docs/tools/skills/)
- [ClawHub 文档（搜索、安装、发布）](https://openclaw.cc/tools/clawhub.html)
- ClawHub 网站：[clawhub.com](https://clawhub.com/)

---

**总结**：Skill = OpenClaw 的「技能包」，**作用**是扩展 AI 能力；**下载**用 `clawhub install <slug>`；**使用** = 在配置里启用并填好 API Key 等 → 重启网关 → 在对话里用自然语言发任务，由 AI 自动选用对应 Skill。