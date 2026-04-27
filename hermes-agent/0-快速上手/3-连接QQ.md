# OpenClaw 连接 QQ

本文介绍如何将 **QQ 机器人** 接入 OpenClaw，通过 QQ 与 AI 助手对话。腾讯为 OpenClaw 提供了简化入口，**扫码即可创建机器人**，无需走完整开放平台审批流程。

---

## 前置条件

- **QQ 号**：个人 QQ 即可，无需企业认证。
- **OpenClaw 已安装且网关在运行**：执行 `openclaw gateway status` 显示为 running。
- 若尚未配置模型，请先完成 [配置模型](2-配置模型.md)。

---

## 一、创建 QQ 机器人（获取 Token）

### 方式一：简化入口（推荐）

1. 在浏览器打开腾讯为 OpenClaw 提供的专用页面：  
   **https://q.qq.com/qqbot/openclaw/login.html**
2. 使用 **手机 QQ 扫码** 登录。
3. 若首次使用 QQ 开放平台，按提示完成 **实名认证**（姓名、身份证、手机号、人脸识别），约几分钟。
4. 登录后点击 **「创建机器人」**，无需填应用名称、描述或上传图标。
5. 创建完成后，页面会给出 **AppID** 和 **AppSecret**，并可能直接给出下面要用到的三条命令（含已拼好的 Token）。请复制保存 **AppID** 和 **AppSecret**，或整条命令备用。

![](https://raw.githubusercontent.com/xiaoshan1234/picture/master/res/2026-03-10-09-51-885efba72ee3d62e1803c674ee87b5be.png)
### 方式二：QQ 开放平台标准流程

1. 访问 [QQ 开放平台](https://open.qq.com)，使用 QQ 登录并完成实名认证。
2. 创建机器人应用，在应用管理中获取 **AppID** 和 **AppSecret**。

**Token 格式**：在 OpenClaw 中配置时使用 `AppID:AppSecret`（中间英文冒号，无空格）。简化入口页若已给出完整命令，可直接复制其中的 Token 部分。

---

## 二、安装 QQ 机器人插件

在终端执行：

```bash
openclaw plugins install @sliverp/qqbot@latest
```

若因网络报 npm 相关错误，可先切换国内镜像再安装：

```bash
npm config set registry https://registry.npmmirror.com
openclaw plugins install @sliverp/qqbot@latest
```

验证：`openclaw plugins list` 中应能看到 `@sliverp/qqbot`。

---

## 三、在 OpenClaw 中添加 QQ 渠道

将上一步得到的 Token（`AppID:AppSecret`）填入以下命令并执行：

```bash
openclaw channels add --channel qqbot --token "AppID:AppSecret"
```

示例（请替换为你的实际值）：

```bash
openclaw channels add --channel qqbot --token "102917561:你的AppSecret"
```

**注意**：Token 必须是 `AppID:AppSecret` 格式，冒号分隔，不要有多余空格或引号错误。

---

## 四、重启网关使配置生效

```bash
openclaw gateway restart
```

等待几秒后检查状态：

```bash
openclaw gateway status
```

---

## 五、验证是否成功

1. 在 **手机 QQ** 或 **电脑 QQ** 的「联系人」或搜索中找到刚创建的机器人。
2. 给机器人发一条消息（如「你好」）。
3. 若机器人能正常回复，说明接入成功。

若无回复，可按下方「常见问题」排查。

---

## 六、群聊使用说明

- **私聊**：默认已支持，直接发消息即可。
- **群聊**：将机器人拉入 QQ 群后，在群里 **@ 机器人** 才会触发回复（默认只响应被 @ 的消息）。

若需调整群消息策略，可编辑 OpenClaw 主配置文件：

- **Windows**：`%USERPROFILE%\.openclaw\openclaw.json`
- **Linux / macOS**：`~/.openclaw/openclaw.json`

在 `channels.qqbot` 中增加或修改 `groupPolicy`：

```json
{
  "channels": {
    "qqbot": {
      "enabled": true,
      "groupPolicy": "open"
    }
  }
}
```

| groupPolicy   | 说明 |
|---------------|------|
| `"off"`       | 不处理群消息，仅私聊（默认多为仅 @ 才回复，视插件实现而定） |
| `"allowlist"` | 仅响应指定群（需配合 allowlist 配置） |
| `"open"`      | 接收所有群消息（慎用，易产生大量调用） |

修改后需执行 `openclaw gateway restart` 生效。

---

## 七、常用命令

| 命令 | 说明 |
|------|------|
| `openclaw gateway status` | 查看网关状态 |
| `openclaw gateway restart` | 重启网关 |
| `openclaw logs --follow` | 查看实时日志（排错时使用） |
| `openclaw plugins list` | 查看已安装插件 |

---

## 八、常见问题

| 现象 | 处理建议 |
|------|----------|
| 安装插件报 npm 错误 | 使用 `npm config set registry https://registry.npmmirror.com` 后重新执行安装命令。 |
| 机器人不回消息 | ① 确认开放平台中机器人状态为「已上线」；② 检查 Token 是否为 `AppID:AppSecret` 格式、无多余空格；③ `openclaw logs --follow` 查看 qqbot 相关报错；④ `openclaw gateway status` 确认网关在运行。 |
| Token 泄露 | 在 QQ 开放平台找到该机器人，点击「重置 AppSecret」，然后重新执行 `openclaw channels add --channel qqbot --token "AppID:新AppSecret"` 并 `openclaw gateway restart`。 |
| Docker 内安装插件报权限错误 | 以 root 或 openclaw 用户进入容器后再执行安装与配置命令。 |

---

## 九、补充说明

- **无需公网 IP / Webhook**：QQ 机器人插件通过 **WebSocket 长连接** 主动连接腾讯服务器，因此无需公网域名、SSL 证书或配置 Webhook，NAT 内网、纯 IP 的 VPS 也可使用，只要机器能访问外网即可。
- **多机器人**：一个 QQ 号最多可创建 **5 个** 机器人，每个机器人可绑定不同 OpenClaw 实例，会话与记忆相互独立。
- **与其他渠道并存**：OpenClaw 支持同时接入 QQ、飞书、Telegram 等，多渠道互不冲突。

更多玩法可参考 [腾讯云：OpenClaw 接入 QQ 机器人](https://cloud.tencent.com/developer/article/2635190) 及社区教程。
