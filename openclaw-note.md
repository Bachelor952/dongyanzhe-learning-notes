# OpenClaw 自动化求职助手部署与踩坑记录

## 一、项目背景

在求职过程中（尤其是投递央企/国企、BOSS直聘等平台），存在以下问题：

- 招聘信息分散，获取效率低
- 简历投递流程繁琐（重复填写信息）
- 单次投递耗时较长（约30分钟）

为提升效率，尝试使用 **OpenClaw** 构建半自动化求职助手，实现：

- 自动打开招聘网站
- 辅助岗位分析
- 后续实现自动填写表单（目标）

---

## 二、整体技术架构

```text
Windows（主机）
   ↓
WSL（Ubuntu 22.04）
   ↓
OpenClaw（Agent控制）
   ↓
CDP协议
   ↓
Microsoft Edge（浏览器自动化）
## 三、环境准备
1. WSL 安装与检查
wsl -l -v

确认版本：

Ubuntu-22.04
WSL2
2. 安装 Microsoft Edge（WSL）
curl -sSL -O https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install microsoft-edge-stable
## 四、关键问题与解决方案（核心部分）
❗问题1：sudo 密码错误
现象：
sudo dpkg -i packages-microsoft-prod.deb
Sorry, try again.
原因：

WSL 用户密码未设置或忘记

解决：
wsl -d Ubuntu-22.04 -u root
passwd 用户名
❗问题2：Edge 安装后无法使用
现象：
E: Package 'microsoft-edge-stable' has no installation candidate
原因：

软件源未正确更新

解决：
sudo apt-get update
❗问题3：OpenClaw 浏览器启动失败（核心问题）
报错：
Missing X server or $DISPLAY
The platform failed to initialize
📌 问题本质：

WSL 无图形界面（GUI），浏览器无法启动

✅ 解决方案（推荐）

使用 Windows 本地 Edge + 远程调试模式（CDP）

Step 1：启动 Windows Edge（关键）
msedge --remote-debugging-port=18800 --user-data-dir="C:\openclaw-profile"
参数说明：
参数	作用
remote-debugging-port	开启浏览器调试接口
user-data-dir	隔离浏览器数据（防止污染主账号）
Step 2：配置 OpenClaw
nano ~/.openclaw/openclaw.json

加入：

{
  "browser": {
    "cdpUrl": "http://127.0.0.1:18800"
  }
}
Step 3：启动服务
openclaw gateway start
Step 4：连接浏览器
openclaw browser --browser-profile openclaw start
Step 5：测试
openclaw browser --browser-profile openclaw open https://www.zhipin.com/
✅ 成功表现：
页面正常打开
无 DISPLAY 报错
OpenClaw 可控制浏览器
## 五、字体乱码问题
❗问题：

BOSS直聘页面中文乱码

原因：

WSL 缺少中文字体

解决：
sudo apt install fonts-noto-cjk
## 六、Gateway 端口冲突问题
❗问题：
Gateway already running (pid xxx)
Port 18789 is already in use
解决：
openclaw gateway stop
openclaw gateway start
## 七、插件提示问题
提示：
plugins.allow is empty
说明：

非官方插件（如 weixin）未显式允许

可选解决：
"plugins": {
  "allow": ["openclaw-weixin"]
}
## 八、安全性考虑（重点）
❗风险：
浏览器自动化可能污染个人账号
Cookie / 登录状态泄露
✅ 解决方案：

使用独立浏览器配置：

msedge --remote-debugging-port=18800 --user-data-dir="C:\openclaw-profile"
优势：
数据隔离
防封号
防隐私泄露
## 九、OpenClaw 使用流程总结
每次使用流程：
# 1. 启动浏览器（Windows）
msedge --remote-debugging-port=18800 --user-data-dir="C:\openclaw-profile"

# 2. 启动服务（WSL）
openclaw gateway start

# 3. 连接浏览器
openclaw browser --browser-profile openclaw start

# 4. 打开招聘网站
openclaw browser --browser-profile openclaw open https://www.zhipin.com/
## 十、当前进展

已完成：

OpenClaw 环境部署
浏览器自动化控制
招聘网站访问

待实现：

岗位信息自动提取
岗位匹配分析
自动填写简历（核心目标）
## 十一、经验总结
WSL 不适合直接运行 GUI 程序（浏览器）
浏览器自动化推荐使用 Windows + CDP
一定要做数据隔离（user-data-dir）
OpenClaw 更适合作为“自动化中枢”，而不是直接爬虫
