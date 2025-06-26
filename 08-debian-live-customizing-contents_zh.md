# Debian Live 手册

## 自定义内容

本章讨论了如何对实时系统内容进行精细调整，不仅仅是选择要包含的软件包。通过包含文件，您可以在实时系统映像中添加或替换任意文件；通过钩子，您可以在构建和启动时的不同阶段执行任意命令；通过预设，您可以在安装软件包时通过提供debconf问题的答案来配置软件包。

### 9.1 包含文件

虽然理想情况下，实时系统应完全由未修改的软件包提供文件，但有时通过文件提供或修改某些内容会很方便。使用包含文件，可以在实时系统映像中添加（或替换）任意文件。_live-build_ 提供了两种使用它们的机制：

- **Chroot本地包含**：允许您向chroot/Live文件系统添加或替换文件。请参阅Chroot本地包含以获取更多信息。
- **二进制本地包含**：允许您在二进制映像中添加或替换文件。请参阅二进制本地包含以获取更多信息。

请参阅术语以获取有关"Live"和"二进制"映像之间区别的更多信息。

#### 9.1.1 Chroot本地包含

Chroot本地包含可用于在chroot/Live文件系统中添加或替换文件，以便它们可以在Live系统中使用。典型用法是填充Live系统用于创建live用户主目录的骨架用户目录（/etc/skel）。另一个用法是提供可以简单添加或替换到映像中的配置文件；如果需要处理，请参阅Chroot本地钩子。

要包含文件，只需将它们添加到您的`config/includes.chroot`目录中。此目录对应于实时系统的根目录/。例如，要在实时系统中添加文件/var/www/index.html，请使用：

```bash
$ mkdir -p config/includes.chroot/var/www  
$ cp /path/to/my/index.html config/includes.chroot/var/www  
```

您的配置将具有以下布局：

```
-- config  
   |-- includes.chroot  
   |   `-- var  
   |       `-- www  
   |           `-- index.html  
```

Chroot本地包含在软件包安装后安装，以便软件包安装的文件被覆盖。

#### 9.1.2 二进制本地包含

要在介质文件系统中包含文档或视频等材料，以便在插入介质时无需启动Live系统即可立即访问，您可以使用二进制本地包含。这与Chroot本地包含的工作方式类似。例如，假设文件~/video_demo.*是Live系统的演示视频，由HTML索引页面描述和链接。只需将材料复制到`config/includes.binary/`，如下所示：

```bash
$ cp ~/video_demo.* config/includes.binary/  
```

这些文件现在将出现在Live介质的根目录中。

### 9.2 钩子

钩子允许在构建的chroot和二进制阶段运行命令以自定义映像。根据您是构建Live映像还是常规系统映像，您必须分别将钩子放在`config/hooks/live`或`config/hooks/normal`中。这些通常被称为本地钩子，因为它们在构建环境中执行。

还有启动时钩子，允许您在映像已构建后，在启动过程中运行命令。

#### 9.2.1 Chroot本地钩子

要在chroot阶段运行命令，请创建一个带有.hook.chroot后缀的钩子脚本，其中包含命令，放在`config/hooks/live`或`config/hooks/normal`目录中。钩子将在应用其余chroot配置后在chroot中运行，因此请记住确保您的配置包括钩子运行所需的所有软件包和文件。请参阅/usr/share/doc/live-build/examples/hooks中提供的示例chroot钩子脚本，以获取各种常见chroot自定义任务的示例，您可以复制或符号链接以在自己的配置中使用它们。

#### 9.2.2 二进制本地钩子

要在二进制阶段运行命令，请创建一个带有.hook.binary后缀的钩子脚本，其中包含命令，放在`config/hooks/live`或`config/hooks/normal`目录中。钩子将在所有其他二进制命令运行后运行，但在最后一个二进制命令binary_checksums之前运行。钩子中的命令不在chroot中运行，因此请注意不要修改构建树之外的任何文件，否则可能会损坏您的构建系统！请参阅/usr/share/doc/live-build/examples/hooks中提供的示例二进制钩子脚本，以获取各种常见二进制自定义任务的示例，您可以复制或符号链接以在自己的配置中使用它们。

#### 9.2.3 启动时钩子

要在启动时执行命令，您可以提供_live-config_钩子，如其手册页的"自定义"部分所述。检查/lib/live/config/中提供的_live-config_自己的钩子，注意序列号。然后提供一个带有适当序列号前缀的钩子，作为`config/includes.chroot/lib/live/config/`中的chroot本地包含，或作为安装修改或第三方软件包中讨论的自定义软件包。

### 9.3 预设Debconf问题

在`config/preseed/`目录中以.cfg后缀加上阶段（.chroot或.binary）的文件被视为debconf预设文件，并在相应阶段通过debconf-set-selections由_live-build_安装。

有关debconf的更多信息，请参阅_debconf_包中的debconf(7)。 