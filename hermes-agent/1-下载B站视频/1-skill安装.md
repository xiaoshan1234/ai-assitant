Hermes 从插件市场安装 Skill 完整指南
Hermes 中的 Skill 并非传统意义上的“插件”，而是可复用的工作流文档，用于指导 Hermes 完成特定任务，其安装主要通过官方插件市场（Skills Hub）实现，支持 命令行 和 Web UI 两种方式，适配不同操作习惯，以下是详细步骤（适配 Hermes 最新版本，兼容 Linux/macOS/WSL2 环境，原生 Windows 需先配置 WSL2）。
一、前置准备
1. 环境验证
确保 Hermes 已成功安装且能正常运行，同时满足以下基础条件：
- 操作系统：Linux、macOS 或 Windows WSL2（官方不支持原生 Windows 环境）
- Python 版本：≥3.10
- Hermes 版本：最新稳定版（可通过 hermes version 查看，需通过 hermes update 更新至最新）
2. 核心自检（必做）
安装 Skill 前，需先通过以下命令完成系统自检，确保无基础配置错误，避免安装失败：
# 运行 Hermes 诊断工具，所有项目需显示绿色 ✓
hermes doctor

若出现红色报错，需先解决对应问题（如 API Key 未配置、依赖缺失等），再进行后续操作，否则 Skill 安装后无法正常运行。
3. 插件市场说明
Hermes 官方插件市场（Skills Hub）核心分为两类 Skill 来源，安装均通过统一命令或界面操作：
- 内置 Skill：Hermes 安装时自动同步，70+ 个，覆盖写作、数据分析、DevOps 等 15+ 类别，无需额外安装，可直接通过斜杠命令触发。
- 社区 Skill：通过 agentskills.io 提供，经 65+ 项安全规则扫描，支持一键安装，是插件市场的核心可扩展资源。
Skill 安装后默认存储在 ~/.hermes/skills/ 目录，按类别分文件夹管理，可手动查看或管理。
二、两种安装方式（优先推荐命令行，简洁高效）
方式一：命令行安装（最常用，适配所有环境）
通过 Hermes 自带的 hermes skills 系列命令，可快速完成 Skill 的搜索、安装、验证，步骤如下：
步骤 1：搜索目标 Skill
先通过搜索命令查找插件市场中的目标 Skill，支持关键词搜索（中文/英文均可）：
# 通用搜索格式：hermes skills search 关键词
# 示例1：搜索公文写作相关 Skill（中文关键词）
hermes skills search 公文写作
# 示例2：搜索会议纪要相关 Skill（英文关键词）
hermes skills search meeting-notes
# 示例3：搜索指定作者/仓库的 Skill
hermes skills search skills-sh/document-writer

搜索结果会显示 Skill 的名称、描述、版本及安装命令，确认目标 Skill 后，执行安装操作。
步骤 2：安装 Skill
使用hermes skills install 命令安装，直接复制搜索结果中的安装命令即可，示例如下：
# 通用安装格式：hermes skills install Skill名称/仓库路径
# 示例1：安装公文写作 Skill
hermes skills install skills-sh/document-writer
# 示例2：安装会议纪要 Skill
hermes skills install skills-sh/meeting-notes
# 示例3：安装网页搜索增强 Skill（查政策、资料必备）
hermes skills install skills-sh/web-search-enhance
安装过程中，终端会显示下载进度和安装状态，出现 Installed successfully 提示即表示安装完成。
步骤 3：验证安装
安装完成后，通过以下命令查看已安装的 Skill，确认目标 Skill 已成功加载：
# 查看所有已安装的 Skill（按类别分组显示）
hermes skills list
# 查看指定 Skill 的详细信息（如版本、描述、使用方式）
hermes skills info skills-sh/document-writer

也可直接在 Hermes 对话中通过斜杠命令触发 Skill（如 /document-writer），能正常响应即表示安装成功。
方式二：Web UI 安装（可视化操作，适合不熟悉命令行的用户）
Hermes 最新版本支持 Web 图形界面（Dashboard），可通过可视化操作完成 Skill 安装，步骤如下：
步骤 1：启动 Web UI（Dashboard）
先更新 Hermes 至最新版本，再启动 Web 管理界面：
# 1. 更新 Hermes 至最新版本（确保支持 Web UI）
hermes update
# 2. 启动 Web Dashboard，默认端口 9119
hermes dashboard

步骤 2：访问 Web UI
打开浏览器，输入地址 127.0.0.1:9119，即可进入 Hermes Web 管理界面（无需登录，本地访问）。
步骤 3：安装 Skill
1. 在 Web UI 左侧导航栏中，点击 Skills 选项，进入 Skill 管理页面；
2. 页面顶部有搜索框，输入目标 Skill 的关键词（如“公文写作”“meeting-notes”），点击搜索；
3. 在搜索结果中，找到目标 Skill，点击右侧 Install 按钮，等待安装完成；
4. 安装完成后，目标 Skill 会显示在“已安装 Skill”列表中，可直接点击开关启用或禁用。
Web UI 还支持批量启用/禁用 Skill，按分类筛选已安装 Skill，操作更直观。
三、安装后的后续操作
1. 启用/禁用 Skill
安装后的 Skill 默认启用，可根据需求手动启用或禁用：
# 命令行方式：启用指定 Skill
hermes skills enable skills-sh/document-writer
# 命令行方式：禁用指定 Skill
hermes skills disable skills-sh/document-writer
# Web UI 方式：直接在 Skills 页面点击 Skill 右侧的开关即可

2. 更新 Skill
当插件市场中的 Skill 有更新时，可通过以下命令更新：
# 通用更新格式：hermes skills update Skill名称/仓库路径
hermes skills update skills-sh/document-writer
# 更新所有已安装的 Skill
hermes skills update --all

3. 卸载 Skill
若无需使用某个 Skill，可通过以下命令卸载：
# 通用卸载格式：hermes skills uninstall Skill名称/仓库路径
hermes skills uninstall skills-sh/document-writer
# 卸载后可通过 hermes skills list 确认卸载结果

四、常见问题与解决方法
1. 问题1：执行 hermes skills 命令提示“command not found”

      解决：安装后未刷新环境变量，执行 source ~/.bashrc（Linux/macOS）或 source ~/.zshrc（macOS 部分终端），刷新后重新执行命令。
2. 问题2：Skill 安装成功，但无法通过斜杠命令触发

      解决：1. 执行 hermes doctor 检查是否有配置错误；2. 重启 Hermes（关闭终端重新启动）；3. 确认 Skill 已启用（通过 hermes skills list 查看启用状态）。
      
3. 问题3：搜索不到目标 Skill

      解决：1. 确认关键词拼写正确，可尝试中英文关键词切换；2. 执行 hermes update 更新 Hermes 及插件市场索引；3. 直接使用 Skill 完整仓库路径安装（可从 agentskills.io 查找）。
      
4. 问题4：安装过程中提示“网络超时”

      解决：检查网络连接，确保能正常访问 GitHub 和 agentskills.io；国内用户可尝试切换网络环境，或使用国内加速镜像重新安装。
      
5. 问题5：Skill 安装后报错“依赖缺失”

      解决：根据终端提示，安装对应的依赖包（如 Python 依赖可通过 pip install 依赖包名称安装），安装完成后重新启用 Skill。
      
五、补充说明
- Skill 是“按需加载”的工作流文档，安装多个不会占用过多系统资源，也不会影响 Hermes 运行速度。
- 推荐优先使用官方插件市场（agentskills.io）的 Skill，均经过安全扫描，避免安装非官方来源的 Skill，防止安全风险。
- 若需自定义 Skill，可在 ~/.hermes/skills/ 目录下手动创建文件夹，编写 SKILL.md 文档，Hermes 会自动识别。
- 从 OpenClaw 迁移至 Hermes 的用户，可通过 hermes claw migrate 命令，将 OpenClaw 的 playbook 转换为 Hermes 可识别的 Skill。