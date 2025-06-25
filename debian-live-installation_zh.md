# Debian Live 手册

# 安装

## 3. 安装

### 3.1 系统要求

构建 Live 系统镜像对主机系统的要求很少：

*   超级用户（root）访问权限
*   最新版本的 `live-build`
*   符合 POSIX 标准的 shell，如 `bash` 或 `dash`
*   `debootstrap`
*   Linux 2.6 或更新版本
*   具有 `dev` 和 `exec` 权限的挂载点。

```bash
# mount <your_mount_point> -odev,exec,remount
```

请注意，使用 Debian 或基于 Debian 的发行版不是必需的 - `live-build` 几乎可以在任何满足上述要求的发行版上运行。

### 3.2 安装 live-build

您可以通过多种不同的方式安装 `live-build`：

*   从 Debian 仓库安装
*   从源代码安装
*   从快照安装

如果您使用的是 Debian，推荐的方式是通过 Debian 仓库安装 `live-build`。

#### 3.2.1 从 Debian 仓库安装

像安装任何其他软件包一样简单地安装 `live-build`：

```bash
# apt-get install live-build
```

#### 3.2.2 从源代码安装

`live-build` 使用 Git 版本控制系统开发。在基于 Debian 的系统上，这由 `git` 软件包提供。要检出最新代码，请执行：

```bash
$ git clone https://salsa.debian.org/live-team/live-build.git
```

您可以通过执行以下命令构建和安装自己的 Debian 软件包：

```bash
$ cd live-build
$ dpkg-buildpackage -b -uc -us
$ cd ..
```

现在安装您感兴趣的新构建的 .deb 文件，例如：

```bash
# dpkg -i live-build_4.0-1_all.deb
```

您也可以通过执行以下命令直接将 `live-build` 安装到您的系统：

```bash
# make install
```

并使用以下命令卸载：

```bash
# make uninstall
```

### 3.3 安装 live-boot 和 live-config

**注意：** 您不需要在系统上安装 `live-boot` 或 `live-config` 来创建自定义的 Live 系统。但是，这样做了也不会造成任何损害，并且对参考目的很有用。如果您只需要文档，现在可以分别安装 `live-boot-doc` 和 `live-config-doc` 软件包。

#### 3.3.1 从 Debian 仓库安装

`live-boot` 和 `live-config` 都可以从 Debian 仓库获得，如安装 live-build 中所述。

#### 3.3.2 从源代码安装

要使用来自 git 的最新源代码，您可以按照以下过程操作。请确保您熟悉"术语"中提到的术语。

*   检出 `live-boot` 和 `live-config` 源代码

```bash
$ git clone https://salsa.debian.org/live-team/live-boot.git
$ git clone https://salsa.debian.org/live-team/live-config.git
```

如果您的目的是从源代码构建这些软件包，请查阅 `live-boot` 和 `live-config` 手册页以了解自定义的详细信息。

*   构建 `live-boot` 和 `live-config` .deb 文件

您必须在目标发行版上构建，或在包含目标平台的 chroot 中构建：这意味着如果您的目标是 **trixie**，那么您应该针对 **trixie** 构建。

如果您需要为与构建系统不同的目标发行版构建 `live-boot`，请使用个人构建器，如 `pbuilder` 或 `sbuild`。例如，对于 **trixie** Live 镜像，在 **trixie** chroot 中构建 `live-boot`。如果您的目标发行版恰好与您的构建系统发行版匹配，您可以使用 dpkg-buildpackage（由 `dpkg-dev` 软件包提供）直接在构建系统上构建：

```bash
$ cd live-boot
$ dpkg-buildpackage -b -uc -us
$ cd ../live-config
$ dpkg-buildpackage -b -uc -us
```

*   使用适用的生成的 .deb 文件

由于 `live-boot` 和 `live-config` 是由 `live-build` 系统安装的，在主机系统中安装软件包是不够的：您应该将生成的 .deb 文件视为任何其他自定义软件包。由于您从源代码构建的目的可能是在官方发布之前在短期内测试新事物，请按照"安装修改的或第三方软件包"来临时在您的配置中包含相关文件。特别是，请注意两个软件包都分为通用部分、文档部分和一个或多个后端。包括通用部分、仅一个与您的配置匹配的后端，以及可选的文档。假设您在当前目录中构建 Live 镜像，并在上面的目录中为两个软件包的单个版本生成了所有 .deb 文件，这些 bash 命令将复制所有相关软件包，包括默认后端：

```bash
$ cp ../live-boot{_,-initramfs-tools,-doc}*.deb config/packages.chroot/
$ cp ../live-config{_,-sysvinit,-doc}*.deb config/packages.chroot/
``` 