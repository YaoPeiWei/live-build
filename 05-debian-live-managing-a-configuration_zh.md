# Debian Live 手册

## 管理配置

本章解释了如何从初始创建到后续修订以及 _live-build_ 软件和实时映像本身的连续发布来管理实时配置。

### 6.1 处理配置更改

实时配置很少在第一次尝试时就完美无缺。可以通过命令行传递 lb config 选项来执行单次构建，但更典型的做法是反复修改这些选项并重新构建，直到满意为止。为了支持这些更改，您需要自动脚本来确保配置保持一致。

#### 6.1.1 为什么使用自动脚本？它们的作用是什么？

lb config 命令将您传递给它的选项存储在 config/* 文件中，并设置许多其他选项为默认值。如果您再次运行 lb config，它不会重置基于初始选项默认的任何选项。因此，例如，如果您再次运行 lb config 并为 \--binary-images 提供新值，则为旧映像类型默认的任何依赖选项可能不再适用于新选项。这些文件也不是为了阅读或编辑而设计的。它们存储了超过一百个选项的值，因此没有人，包括您自己，能够看到您实际指定了哪些选项。最后，如果您运行 lb config，然后升级 _live-build_ 并且它恰好重命名了一个选项，config/* 仍将包含以旧选项命名的变量，这些变量不再有效。

出于所有这些原因，auto/* 脚本将使您的生活更轻松。它们是 lb config、lb build 和 lb clean 命令的简单包装器，旨在帮助您管理配置。auto/config 脚本存储您的 lb config 命令及其所有所需选项，auto/clean 脚本删除包含配置变量值的文件，而 auto/build 脚本则为每次构建保留一个 build.log 日志。每次运行相应的 lb 命令时，这些脚本都会自动运行。通过使用这些脚本，您的配置更易于阅读，并且在每次修订时保持内部一致。此外，当您在阅读更新的文档后升级 _live-build_ 时，识别和修复需要更改的选项将变得更加容易。

#### 6.1.2 使用示例自动脚本

为了方便起见，_live-build_ 附带了示例自动 shell 脚本供复制和编辑。启动一个新的默认配置，然后将示例复制到其中：

```
$ mkdir mylive && cd mylive && lb config
$ mkdir auto
$ cp /usr/share/doc/live-build/examples/auto/* auto/
```

编辑 auto/config，添加您认为合适的任何选项。例如：

```sh
#!/bin/sh
lb config noauto \\
 --distribution stable \\
 --binary-images hdd \\
 --mirror-bootstrap http://ftp.ch.debian.org/debian/ \\
 --mirror-binary http://ftp.ch.debian.org/debian/ \\
 "${@}"
```

现在，每次使用 lb config 时，auto/config 将根据这些选项重置配置。当您想要更改它们时，请编辑此文件中的选项，而不是将它们传递给 lb config。当您使用 lb clean 时，auto/clean 将清理 config/* 文件以及任何其他构建产品。最后，当您使用 lb build 时，auto/build 将在 build.log 中记录构建日志。

**注意：** 这里使用了一个特殊的 noauto 参数来抑制对 auto/config 的另一次调用，从而防止无限递归。确保在进行编辑时不要意外删除它。此外，当您为了可读性而将 lb config 命令分成多行时，如上例所示，请注意不要忘记每行末尾的反斜杠 (\\)。

### 6.2 克隆通过 Git 发布的配置

使用 lb config --config 选项克隆包含实时系统配置的 Git 仓库。如果您希望基于 Debian Live 项目维护的配置，请查看 [Debian Live Project](https://salsa.debian.org/live-team/) 中名为 live-images 的仓库。此仓库包含实时系统预构建映像的配置。

例如，要构建标准映像，请按以下方式使用 live-images 仓库：

```
$ mkdir live-images && cd live-images
$ lb config --config https://salsa.debian.org/live-team/live-images.git::debian
$ cd images/standard
```

编辑 auto/config 和配置树中您需要的任何其他内容以满足您的需求。例如，非官方的非自由预构建映像是通过简单地添加 \--archive-areas "main contrib non-free" 制作的。

您可以选择通过将以下内容添加到您的 ${HOME}/.gitconfig 来定义 Git 配置中的快捷方式：

```
[url "https://salsa.debian.org/live-team/"]
 insteadOf = lso:
```

这使您可以在需要指定 salsa.debian.org git 仓库地址的任何地方使用 lso:。如果您还删除了可选的 .git 后缀，使用此配置启动新映像就像：

```
$ lb config --config lso:live-images::debian
```

克隆整个 live-images 仓库会拉取用于多个映像的配置。如果您在完成第一个映像后想要构建不同的映像，请切换到另一个目录并根据需要进行更改。

无论如何，请记住，每次都必须以超级用户身份构建映像：lb build 