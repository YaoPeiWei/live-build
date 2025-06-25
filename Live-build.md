# Debian Live Build 手册

## 目录
1. [简介](#简介)
2. [安装](#安装)
3. [基本概念](#基本概念)
4. [配置](#配置)
5. [构建过程](#构建过程)
6. [高级配置](#高级配置)
7. [故障排除](#故障排除)
8. [示例](#示例)

## 简介

Debian Live Build 是一套用于创建 Debian Live 系统的工具集。Live 系统是指可以从可移动介质（如 USB 闪存驱动器、CD/DVD）启动的完整 Debian 系统，无需安装到硬盘上。

### 主要特性

- **模块化设计**: 使用配置文件来定制 Live 系统
- **多种输出格式**: 支持 ISO、USB、网络启动等多种格式
- **高度可定制**: 可以添加自定义软件包、配置文件和脚本
- **自动化构建**: 支持自动化构建流程

### 系统要求

- Debian 或基于 Debian 的系统
- 至少 2GB 可用磁盘空间
- 网络连接（用于下载软件包）

## 安装

### 安装 live-build 工具

```bash
# 更新软件包列表
sudo apt update

# 安装 live-build
sudo apt install live-build
```

### 安装可选依赖

```bash
# 安装额外的构建工具
sudo apt install live-config live-boot live-manual-html
```

## 基本概念

### 目录结构

live-build 使用以下目录结构：

```
live-build/
├── auto/
│   ├── config/
│   └── scripts/
├── config/
│   ├── bootstrap/
│   ├── chroot/
│   ├── common/
│   ├── hooks/
│   └── package-lists/
└── local/
```

### 配置文件类型

1. **bootstrap**: 定义基础系统的来源
2. **chroot**: 配置目标系统的软件包和设置
3. **common**: 通用配置选项
4. **hooks**: 自定义脚本和钩子
5. **package-lists**: 软件包列表管理

## 配置

### 基本配置

创建基本配置文件：

```bash
# 创建新的 live-build 项目
mkdir my-live-system
cd my-live-system

# 初始化配置
lb config
```

### 配置文件选项

#### 1. 系统架构

```bash
# 设置目标架构
lb config --architectures amd64
```

#### 2. 发行版

```bash
# 设置 Debian 发行版
lb config --distribution bookworm
```

#### 3. 桌面环境

```bash
# 设置桌面环境
lb config --desktop gnome
```

#### 4. 镜像源

```bash
# 设置软件源
lb config --mirror-bootstrap http://deb.debian.org/debian/
lb config --mirror-chroot http://deb.debian.org/debian/
```

### 高级配置

#### 自定义软件包

创建 `config/package-lists/desktop.list.chroot`：

```
# 桌面环境软件包
task-gnome-desktop
firefox-esr
libreoffice
```

#### 自定义脚本

创建 `config/hooks/` 目录下的脚本：

```bash
#!/bin/bash
# config/hooks/0100-custom-script.chroot

# 自定义配置
echo "Custom configuration applied"
```

## 构建过程

### 构建阶段

live-build 的构建过程分为以下几个阶段：

1. **bootstrap**: 创建基础系统
2. **chroot**: 在 chroot 环境中安装软件包
3. **binary**: 创建最终的 Live 系统镜像

### 开始构建

```bash
# 完整构建
sudo lb build

# 仅构建特定阶段
sudo lb bootstrap
sudo lb chroot
sudo lb binary
```

### 构建选项

```bash
# 详细输出
sudo lb build --verbose

# 并行构建
sudo lb build --parallel

# 清理构建
sudo lb clean
```

## 高级配置

### 自定义内核

```bash
# 配置自定义内核
lb config --linux-packages linux-image
```

### 网络配置

```bash
# 配置网络设置
lb config --net-root-filesystem nfs
```

### 用户配置

```bash
# 设置默认用户
lb config --user live
lb config --user-fullname "Live User"
```

### 本地化

```bash
# 设置语言和地区
lb config --language zh-CN
lb config --locale zh_CN.UTF-8
```

## 故障排除

### 常见问题

#### 1. 构建失败

```bash
# 检查错误日志
tail -f /var/log/live/build.log

# 清理并重新构建
sudo lb clean
sudo lb build
```

#### 2. 软件包冲突

```bash
# 检查软件包依赖
apt-cache policy package-name

# 解决冲突
apt-cache depends package-name
```

#### 3. 磁盘空间不足

```bash
# 检查可用空间
df -h

# 清理临时文件
sudo lb clean
```

### 调试技巧

1. **使用详细模式**: `--verbose` 选项
2. **检查配置文件**: 验证配置文件的语法
3. **查看日志**: 检查构建日志文件
4. **分步构建**: 逐步执行构建过程

## 示例

### 基本 GNOME Live 系统

```bash
# 创建项目目录
mkdir gnome-live
cd gnome-live

# 配置基本系统
lb config \
  --architectures amd64 \
  --distribution bookworm \
  --desktop gnome \
  --mirror-bootstrap http://deb.debian.org/debian/ \
  --mirror-chroot http://deb.debian.org/debian/

# 构建系统
sudo lb build
```

### 自定义开发环境

```bash
# 配置开发环境
lb config \
  --architectures amd64 \
  --distribution bookworm \
  --desktop xfce \
  --packages "build-essential git vim"

# 添加自定义软件包
echo "build-essential" > config/package-lists/development.list.chroot
echo "git" >> config/package-lists/development.list.chroot
echo "vim" >> config/package-lists/development.list.chroot

# 构建
sudo lb build
```

### 网络启动系统

```bash
# 配置网络启动
lb config \
  --architectures amd64 \
  --distribution bookworm \
  --net-root-filesystem nfs \
  --net-root-path /srv/nfs/live

# 构建网络启动镜像
sudo lb build
```

## 最佳实践

### 1. 版本控制

- 将配置文件纳入版本控制
- 使用标签标记不同版本
- 记录配置变更

### 2. 自动化

- 使用脚本自动化构建过程
- 集成到 CI/CD 流程
- 定期更新基础镜像

### 3. 测试

- 在多种硬件上测试
- 验证所有功能正常工作
- 进行性能测试

### 4. 文档

- 记录自定义配置
- 维护构建说明
- 记录已知问题

## 参考资源

- [官方文档](https://live-team.pages.debian.net/live-manual/)
- [Debian Wiki](https://wiki.debian.org/DebianLive)
- [GitHub 仓库](https://salsa.debian.org/live-team/live-build)
- [邮件列表](https://lists.debian.org/debian-live/)

## 许可证

本手册基于 Debian Live Build 官方文档翻译，遵循相应的开源许可证。

---

*最后更新: 2024年*
