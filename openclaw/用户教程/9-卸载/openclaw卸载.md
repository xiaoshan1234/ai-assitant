OpenClaw 完全卸载指南
OpenClaw（社区常称"小龙虾"）是一款开源、本地优先的AI Agent框架。以下是适用于Windows/macOS/Linux三大平台的完整卸载流程，包含官方推荐方法与手动清理步骤，确保彻底移除所有组件。

---
一、卸载前准备
1. 备份重要数据（可选）
配置文件与对话记录默认存储路径：
- Windows: C:\Users\<你的用户名>\.openclaw\
- macOS/Linux: ~/.openclaw/
如需保留数据，建议先复制上述目录至其他位置。
2. 关闭相关进程
- 关闭所有OpenClaw窗口与终端会话
- 停止后台服务（后续步骤会详细说明）

---
二、官方一键卸载（推荐）
这是最简便的卸载方式，适用于通过官方脚本或npm安装的OpenClaw。
2.1 标准一键卸载
在终端中执行以下命令（管理员/root权限）：
# 适用于已安装openclaw命令的情况
openclaw uninstall --all --yes --non-interactive

# 如openclaw命令已失效，使用npx执行
npx -y openclaw uninstall --all --yes --non-interactive
2.2 一键卸载说明
该命令会自动完成：
- 停止并卸载Gateway网关服务
- 移除全局npm包
- 删除状态目录与配置文件
- 清理工作区文件

---
三、分平台手动卸载步骤
若一键卸载失败或需手动清理，可按以下步骤操作。
3.1 Windows系统（PowerShell管理员模式）
# 1. 停止所有OpenClaw进程
Stop-Process -Name "openclaw" -Force -ErrorAction SilentlyContinue

# 2. 停止并卸载Gateway服务
openclaw gateway stop
openclaw gateway uninstall

# 3. 卸载全局npm模块
npm uninstall -g openclaw

# 4. 清理残留文件
Remove-Item -Recurse -Force "$env:USERPROFILE\.openclaw"  # 状态与配置
Remove-Item -Recurse -Force "$env:APPDATA\npm\node_modules\openclaw"  # npm安装目录
Remove-Item -Force "$env:APPDATA\npm\openclaw.cmd"  # 命令行入口
Remove-Item -Force "$env:USERPROFILE\.local\bin\openclaw"  # 包装脚本（如有）

# 5. 清理环境变量（如有自定义）
# 打开系统属性 → 高级 → 环境变量，删除OPENCLAW相关条目
3.2 macOS/Linux系统（终端）
# 1. 停止并卸载Gateway服务
openclaw gateway stop
openclaw gateway uninstall

# 2. 卸载全局npm模块
npm uninstall -g openclaw

# 3. 清理残留文件
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"  # 状态与配置
rm -f ~/.local/bin/openclaw  # 包装脚本（如有）
rm -rf ~/openclaw  # 源码安装目录（如有）
rm -rf "$(npm config get prefix)/lib/node_modules/openclaw"  # npm安装目录
rm -f "$(npm config get prefix)/bin/openclaw"  # 命令行入口

# 4. 清理自定义配置文件（如有）
if [ -n "$OPENCLAW_CONFIG_PATH" ] && [ "$OPENCLAW_CONFIG_PATH" != "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/config.yaml" ]; then
  rm -f "$OPENCLAW_CONFIG_PATH"
fi

# 5. 清理工作区（可选，删除所有Agent数据）
rm -rf ~/.openclaw/workspace

---
四、特殊安装方式的卸载
4.1 Docker容器安装版
# 1. 停止并删除OpenClaw容器
docker stop openclaw_container
docker rm openclaw_container

# 2. 删除OpenClaw镜像
docker rmi openclaw_image

# 3. 清理挂载的数据卷（如有）
docker volume rm openclaw_volume

# 4. 清理本地工作目录（如有）
rm -rf ~/openclaw-docker/workspace
4.2 源码手动安装版
# 1. 停止服务（如已手动启动）
pkill -f "openclaw"
cd ~/openclaw && ./scripts/stop.sh  # 如提供停止脚本

# 2. 卸载服务（如已手动安装为系统服务）
sudo systemctl disable openclaw
sudo systemctl stop openclaw
sudo rm /etc/systemd/system/openclaw.service
sudo systemctl daemon-reload

# 3. 删除源码目录
rm -rf ~/openclaw  # 替换为实际安装路径

# 4. 清理配置与数据
rm -rf ~/.openclaw

---
五、验证卸载是否彻底
5.1 命令行验证
# 检查命令是否存在
which openclaw && echo "卸载失败" || echo "卸载成功"

# 检查进程是否残留
ps aux | grep openclaw | grep -v grep  # Linux/macOS
Get-Process -Name "openclaw" -ErrorAction SilentlyContinue  # Windows

# 检查配置目录是否存在
ls -la ~/.openclaw  # Linux/macOS
Test-Path "$env:USERPROFILE\.openclaw"  # Windows
5.2 系统级验证
- Windows: 控制面板 → 程序和功能，确认无OpenClaw条目；注册表搜索"openclaw"并删除相关键值
- macOS: Spotlight搜索"openclaw"，确认无相关应用；检查/Applications目录
- Linux: 检查/usr/local/bin、/usr/bin等目录，确认无openclaw可执行文件

---
六、常见问题处理
1. 命令未找到：使用完整路径执行或通过npx调用（npx openclaw ...）
2. 权限不足：以管理员/root身份重新运行命令
3. 进程无法停止：使用任务管理器（Windows）或kill -9 <PID>（Linux/macOS）强制终止
4. 卸载后仍有残留：使用系统搜索工具查找"openclaw"相关文件并手动删除

---
总结
推荐优先使用官方一键卸载命令，可快速完成所有清理工作。若遇到问题，再按平台执行手动卸载步骤，确保彻底移除OpenClaw及其所有组件，释放系统资源。