# OpenClaw 半自动化求职助手 - 知识笔记

> **项目简介**：基于 WSL 环境 + Edge 浏览器 + CDP 协议，实现招聘平台自动化操作

---

## 📋 目录
1. [项目背景与目标](#项目背景与目标)
2. [技术架构](#技术架构)
3. [环境搭建](#环境搭建)
4. [核心方案：浏览器接管](#核心方案)
5. [关键问题与解决方案](#关键问题与解决方案)
6. [使用规范](#使用规范)
7. [项目总结与优化](#项目总结与优化)

---

## 项目背景与目标

### 📌 现状问题
在求职投递（Boss 直聘、智联招聘等平台）过程中：
- 招聘信息分散，人工获取效率低
- 简历投递流程重复，需反复填写个人信息
- 单次完整投递耗时约 **20~30 分钟**

### 🎯 项目目标
构建半自动化求职助手，核心能力：
- ✅ 自动打开招聘网站
- ✅ 辅助岗位信息智能分析
- ✅ 浏览器自动化控制（支持后续扩展自动投递功能）

---

## 技术架构

```
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
```

### 🔑 关键技术栈
| 组件 | 说明 |
|------|------|
| **WSL 2** | Linux 开发环境 |
| **Ubuntu 22.04** | 基础系统 |
| **Microsoft Edge** | 浏览器自动化载体 |
| **CDP 协议** | Chrome Debug Protocol - 浏览器控制协议 |
| **OpenClaw** | AI Agent 自动化控制核心 |

---

## 环境搭建

### 第一步：检查 WSL 环境

```bash
wsl -l -v
```

**要求**：Ubuntu-22.04 + WSL2

### 第二步：在 WSL 中安装 Microsoft Edge

```bash
# 下载微软软件源配置包
curl -sSL -O https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb

# 安装软件源
sudo dpkg -i packages-microsoft-prod.deb

# 更新软件源
sudo apt-get update

# 安装Edge浏览器
sudo apt-get install microsoft-edge-stable

# 验证安装
microsoft-edge --version
```

### 第三步：解决常见环境问题

#### 问题：Edge 无法安装
```
报错：E: Package 'microsoft-edge-stable' has no installation candidate
原因：软件源未正确更新
解决：重新执行上述安装步骤
```

#### 问题：中文乱码 / 方块字
```bash
sudo apt install fonts-noto-cjk
```

---

## 核心方案

### ⭐ 推荐方案：浏览器接管（Windows 本地浏览器 + CDP 协议）

**背景**：WSL 是无 GUI 环境，无法直接运行 Edge 图形化界面
**解决**：改用 Windows 本地浏览器 + CDP 协议实现控制

### 实现步骤

#### Step 1：Windows 端启动 Edge 调试模式

```cmd
msedge --remote-debugging-port=18800 --user-data-dir="C:\openclaw-profile"
```

| 参数 | 说明 |
|------|------|
| `--remote-debugging-port=18800` | CDP 调试端口（18800 为示例端口） |
| `--user-data-dir="C:\openclaw-profile"` | 创建独立浏览器配置，避免污染主浏览器数据 |

#### Step 2：修改 OpenClaw 配置文件

```bash
nano ~/.openclaw/openclaw.json
```

添加配置：
```json
{
  "browser": {
    "cdpUrl": "http://127.0.0.1:18800"
  }
}
```

#### Step 3：启动 OpenClaw 网关服务

```bash
# 启动网关
openclaw gateway start

# 查看状态
openclaw gateway status
```

#### Step 4：启动浏览器控制

```bash
openclaw browser --browser-profile openclaw start
```

#### Step 5：自动打开招聘网站

```bash
openclaw browser --browser-profile openclaw open https://www.zhipin.com/
```

### ✅ 运行效果验证

- ✅ 自动打开 Boss 直聘 / 智联招聘等平台
- ✅ 中文字体正常显示，无乱码
- ✅ 浏览器完全受 OpenClaw 控制
- ✅ 控制台可正常下发自动化指令

---

## 关键问题与解决方案

### 问题 1：OpenClaw 启动浏览器失败

**报错**：`Missing X server or $DISPLAY`

**原因**：WSL 为无 GUI 环境，无法直接运行图形化浏览器

**解决方案**：改用 Windows 本地浏览器 + CDP 协议实现控制

---

### 问题 2：Gateway 端口占用

**报错**：`Port 18789 is already in use`

**解决**：
```bash
openclaw gateway stop
openclaw gateway start
```

---

### 问题 3：浏览器数据污染

**原因**：使用默认 Edge 用户配置

**解决**：启动时添加独立数据目录
```cmd
msedge --remote-debugging-port=18800 --user-data-dir="C:\openclaw-profile"
```

---

### 问题 4：插件警告

**提示**：`plugins.allow is empty`

**说明**：仅提示信息，不影响核心功能

**可选优化**：
```json
"plugins": {
  "allow": ["openclaw-weixin"]
}
```

---

## 使用规范

### 📍 启动顺序

1. **Windows 端**：启动 Edge 调试模式
   ```cmd
   msedge --remote-debugging-port=18800 --user-data-dir="C:\openclaw-profile"
   ```

2. **WSL 端**：启动 OpenClaw Gateway
   ```bash
   openclaw gateway start
   ```

3. **WSL 端**：启动浏览器控制
   ```bash
   openclaw browser --browser-profile openclaw start
   ```

4. **WSL 端**：执行网页打开指令
   ```bash
   openclaw browser --browser-profile openclaw open https://www.zhipin.com/
   ```

### 🔴 关闭建议

**优雅关闭**：
```bash
openclaw browser stop
openclaw gateway stop
```

**说明**：直接关闭终端也可，无严重影响

---

## 项目总结与优化

### ✅ 已完成的核心功能

- ✅ OpenClaw 在 WSL 环境完整部署
- ✅ 基于 CDP 协议实现浏览器自动化控制
- ✅ 成功实现招聘网站自动打开
- ✅ 完成多类环境异常问题排查与解决

### 🔄 后续优化方向

| 优化方向 | 描述 |
|---------|------|
| **岗位自动筛选** | 根据求职者条件智能筛选匹配岗位 |
| **信息自动提取** | 自动提取岗位核心信息（薪资、要求等） |
| **自动投递功能** | 完整的简历自动投递流程 |
| **AI 大模型集成** | 接入大模型分析岗位-简历匹配度 |

### 💡 经验总结

| 经验 | 说明 |
|------|------|
| **WSL 限制** | WSL 环境不适合直接运行 GUI 程序（如浏览器） |
| **最优方案** | 浏览器自动化最优方案：Windows 本地浏览器 + CDP 协议 |
| **环境隔离** | 必须做浏览器环境隔离（`user-data-dir`） |
| **工具定位** | OpenClaw 定位：自动化控制中枢，而非单纯的爬虫工具 |

---

## 📚 快速参考

### 常用命令速查表

```bash
# 检查 WSL 版本
wsl -l -v

# 安装 Edge
sudo apt-get update && sudo apt-get install microsoft-edge-stable

# 启动 OpenClaw Gateway
openclaw gateway start

# 启动浏览器控制
openclaw browser --browser-profile openclaw start

# 打开网页
openclaw browser --browser-profile openclaw open <URL>

# 关闭浏览器控制
openclaw browser stop

# 关闭 Gateway
openclaw gateway stop
```

---

**最后更新**：2026年4月14日
**笔记类型**：技术方案与最佳实践
**适用人群**：AI Agent 开发者、求职自动化研究者
