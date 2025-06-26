# Debian Live 手册

## 自定义运行时行为

所有在运行时完成的配置都是通过 _live-config_ 完成的。以下是用户感兴趣的一些常见 _live-config_ 选项。完整的选项列表可以在 _live-config_ 的手册页中找到。

### 10.1 自定义 live 用户

一个重要的考虑是，live 用户是在启动时由 _live-boot_ 创建的，而不是在构建时由 _live-build_ 创建的。这不仅影响到与 live 用户相关的材料在构建中的引入位置，还影响到与 live 用户相关的组和权限。

您可以通过使用 _live-config_ 的任何配置选项来指定 live 用户将属于的附加组。例如，要将 live 用户添加到 fuse 组，您可以在 `config/includes.chroot/etc/live/config.conf.d/10-user-setup.conf` 中添加以下文件：

```
LIVE_USER_DEFAULT_GROUPS="audio cdrom dip floppy video plugdev netdev powerdev scanner bluetooth fuse"
```

或者使用 `live-config.user-default-groups=audio,cdrom,dip,floppy,video,plugdev,netdev,powerdev,scanner,bluetooth,fuse` 作为启动参数。

也可以更改默认用户名 "user" 和默认密码 "live"。如果您出于任何原因想要这样做，可以轻松实现：

要更改默认用户名，您可以简单地在配置中指定它：

```
$ lb config --bootappend-live "boot=live components username=live-user"
```

更改默认密码的一种可能方法是通过启动时钩子来实现，如启动时钩子中所述。为此，您可以使用 `/usr/share/doc/live-config/examples/hooks` 中的 "passwd" 钩子，适当地为其添加前缀（例如 2000-passwd），并将其添加到 `config/includes.chroot/lib/live/config/` 中。

### 10.2 自定义语言环境和语言

当 live 系统启动时，语言涉及两个步骤：

- 语言环境生成
- 设置键盘配置

构建 Live 系统时的默认语言环境是 `locales=en_US.UTF-8`。要定义应生成的语言环境，请在 `lb config` 的 `--bootappend-live` 选项中使用 `locales` 参数，例如：

```
$ lb config --bootappend-live "boot=live components locales=de_CH.UTF-8"
```

可以将多个语言环境指定为逗号分隔的列表。

此参数以及下面指示的键盘配置参数也可以在内核命令行中使用。您可以通过 `language_country` 指定语言环境（在这种情况下使用默认编码）或完整的 `language_country.encoding` 单词。支持的语言环境列表及其编码可以在 `/usr/share/i18n/SUPPORTED` 中找到。

控制台和 X 键盘配置均由 _live-config_ 使用 `console-setup` 包执行。要配置它们，请通过 `--bootappend-live` 选项使用 `keyboard-layouts`、`keyboard-variants`、`keyboard-options` 和 `keyboard-model` 启动参数。这些的有效选项可以在 `/usr/share/X11/xkb/rules/base.lst` 中找到。要查找给定语言的布局和变体，请尝试搜索语言的英文名称和/或该语言所在国家，例如：

```
$ egrep -i '(^!|german.*switzerland)' /usr/share/X11/xkb/rules/base.lst
```

请注意，每个变体在描述中列出了其适用的布局。

通常，只需配置布局即可。例如，要在 X 中获取德语和瑞士德语键盘布局的语言环境文件，请使用：

```
$ lb config --bootappend-live "boot=live components locales=de_CH.UTF-8 keyboard-layouts=ch"
```

然而，对于非常具体的用例，您可能希望包括其他参数。例如，要在 TypeMatrix EZ-Reach 2030 USB 键盘上设置法语系统和法语-Dvorak 布局（称为 Bepo），请使用：

```
$ lb config --bootappend-live \\
 "boot=live components locales=fr_FR.UTF-8 keyboard-layouts=fr keyboard-variants=bepo keyboard-model=tm2030usb"
```

对于每个 `keyboard-*` 选项，可以将多个值指定为逗号分隔的列表，但 `keyboard-model` 只能接受一个值。请参阅 `keyboard(5)` 手册页以获取 XKBMODEL、XKBLAYOUT、XKBVARIANT 和 XKBOPTIONS 变量的详细信息和示例。如果给出了多个 `keyboard-variants` 值，它们将与 `keyboard-layouts` 值一一匹配（请参阅 `setxkbmap(1)` 的 `-variant` 选项）。允许空值；例如，要定义两个布局，默认是美国 QWERTY，另一个是美国 Dvorak，请使用：

```
$ lb config --bootappend-live \\
 "boot=live components keyboard-layouts=us,us keyboard-variants=,dvorak"
```

### 10.3 持久性

Live CD 范式是一个预安装的系统，它从只读介质（如 CD-ROM）运行，其中的写入和修改在运行它的主机硬件重启后不会保留。

Live 系统是这种范式的推广，因此支持除 CD 以外的其他介质；但在其默认行为中，它应被视为只读，系统的所有运行时演变在关机时都会丢失。

"持久性"是用于跨重启保存系统部分或全部运行时演变的不同解决方案的通用名称。要了解其工作原理，了解即使系统从只读介质启动和运行，文件和目录的修改也会写入可写介质（通常是 RAM 磁盘（tmpfs））是很有帮助的，而 RAM 磁盘的数据在重启后不会保留。

存储在此 RAM 磁盘上的数据应保存在可写的持久介质上，如本地存储介质、网络共享甚至多会话（重）写入 CD/DVD 的会话。所有这些介质在 Live 系统中以不同方式支持，除了最后一个之外，所有这些都需要在启动时指定特殊的启动参数：持久性。

如果设置了启动参数持久性（并且未设置 nopersistence），则在启动期间将探测本地存储介质（例如硬盘、USB 驱动器）以查找持久性卷。可以通过指定 _live-boot_(7) 手册页中描述的某些启动参数来限制要使用的持久性卷类型。持久性卷可以是以下任何一种：

- 通过其 GPT 名称标识的分区。
- 通过其文件系统标签标识的文件系统。
- 位于任何可读文件系统（甚至是外部操作系统的 NTFS 分区）根目录中的图像文件，通过其文件名标识。

覆盖的卷标签必须是持久性，但除非其根目录中包含名为 `persistence.conf` 的文件，否则将被忽略，该文件用于完全自定义卷的持久性，也就是说，指定您希望在重启后保存在持久性卷中的目录。有关更多详细信息，请参阅 `persistence.conf` 文件。

以下是如何准备用于持久性的卷的一些示例。它可以是硬盘或 USB 密钥上的 ext4 分区，例如：

```
# mkfs.ext4 -L persistence /dev/sdb1
```

另请参阅使用 USB 密钥上剩余的空间。

如果您的设备上已经有一个分区，您可以通过以下命令之一更改标签：

```
# tune2fs -L persistence /dev/sdb1 # 适用于 ext2,3,4 文件系统
```

以下是如何创建用于持久性的基于 ext4 的图像文件的示例：

```
$ dd if=/dev/null of=persistence bs=1 count=0 seek=1G # 用于 1GB 大小的图像文件
$ /sbin/mkfs.ext4 -F persistence
```

一旦创建了图像文件，例如，要使 /usr 持久化但仅保存您对该目录所做的更改而不是 /usr 的所有内容，您可以使用 "union" 选项。如果图像文件位于您的主目录中，请将其复制到硬盘文件系统的根目录并挂载到 /mnt，如下所示：

```
# cp persistence /
# mount -t ext4 /persistence /mnt
```

然后，创建 `persistence.conf` 文件添加内容并卸载图像文件。

```
# echo "/usr union" >> /mnt/persistence.conf
# umount /mnt
```

现在，使用 "persistence" 启动参数重启到您的 Live 介质中。

#### 10.3.1 persistence.conf 文件

带有持久性标签的卷必须通过 `persistence.conf` 文件进行配置，以使任意目录持久化。该文件位于卷的文件系统根目录中，控制其使哪些目录持久化，以及以何种方式持久化。

如何配置自定义覆盖挂载在 `persistence.conf(5)` 手册页中有详细描述，但一个简单的示例应该足以满足大多数用途。假设我们要在 /dev/sdb1 分区上的 ext4 文件系统中使我们的主目录和 APT 缓存持久化：

```
# mkfs.ext4 -L persistence /dev/sdb1
# mount -t ext4 /dev/sdb1 /mnt
# echo "/home" >> /mnt/persistence.conf
# echo "/var/cache/apt" >> /mnt/persistence.conf
# umount /mnt
```

然后我们重启。在第一次启动期间，/home 和 /var/cache/apt 的内容将被复制到持久性卷中，从那时起对这些目录的所有更改都将保存在持久性卷中。请注意，`persistence.conf` 文件中列出的任何路径都不能包含空格或特殊的 . 和 .. 路径组件。此外，/lib、/lib/live（或其任何子目录）或 / 不能使用自定义挂载进行持久化。作为此限制的解决方法，您可以将 / union 添加到 `persistence.conf` 文件中以实现完全持久化。

#### 10.3.2 使用多个持久性存储

对于不同的用例，有不同的方法可以使用多个持久性存储。例如，同时使用多个卷或在各种卷中仅选择一个用于非常具体的目的。

可以同时使用多个不同的自定义覆盖卷（每个卷都有自己的 `persistence.conf` 文件），但如果多个卷使同一目录持久化，则只会使用其中一个。如果任何两个挂载是"嵌套的"（即一个是另一个的子目录），则父目录将在子目录之前挂载，因此没有挂载会被另一个隐藏。如果它们列在同一个 `persistence.conf` 文件中，嵌套的自定义挂载是有问题的。如果您确实需要，请参阅 `persistence.conf(5)` 手册页以了解如何处理这种情况（提示：通常不需要）。

一个可能的用例：如果您希望将用户数据（即 /home）和超级用户数据（即 /root）存储在不同的分区中，请创建两个带有持久性标签的分区，并在每个分区中添加一个 `persistence.conf` 文件，如下所示，

```
# echo "/home" > persistence.conf
```

用于保存用户文件的第一个分区和

```
# echo "/root" > persistence.conf
```

用于存储超级用户文件的第二个分区。最后，使用持久性启动参数。

如果用户需要相同类型的多个持久性存储用于不同位置或测试，例如私人和工作，持久性标签启动参数与持久性启动参数结合使用将允许多个但唯一的持久性介质。例如，如果用户想要使用标记为私人的数据持久性分区来存储个人数据，如浏览器书签或其他类型，他们将使用启动参数：`persistence persistence-label=private`。而要存储与工作相关的数据，如文档、研究项目或其他类型，他们将使用启动参数：`persistence persistence-label=work`。

请记住，这些卷中的每一个，私人和工作，也需要在其根目录中有一个 `persistence.conf` 文件。_live-boot_ 手册页包含有关如何使用这些标签与旧名称的更多信息。

#### 10.3.3 使用加密的持久性

使用持久性功能意味着某些敏感数据可能会面临风险。特别是如果持久性数据存储在便携设备上，如 USB 密钥或外部硬盘驱动器上。这时加密就派上用场了。即使整个过程可能看起来很复杂，因为需要采取的步骤很多，但使用 _live-boot_ 处理加密分区实际上很容易。为了使用 **luks**，这是支持的加密类型，您需要在创建加密分区的机器上和您将要使用加密持久性分区的 live 系统中安装 _cryptsetup_。

要在您的机器上安装 _cryptsetup_：

```
# apt-get install cryptsetup
```

要在您的 live 系统中安装 _cryptsetup_，请将其添加到您的包列表中：

```
$ lb config
$ echo "cryptsetup cryptsetup-initramfs" > config/package-lists/encryption.list.chroot
```

一旦您有了带有 _cryptsetup_ 的 live 系统，您基本上只需要创建一个新分区，加密它并使用持久性和 `persistence-encryption=luks` 参数启动。我们本可以提前预见到这一步，并按照通常的程序添加启动参数：

```
$ lb config --bootappend-live "boot=live components persistence persistence-encryption=luks"
```

让我们详细了解一下对于那些不熟悉加密的人。在以下示例中，我们将使用一个 USB 密钥上的分区，该分区对应于 /dev/sdc2。请注意，您需要确定在您的具体情况下要使用哪个分区。

第一步是插入您的 USB 密钥并确定它是哪个设备。_live-manual_ 中推荐的列出设备的方法是使用 `ls -l /dev/disk/by-id`。之后，创建一个新分区，然后使用密码短语加密它，如下所示：

```
# cryptsetup --verify-passphrase luksFormat /dev/sdc2
```

然后在虚拟设备映射器中打开 luks 分区。使用您喜欢的任何名称。我们在这里使用 **live** 作为示例：

```
# cryptsetup luksOpen /dev/sdc2 live
```

下一步是在创建文件系统之前用零填充设备：

```
# dd if=/dev/zero of=/dev/mapper/live
```

现在，我们准备创建文件系统。请注意，我们正在添加标签持久性，以便设备在启动时作为持久性存储挂载。

```
# mkfs.ext4 -L persistence /dev/mapper/live
```

要继续我们的设置，我们需要挂载设备，例如在 /mnt 中。

```
# mount /dev/mapper/live /mnt
```

并在分区的根目录中创建 `persistence.conf` 文件。这是，如前所述，严格必要的。请参阅 `persistence.conf` 文件。

```
# echo "/ union" > /mnt/persistence.conf
```

然后卸载挂载点：

```
# umount /mnt
```

可选地，尽管这可能是保护我们刚刚添加到分区的数据的好方法，我们可以关闭设备：

```
# cryptsetup luksClose live
```

让我们总结一下这个过程。到目前为止，我们已经创建了一个支持加密的 live 系统，可以按照在 USB 密钥上复制 ISO 混合映像中所述复制到 USB 密钥上。我们还创建了一个加密分区，可以位于同一个 USB 密钥中以便携带，并且我们已将加密分区配置为用作持久性存储。因此，现在我们只需要启动 live 系统。在启动时，_live-boot_ 将提示我们输入密码短语，并将挂载加密分区以用作持久性。 