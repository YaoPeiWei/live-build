# Debian Live 手册

## 工具概述

本章包含了构建实时系统时使用的三个主要工具的概述：_live-build_、_live-boot_ 和 _live-config_。

### 5.1 live-build 包

_live-build_ 是一组用于构建实时系统的脚本。这些脚本也被称为"命令"。

_live-build_ 的理念是成为一个框架，使用配置目录来完全自动化和自定义构建实时映像的各个方面。

许多概念与使用 _debhelper_ 构建 Debian 包的概念相似：

- 脚本有一个用于配置其操作的中心位置。在 _debhelper_ 中，这是包树的 debian/ 子目录。例如，dh_install 将查找名为 debian/install 的文件，以确定哪些文件应该存在于特定的二进制包中。同样，_live-build_ 将其配置完全存储在 config/ 子目录下。
- 脚本是独立的——也就是说，始终可以安全地运行每个命令。

与 _debhelper_ 不同，_live-build_ 提供了生成骨架配置目录的工具。这可以被认为类似于 _dh-make_ 等工具。有关这些工具的更多信息，请继续阅读，因为本节的其余部分讨论了四个最重要的命令。请注意，前面的 lb 是 _live-build_ 命令的通用包装器。

- **lb config**：负责初始化实时系统配置目录。有关更多信息，请参见 lb config 命令。
- **lb build**：负责启动实时系统构建。有关更多信息，请参见 lb build 命令。
- **lb clean**：负责删除实时系统构建的部分内容。有关更多信息，请参见 lb clean 命令。

#### 5.1.1 lb config 命令

如 live-build 中所述，组成 _live-build_ 的脚本使用 source 命令从名为 config/ 的单个目录中读取其配置。由于手动构建此目录既耗时又容易出错，因此可以使用 lb config 命令创建初始骨架配置树。

不带任何参数发出 lb config 命令会创建 config/ 子目录，该子目录中填充了一些配置文件中的默认设置，以及两个名为 auto/ 和 local/ 的骨架树。

```
$ lb config
[2025-02-15 12:34:56] lb config
P: 使用 http 代理: http://127.0.0.1:3142
P: 为 debian/testing/amd64 系统创建配置树
P: 符号链接钩子...
```

不带任何参数使用 lb config 适用于需要非常基本映像的用户，或打算稍后通过 auto/config 提供更完整配置的用户（有关详细信息，请参见管理配置）。

通常，您会希望指定一些选项。例如，指定在构建映像时使用哪个包管理器：

```
$ lb config --apt aptitude
```

可以指定许多选项，例如：

```
$ lb config --binary-images netboot --bootappend-live "boot=live components hostname=live-host username=live-user" ...
```

完整的选项列表可在 lb_config 手册页中找到。

#### 5.1.2 lb build 命令

lb build 命令从 config/ 目录中读取您的配置。然后运行构建实时系统所需的低级命令。

#### 5.1.3 lb clean 命令

lb clean 命令的工作是删除构建的各个部分，以便后续构建可以从干净状态开始。默认情况下，chroot、binary 和 source 阶段会被清理，但缓存会保留。还可以清理单个阶段。例如，如果您进行了仅影响二进制阶段的更改，请在构建新二进制文件之前使用 lb clean --binary。如果您的更改使引导和/或包缓存无效，例如对 --mode、--architecture 或 --bootstrap 的更改，您必须使用 lb clean --purge。有关完整的选项列表，请参见 lb_clean 手册页。

### 5.2 live-boot 包

_live-boot_ 是一组为 _initramfs-tools_ 提供钩子的脚本，用于生成能够引导实时系统的 initramfs，例如由 _live-build_ 创建的系统。这包括实时系统 ISO、netboot tarballs 和 USB 记忆棒映像。

在启动时，它将查找包含 /live/ 目录的只读媒体，其中存储了根文件系统（通常是压缩文件系统映像，如 squashfs）。如果找到，它将使用 OverlayFS 创建一个可写环境，以便 Debian 类系统可以从中引导。

有关 Debian 中初始 ramfs 的更多信息，请参见 Debian Linux 内核手册中的 initramfs 章节。

### 5.3 live-config 包

_live-config_ 由在 _live-boot_ 之后启动时运行的脚本组成，用于自动配置实时系统。它处理诸如设置主机名、区域设置和时区、创建实时用户、抑制 cron 作业以及执行实时用户自动登录等任务。 