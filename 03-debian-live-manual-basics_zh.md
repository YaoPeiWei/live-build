# Debian Live 手册

# 基础知识

本章包含构建过程的简要概述以及使用三种最常用镜像类型的说明。最通用的镜像类型 iso-hybrid 可用于虚拟机、光学介质或 USB 便携存储设备。在某些特殊情况下，如后面所解释的，hdd 类型可能更合适。本章包括构建和使用 netboot 类型镜像的详细说明，由于服务器上所需的设置，这稍微复杂一些。对于不熟悉网络启动的人来说，这是一个稍微高级的主题，但这里包含它是因为一旦设置完成，它是测试和部署镜像以便在本地网络上启动的非常方便的方法，无需处理镜像介质的麻烦。

该章节最后快速介绍了 webbooting，这可能是使用不同镜像用于不同目的的最简单方法，可以根据需要使用互联网作为手段从一个镜像切换到另一个镜像。

在本章中，我们将经常引用 _live-build_ 产生的默认文件名。如果您下载的是预构建镜像，实际文件名可能会有所不同。

## 4.1 什么是 live 系统？

live 系统通常是指从可移动介质（如 CD-ROM 或 USB 闪存驱动器）或从网络启动到计算机上的操作系统，无需在常用驱动器上进行任何安装即可使用，运行时完成自动配置（参见术语）。

对于 live 系统，它是一个为支持的架构之一（目前是 amd64 和 arm64）构建的操作系统。它由以下部分组成：

* **Linux 内核镜像**，通常命名为 vmlinuz*
* **初始 RAM 磁盘镜像 (initrd)**：为 Linux 启动设置的 RAM 磁盘，包含可能需要的模块来挂载系统镜像以及执行此操作的一些脚本。
* **系统镜像**：操作系统的文件系统镜像。通常使用 SquashFS 压缩文件系统来最小化 live 系统镜像大小。请注意，它是只读的。因此，在启动期间，live 系统将使用 RAM 磁盘和 'union' 机制来启用运行系统内的文件写入。但是，除非使用可选的持久性（参见持久性），否则所有修改都将在关机时丢失。
* **引导加载程序**：为从选定介质启动而制作的一小段代码，可能呈现提示或菜单以允许选择选项/配置。它加载 Linux 内核及其 initrd 以运行相关的系统文件系统。根据目标介质和包含前面提到的组件的文件系统格式，可以使用不同的解决方案：isolinux 用于从 ISO9660 格式的 CD 或 DVD 启动，syslinux 用于从 VFAT 分区启动 HDD 或 USB 驱动器，extlinux 用于 ext2/3/4 和 btrfs 分区，pxelinux 用于 PXE 网络启动，GRUB 用于 ext2/3/4 分区等。

您可以使用 _live-build_ 从您的规范构建系统镜像，设置 Linux 内核、其 initrd 和引导加载程序来运行它们，所有这些都在一个依赖于介质的格式中（ISO9660 镜像、磁盘镜像等）。

## 4.2 下载预构建镜像

您可以从 ‹https://www.debian.org/CD/live/› 下载预构建镜像之一。为许多流行的桌面环境（GNOME、Xfce、KDE 等）准备了特定的 live 镜像。

如果您不确定要下载哪个文件，请使用 'stable' 发行版中的 'Live GNOME' 镜像。然后您可以跳过阅读下一节并在虚拟机中运行镜像。

## 4.3 第一步：构建 ISO hybrid 镜像

无论镜像类型如何，每次构建镜像时都需要执行相同的基本步骤。作为第一个示例，创建一个构建目录，切换到该目录，然后执行以下 _live-build_ 命令序列来创建一个包含默认 live 系统（无 X.org）的基本 ISO hybrid 镜像。它适合刻录到 CD 或 DVD 介质，也适合复制到 USB 闪存驱动器上。

工作目录的名称完全由您决定，但如果您查看 _live-manual_ 中使用的示例，为每个目录使用一个帮助您识别正在处理的镜像的名称是一个好主意，特别是如果您正在处理或试验不同的镜像类型。在这种情况下，您将构建一个默认系统，所以让我们将其称为，例如，live-default。

```bash
$ mkdir live-default && cd live-default
```

然后，运行 lb config 命令。这将在当前目录中创建一个 "config/" 层次结构供其他命令使用：

```bash
$ lb config
```

这些命令没有传递参数，因此将使用它们各种选项的默认值。有关更多详细信息，请参见 lb config 命令。

现在 "config/" 层次结构存在，使用 lb build 命令构建镜像：

```bash
# lb build
```

这个过程可能需要一段时间，具体取决于您计算机的速度和网络连接。完成后，当前目录中应该有一个 live-image-amd64.hybrid.iso 镜像文件，可以使用。

**注意：** 如果您在 amd64 系统上构建，生成的镜像名称将是 live-image-amd64.hybrid.iso。请在本手册中记住这个命名约定。

## 4.4 使用 ISO hybrid live 镜像

在构建或下载 ISO hybrid 镜像后，通常的下一步是准备启动介质，无论是 CD-R(W) 或 DVD-R(W) 光学介质还是 USB 闪存驱动器。

### 4.4.1 将 ISO 镜像刻录到物理介质

刻录 ISO 镜像很容易。只需安装 _xorriso_ 并从命令行使用它来刻录镜像。例如：

```bash
# apt-get install xorriso
$ xorriso -as cdrecord -v dev=/dev/sr0 blank=as_needed live-image-amd64.hybrid.iso
```

### 4.4.2 将 ISO hybrid 镜像复制到 USB 闪存驱动器

使用 xorriso 准备的 ISO 镜像可以简单地使用 cp 程序或等效程序复制到 USB 闪存驱动器。插入一个大小足够容纳镜像文件的 USB 闪存驱动器，并确定它是哪个设备，我们在此后将其称为 ${USBSTICK}。这是您密钥的设备文件，如 /dev/sdb，而不是分区，如 /dev/sdb1！您可以通过在插入闪存驱动器后查看 dmesg 的输出来找到正确的设备名称，或者更好的是，使用 ls -l /dev/disk/by-id。

一旦确定您有正确的设备名称，使用 cp 命令将镜像复制到闪存驱动器。**这肯定会覆盖您闪存驱动器上的任何先前内容！**

```bash
$ cp live-image-amd64.hybrid.iso ${USBSTICK}
$ sync
```

**注意：** _sync_ 命令有助于确保内核在复制镜像时存储在内存中的所有数据都写入 USB 闪存驱动器。

### 4.4.3 使用 USB 闪存驱动器上的剩余空间

将 live-image-amd64.hybrid.iso 复制到 USB 闪存驱动器后，设备上的第一个分区将被 live 系统填满。要使用剩余的可用空间，请使用分区工具（如 _gparted_ 或 _parted_）在闪存驱动器上创建新分区。

```bash
# gparted ${USBSTICK}
```

创建分区后，其中 ${PARTITION} 是分区的名称，如 /dev/sdb2，您必须在其上创建文件系统。一个可能的选择是 ext4。

```bash
# mkfs.ext4 ${PARTITION}
```

**注意：** 如果您想与 Windows 一起使用额外空间，显然该操作系统通常无法访问除第一个分区之外的任何分区。我们邮件列表上讨论了一些解决此问题的方法，但通常建议使用 FAT32 文件系统，因为 Windows 可以读取它。但是，请注意，FAT32 有 4GB 的文件大小限制，这可能不适合某些用途。

### 4.4.4 启动 live 介质

首次启动您的 live 介质时，无论是 CD、DVD、U 盘还是 PXE 启动，可能首先需要在您计算机的 BIOS 中进行一些设置。由于 BIOS 的功能和按键绑定差异很大，我们无法在此深入探讨该主题。有些 BIOS 提供一个按键，在启动时调出启动设备菜单，如果您的系统上有此功能，这是最简单的方法。否则，您需要进入 BIOS 配置菜单并更改启动顺序，将 live 系统的启动设备置于正常启动设备之前。

一旦您启动了介质，就会看到一个启动菜单。如果您直接按回车键，系统将使用默认条目 "Live" 和默认选项启动。有关启动选项的更多信息，请参阅菜单中的"help"条目以及 live 系统内的 live-boot 和 live-config 手册页。

假设您选择了 "Live" 并启动了默认的桌面 live 镜像，在启动消息滚动过后，您应该会自动登录到用户帐户并看到一个准备就绪的桌面。如果您启动的是纯控制台镜像，您应该会自动在控制台登录到用户帐户并看到一个准备就绪的 shell 提示符。

## 4.5 使用虚拟机进行测试

在虚拟机 (VM) 中运行 live 镜像可以极大地节省开发时间。但这并非没有需要注意的地方：

*   运行虚拟机需要足够的内存以同时支持客户机和主机操作系统，并推荐使用支持硬件虚拟化的 CPU。
*   在虚拟机中运行存在一些固有的局限性，例如，较差的视频性能、有限的模拟硬件选择。
*   当为特定硬件开发时，没有什么能替代在硬件本身上运行。
*   偶尔会出现仅与在虚拟机中运行相关的错误。如有疑问，请直接在硬件上测试您的镜像。
*   如果您可以在这些限制下工作，请考察可用的虚拟机软件，并选择一个适合您需求的。

### 4.5.1 使用 QEMU 测试 ISO 镜像

Debian 中功能最强大的虚拟机是 QEMU。如果您的处理器支持硬件虚拟化，请使用 qemu-kvm 软件包；qemu-kvm 软件包的描述简要列出了相关要求。

首先，如果您的处理器支持，请安装 qemu-kvm。如果不支持，则安装 qemu，在这种情况下，以下示例中的程序名将是 qemu 而不是 kvm。qemu-utils 软件包对于使用 qemu-img 创建虚拟磁盘镜像也很有价值。

```bash
# apt-get install qemu-kvm qemu-utils
```

启动 ISO 镜像很简单：

```bash
$ kvm -cdrom live-image-amd64.hybrid.iso -m 4G
```

有关更多详细信息，请参阅 man 手册页。

**注意：** 对于包含桌面环境并希望使用 qemu 进行测试的 live 系统，您可能希望在您的 live-build 配置中包含 spice-vdagent 软件包。这将自动调整分辨率并在虚拟机和主机之间启用剪贴板。

```bash
$ echo "spice-vdagent" >> config/package-lists/spice.list.chroot
```

### 4.5.2 使用 VirtualBox 测试 ISO 镜像

为了使用 VirtualBox 测试 ISO：

```bash
# apt-get install virtualbox virtualbox-qt virtualbox-dkms
$ virtualbox
```

创建一个新的虚拟机，更改存储设置以使用 live-image-amd64.hybrid.iso 作为 CD/DVD 设备，然后启动虚拟机。

**注意：** 对于包含 X.org 并希望使用 VirtualBox 进行测试的 live 系统，您可能希望在您的 live-build 配置中包含 VirtualBox X.org 驱动程序包 virtualbox-guest-dkms 和 virtualbox-guest-x11。否则，分辨率将限制为 800x600。

```bash
$ echo "virtualbox-guest-dkms virtualbox-guest-x11" >> config/package-lists/my.list.chroot
```

为了让 dkms 软件包正常工作，还需要安装镜像中使用的内核版本的内核头文件。与其在上面创建的软件包列表中手动列出正确的 linux-headers 软件包，不如让 live-build 自动选择正确的软件包。

```bash
  $ lb config --linux-packages "linux-image linux-headers"
```

## 4.6 构建和使用 HDD 镜像

构建 HDD 镜像在所有方面都与构建 ISO hybrid 镜像类似，只是您需要指定 -b hdd，并且生成的文件名是 live-image-amd64.img，该文件不能刻录到光学介质上。它适用于从 U 盘、USB 硬盘和各种其他便携式存储设备启动。通常，ISO hybrid 镜像也可以用于此目的，但是如果您的 BIOS 不能正确处理 hybrid 镜像，那么您就需要一个 HDD 镜像。

**注意：** 如果您使用前面的示例创建了 ISO hybrid 镜像，则需要使用 lb clean 命令清理您的工作目录（请参阅 lb clean 命令）：

```bash
# lb clean --binary
```

像以前一样运行 lb config 命令，只是这次指定 HDD 镜像类型：

```bash
$ lb config -b hdd
```

现在使用 lb build 命令构建镜像：

```bash
# lb build
```

构建完成后，当前目录中应该会有一个 live-image-amd64.img 文件。

生成的二进制镜像包含一个 VFAT 分区和 syslinux 引导加载程序，可以直接写入 USB 设备。再次强调，在 USB 上使用 HDD 镜像就像使用 ISO hybrid 镜像一样。请遵循"使用 ISO hybrid live 镜像"中的说明，只是使用文件名 live-image-amd64.img 而不是 live-image-amd64.hybrid.iso。

同样，要使用 Qemu 测试 HDD 镜像，请按照上面"使用 QEMU 测试 ISO 镜像"中的说明安装 qemu。然后根据您的主机系统需要运行 kvm 或 qemu，并将 live-image-amd64.img 指定为第一个硬盘驱动器。

```bash
$ kvm -hda live-image-amd64.img 
```

## 4.7 网络启动

网络启动允许客户端计算机通过网络启动 live 系统，而无需本地存储镜像。这需要服务器端的一些设置，但一旦配置完成，它提供了一种非常方便的方式来测试和部署镜像。

要构建网络启动镜像，您需要清理之前的构建并重新配置：

```bash
# lb clean
```

在这种情况下，lb clean --binary 不足以清理必要的阶段。原因是，在网络启动设置中，需要使用不同的 initramfs 配置，_live-build_ 在构建网络启动镜像时会自动执行此操作。由于 initramfs 创建属于 chroot 阶段，在现有构建目录中切换到网络启动意味着也要重建 chroot 阶段。因此，需要使用 lb clean（这将同时删除 chroot 阶段）。

按如下方式运行 lb config 命令来配置您的镜像进行网络启动：

```bash
$ lb config -b netboot --net-root-path "/srv/debian-live" --net-root-server "192.168.0.2"
```

与 ISO 和 HDD 镜像相比，网络启动本身不向客户端提供文件系统镜像，因此必须通过 NFS 提供文件。可以通过 lb config 选择不同的网络文件系统。--net-root-path 和 --net-root-server 选项分别指定启动时文件系统镜像所在的 NFS 服务器的位置和服务器。确保这些设置为适合您的网络和服务器的值。

现在使用 lb build 命令构建镜像：

```bash
# lb build
```

在网络启动中，客户端运行一个通常驻留在以太网卡 EPROM 中的小程序。该程序发送 DHCP 请求以获取 IP 地址和关于下一步做什么的信息。通常，下一步是通过 TFTP 协议获取更高级别的引导加载程序。这可能是 pxelinux、GRUB，甚至直接启动到像 Linux 这样的操作系统。

例如，如果您在 /srv/debian-live 目录中解压生成的 live-image-amd64.netboot.tar 存档，您将在 live/filesystem.squashfs 中找到文件系统镜像，在 tftpboot/ 中找到内核、initrd 和 pxelinux 引导加载程序。

我们现在必须在服务器上配置三个服务来启用网络启动：DHCP 服务器、TFTP 服务器和 NFS 服务器。

### 4.7.1 DHCP 服务器

我们必须配置网络的 DHCP 服务器，以确保为网络启动客户端系统提供 IP 地址，并通告 PXE 引导加载程序的位置。

以下是为 ISC DHCP 服务器 isc-dhcp-server 在 /etc/dhcp/dhcpd.conf 配置文件中编写的示例：

```bash
# /etc/dhcp/dhcpd.conf - isc-dhcp-server 的配置文件

ddns-update-style none;

option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

log-facility local7;

subnet 192.168.0.0 netmask 255.255.255.0 {
  range 192.168.0.1 192.168.0.254;
  filename "pxelinux.0";
  next-server 192.168.0.2;
  option subnet-mask 255.255.255.0;
  option broadcast-address 192.168.0.255;
  option routers 192.168.0.1;
}
```

### 4.7.2 TFTP 服务器

这为系统在运行时提供内核和初始 ramdisk。

您应该安装 _tftpd-hpa_ 软件包。它可以提供根目录（通常是 /srv/tftp）内包含的所有文件。要让它在 /srv/debian-live/tftpboot 内提供文件，请以 root 身份运行以下命令：

```bash
# dpkg-reconfigure -plow tftpd-hpa
```

并在询问时填写新的 tftp 服务器目录。

### 4.7.3 NFS 服务器

一旦客户端计算机下载并启动了 Linux 内核并加载了其 initrd，它将尝试通过 NFS 服务器挂载 Live 文件系统镜像。

您需要安装 _nfs-kernel-server_ 软件包。

然后，通过向 /etc/exports 添加如下行来通过 NFS 提供文件系统镜像：

```bash
/srv/debian-live *(ro,async,no_root_squash,no_subtree_check)
```

并使用以下命令告诉 NFS 服务器这个新的导出：

```bash
# exportfs -rv
```

设置这三个服务可能有点棘手。您可能需要一些耐心来让它们一起工作。有关更多信息，请参见 syslinux wiki 上的 ‹https://wiki.syslinux.org/wiki/index.php?title=PXELINUX› 或 Debian 安装程序手册的 TFTP 网络启动部分 ‹https://www.debian.org/releases/stable/amd64/ch04s05.en.html›。它们可能会有所帮助，因为它们的过程非常相似。

### 4.7.4 网络启动测试 HowTo

使用 _live-build_ 创建网络启动镜像很容易，但在物理机器上测试镜像可能非常耗时。

为了让我们的生活更轻松，我们可以使用虚拟化。

### 4.7.5 Qemu

* 安装 _qemu_、_bridge-utils_、_sudo_。

编辑 /etc/qemu-ifup：

```bash
#!/bin/sh
sudo -p "Password for $0:" /sbin/ifconfig $1 172.20.0.1
echo "Executing /etc/qemu-ifup"
echo "Bringing up $1 for bridged mode..."
sudo /sbin/ifconfig $1 0.0.0.0 promisc up
echo "Adding $1 to br0..."
sudo /usr/sbin/brctl addif br0 $1
sleep 2
```

获取或构建 grub-floppy-netboot。

使用 "-net nic,vlan=0 -net tap,vlan=0,ifname=tun0" 启动 qemu

## 4.8 网络启动

网络启动是使用互联网作为手段检索和启动 live 系统的便捷方法。网络启动的要求很少。一方面，您需要一个带有引导加载程序、初始 ramdisk 和内核的介质。另一方面，需要一个网络服务器来存储包含文件系统的 squashfs 文件。

### 4.8.1 获取网络启动文件

像往常一样，您可以自己构建镜像或使用预构建文件。使用预构建镜像对于进行初始测试很方便，直到可以微调自己的需求。如果您构建了 live 镜像，您将在构建目录的 binary/live/ 下找到网络启动所需的文件。这些文件称为 vmlinuz、initrd.img 和 filesystem.squashfs。

也可以从已存在的 iso 镜像中提取这些文件。为了实现这一点，按如下方式循环挂载镜像：

```bash
# mount -o loop image.iso /mnt
```

文件位于 live/ 目录下。在这种特定情况下，它将是 /mnt/live/。这种方法的缺点是需要 root 权限才能挂载镜像。但是，它的优点是易于编写脚本，因此易于自动化。

但是毫无疑问，从 iso 镜像中提取文件并同时上传到网络服务器的最简单方法是使用 midnight commander 或 _mc_。如果您安装了 _genisoimage_ 软件包，双窗格文件管理器允许您在一个窗格中浏览 iso 文件的内容，并通过 ftp 在另一个窗格中上传文件。尽管这种方法需要手动工作，但它不需要 root 权限。

### 4.8.2 启动网络启动镜像

虽然一些用户更喜欢使用虚拟化来测试网络启动，但我们在这里引用真实硬件来匹配以下可能的使用案例，这应该只被视为示例。

为了启动网络启动镜像，拥有上述组件就足够了，即在名为 live/ 的目录中的 usb 闪存驱动器中的 vmlinuz 和 initrd.img，并安装 syslinux 作为引导加载程序。然后从 usb 闪存驱动器启动并在启动选项中输入 fetch=URL/PATH/TO/FILE。_live-boot_ 将检索 squashfs 文件并将其存储到 ram 中。这样，可以使用下载的压缩文件系统作为常规 live 系统。例如：

```bash
append boot=live components fetch=http://192.168.2.50/images/webboot/filesystem.squashfs
```

**使用案例：** 您有一个网络服务器，其中存储了两个 squashfs 文件，一个包含完整桌面，例如 gnome，另一个是标准桌面。如果您需要一台机器的图形环境，可以插入 usb 闪存驱动器并网络启动 gnome 镜像。如果您需要第二种类型镜像中包含的工具之一，也许用于另一台机器，可以网络启动标准镜像。