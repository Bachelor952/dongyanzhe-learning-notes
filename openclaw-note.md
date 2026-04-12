🚀 OpenClaw 半自动化求职助手搭建与实践（WSL + Edge + CDP）
基于 WSL 环境 + Edge 浏览器 + CDP 协议，实现招聘平台自动化操作
一、项目背景
在求职投递（Boss 直聘、智联招聘等平台）过程中，存在大量低效问题：
招聘信息分散，人工获取效率低
简历投递流程重复，需反复填写个人信息
单次完整投递耗时约 20~30 分钟
项目目标：使用 OpenClaw 构建半自动化求职助手，核心能力：
自动打开招聘网站
辅助岗位信息智能分析
浏览器自动化控制（支持后续扩展自动投递功能）
二、整体技术架构
text
Windows（主机）
│
├── Microsoft Edge（浏览器自动化载体）
│   └── 开启 CDP 调试端口
│
├── WSL（Ubuntu 22.04）
│   └── OpenClaw（Agent 控制核心）
│
└── OpenClaw Gateway（本地网关服务）
    └── 通过 CDP 协议控制浏览器
三、环境搭建
1. WSL 环境检查
执行命令确认 WSL 版本：
bash
运行
wsl -l -v
✅ 要求：Ubuntu-22.04 + WSL2
2. WSL 安装 Microsoft Edge
bash
运行
# 下载微软软件源配置包
curl -sSL -O https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb
# 安装软件源
sudo dpkg -i packages-microsoft-prod.deb
# 更新软件源
sudo apt-get update
# 安装Edge浏览器
sudo apt-get install microsoft-edge-stable
验证安装是否成功：
bash
运行
microsoft-edge --version
四、关键问题与解决方案
❌ 问题 1：Edge 无法安装
报错信息：
plaintext
E: Package 'microsoft-edge-stable' has no installation candidate
原因：软件源未正确更新解决方案：
bash
运行
sudo apt-get update
sudo apt-get install microsoft-edge-stable
❌ 问题 2：OpenClaw 启动浏览器失败
报错信息：
plaintext
Missing X server or $DISPLAY
原因：WSL 为无 GUI 环境，无法直接运行图形化浏览器核心解决方案：改用 Windows 本地浏览器 + CDP 协议 实现控制
五、核心方案：浏览器接管（推荐方案）
Step 1：Windows 端启动 Edge 调试模式
cmd
msedge --remote-debugging-port=18800 --user-data-dir="C:\openclaw-profile"
18800：CDP 调试端口
--user-data-dir：创建独立浏览器配置，避免污染主浏览器数据
Step 2：修改 OpenClaw 配置文件
bash
运行
nano ~/.openclaw/openclaw.json
添加配置内容：
json
{
  "browser": {
    "cdpUrl": "http://127.0.0.1:18800"
  }
}
Step 3：启动 OpenClaw 网关服务
bash
运行
# 启动网关
openclaw gateway start
# 查看状态
openclaw gateway status
Step 4：启动浏览器控制
bash
运行
openclaw browser --browser-profile openclaw start
Step 5：自动打开招聘网站
bash
运行
openclaw browser --browser-profile openclaw open https://www.zhipin.com/
六、运行效果
✅ 自动打开 Boss 直聘 / 智联招聘等平台✅ 中文字体正常显示，无乱码✅ 浏览器完全受 OpenClaw 控制✅ 控制台可正常下发自动化指令
七、常见问题汇总
❌ 问题：中文乱码 / 方块字
原因：WSL 缺少中文字体解决：
bash
运行
sudo apt install fonts-noto-cjk
❌ 问题：Gateway 端口占用
报错信息：Port 18789 is already in use解决：
bash
运行
openclaw gateway stop
openclaw gateway start
❌ 问题：浏览器数据污染
原因：使用默认 Edge 用户配置解决：启动时添加独立数据目录
cmd
--user-data-dir="C:\openclaw-profile"
❌ 问题：插件警告
提示信息：plugins.allow is empty说明：仅提示信息，不影响核心功能可选优化：
json
"plugins": {
  "allow": ["openclaw-weixin"]
}
八、使用规范（重要）
启动顺序
Windows 端启动 Edge 调试模式
WSL 端启动 OpenClaw Gateway
WSL 端启动浏览器控制
WSL 端执行网页打开指令
关闭建议
bash
运行
# 优雅关闭
openclaw browser stop
openclaw gateway stop
直接关闭终端也可，无严重影响
九、项目总结
本项目已完成核心功能落地：✅ OpenClaw 在 WSL 环境完整部署✅ 基于 CDP 协议实现浏览器自动化控制✅ 成功实现招聘网站自动打开✅ 完成多类环境异常问题排查与解决
十、后续优化方向
🔄 自动筛选匹配岗位🔄 自动提取岗位核心信息🔄 高阶功能：自动投递简历🔄 接入 AI 大模型，分析岗位 - 简历匹配度
十一、经验总结
WSL 环境不适合直接运行 GUI 程序（如浏览器）
浏览器自动化最优方案：Windows 本地浏览器 + CDP 协议
必须做浏览器环境隔离（user-data-dir）
OpenClaw 定位：自动化控制中枢，而非单纯的爬虫工具
