# Debian Live 手册

## 自定义软件包安装

或许最基本的实时系统自定义是选择要包含在映像中的软件包。本章将指导您通过各种构建时选项来自定义 _live-build_ 的软件包安装。影响可安装软件包的最广泛选择是发行版和存档区域。为了确保良好的下载速度，您应该选择一个附近的发行版镜像。您还可以添加自己的存储库以获取回溯、实验或自定义软件包，或直接将软件包作为文件包含。您可以定义软件包列表，包括将安装许多相关软件包的元软件包，例如特定桌面或语言的软件包。最后，许多选项可以在构建时对 _apt_ 或 _aptitude_ 进行一些控制，以便在安装软件包时使用。如果您使用代理，想要禁用推荐软件包的安装以节省空间，或者需要通过APT固定来控制安装的软件包版本，这些选项可能会很方便。

### 8.1 软件包来源

#### 8.1.1 发行版、存档区域和模式

您选择的发行版对可以包含在实时映像中的软件包有最广泛的影响。指定代号，默认为 **testing**。可以通过其代号指定存档中携带的任何当前发行版。（有关详细信息，请参阅术语。）\--distribution 选项不仅影响存档中的软件包来源，还指示 _live-build_ 启用其他来源。

例如，要针对 **stable** 版本进行构建，启用 _security_、_updates_（默认启用）以及 _proposed-updates_ 和 _backports_，请指定：

```
$ lb config --distribution stable --proposed-updates true --backports true
```

类似地，对于 **unstable** 版本，**sid**，没有 _security_ 或 _updates_，请指定：

```
$ lb config --distribution sid
```

在发行版存档中，存档区域是存档的主要划分。在 Debian 中，这些是 main、contrib 和 non-free。只有 main 包含属于 Debian 发行版的软件，因此这是默认设置。可以指定一个或多个值，例如：

```
$ lb config --archive-areas "main contrib non-free"
```

通过 \--mode 选项为某些 Debian 衍生版提供实验性支持。默认情况下，如果您在 Debian 或未知系统上构建，则此选项设置为 debian。 如果在任何受支持的衍生版上调用 lb config，它将默认创建该衍生版的映像。如果在例如 ubuntu 模式下运行 lb config，则为指定的衍生版支持发行版名称和存档区域，而不是 Debian 的名称和区域。该模式还修改 _live-build_ 的行为以适应衍生版。

**注意：** 为这些模式添加的项目主要负责支持这些选项的用户。Debian Live 项目则根据衍生项目的反馈在尽力而为的基础上提供开发支持，因为我们自己不开发或支持这些衍生版。

#### 8.1.2 发行版镜像

Debian 存档通过全球大型镜像网络进行复制，以便每个地区的人们可以选择附近的镜像以获得最佳下载速度。每个 \--mirror-* 选项管理在构建的各个阶段使用的发行版镜像。回想一下构建阶段，**bootstrap** 阶段是 _debootstrap_ 用最小系统初始化填充 chroot 的阶段，而 **chroot** 阶段是构建实时系统文件系统的 chroot 阶段。因此，相应的镜像开关用于这些阶段，稍后在 **binary** 阶段，使用 \--mirror-binary 和 \--mirror-binary-security 值，取代早期阶段使用的任何镜像。

#### 8.1.3 构建时使用的发行版镜像

要将构建时使用的发行版镜像设置为指向本地镜像，只需设置 \--mirror-bootstrap 和 \--mirror-chroot-security，如下所示。

```
$ lb config --mirror-bootstrap http://localhost/debian/ \\
  --mirror-chroot-security http://localhost/debian-security/
```

chroot 镜像由 \--mirror-chroot 指定，默认为 \--mirror-bootstrap 值。

#### 8.1.4 运行时使用的发行版镜像

\--mirror-binary* 选项管理放置在二进制映像中的发行版镜像。这些可以用于在运行实时系统时安装其他软件包。默认情况下使用 deb.debian.org，这是一项基于用户的 IP 家族和镜像的可用性等因素选择地理上接近的镜像的服务。这是一个合适的选择，当您无法预测哪个镜像对所有用户来说是最好的时。或者，您可以指定自己的值，如下例所示。从此配置构建的映像仅适用于网络上可以访问"镜像"的用户。

```
$ lb config --mirror-binary http://mirror/debian/ \\
  --mirror-binary-security http://mirror/debian-security/ \\
  --mirror-binary-backports http://mirror/debian-backports/
```

#### 8.1.5 其他存储库

您可以添加更多存储库，扩展超出目标发行版可用的软件包选择。这些可能是例如用于回溯、实验或自定义软件包。要配置其他存储库，请创建 config/archives/your-repository.list.chroot 和/或 config/archives/your-repository.list.binary 文件。与 \--mirror-* 选项一样，这些管理在构建映像时的 **chroot** 阶段和 **binary** 阶段使用的存储库，即用于运行实时系统时。

例如，config/archives/live.list.chroot 允许您在实时系统构建时从 debian-live 快照存储库安装软件包。

```
deb http://debian-live.alioth.debian.org/ sid-snapshots main contrib non-free
```

如果将相同的行添加到 config/archives/live.list.binary，存储库将被添加到您的实时系统的 /etc/apt/sources.list.d/ 目录中。

如果存在此类文件，它们将被自动拾取。

您还应该将用于签署存储库的 ASCII 装甲 GPG 密钥放入 config/archives/your-repository.key.{binary,chroot} 文件中。

如果需要自定义 APT 固定，这样的 APT 首选项片段可以放置在 config/archives/your-repository.pref.{binary,chroot} 文件中，并将自动添加到您的实时系统的 /etc/apt/preferences.d/ 目录中。

同样，如果您需要自定义 APT_AUTH.CONF(5) 身份验证配置，可以将其放置在 config/archives/your-repository.auth.{binary,chroot} 文件中，并将自动添加到您的实时系统的 /etc/apt/auth.conf.d/ 目录中。

### 8.2 选择要安装的软件包

有多种方法可以选择 _live-build_ 将在您的映像中安装哪些软件包，以满足各种不同的需求。您可以简单地在软件包列表中命名要安装的单个软件包。您还可以在这些列表中使用元软件包，或使用软件包控制文件字段选择它们。最后，您可以将软件包文件放在您的 config/ 目录中，这非常适合在新软件包或实验软件包在存储库中可用之前进行测试。

#### 8.2.1 软件包列表

软件包列表是一种强大的表达方式，用于指定应安装哪些软件包。列表语法支持条件部分，使其易于构建列表并将其适应于多种配置。软件包名称也可以在构建时使用 shell 助手注入到列表中。

**注意：** 指定不存在的软件包时 _live-build_ 的行为由您选择的 APT 实用程序决定。有关详细信息，请参阅选择 apt 或 aptitude。

#### 8.2.2 使用元软件包

填充您的软件包列表的最简单方法是使用由您的发行版维护的任务元软件包。例如：

```
$ lb config
$ echo task-gnome-desktop > config/package-lists/desktop.list.chroot
```

这取代了live-build 2.x中支持的旧的预定义列表方法。与预定义列表不同，任务元软件包并不特定于Live System项目。相反，它们由发行版内的专业工作组维护，因此反映了每个小组关于哪些软件包最能满足预期用户需求的共识。它们还涵盖了比它们所取代的预定义列表更广泛的用例。

所有任务元软件包都以task-为前缀，因此确定哪些可用的快速方法是匹配软件包名称（尽管它可能包含一些与名称匹配但不是元软件包的错误匹配）：

```
$ apt-cache search --names-only ^task-
```

除了这些，您还会发现其他具有各种用途的元软件包。有些是更广泛任务包的子集，如gnome-core，而其他则是Debian Pure Blend的单个专用部分，例如education-*元软件包。要列出存档中的所有元软件包，请安装debtags软件包并列出所有带有role::metapackage标签的软件包，如下所示：

```
$ debtags search role::metapackage
```

#### 8.2.3 本地软件包列表

无论您列出的是元软件包、单个软件包还是两者的组合，所有本地软件包列表都存储在config/package-lists/中。由于可以使用多个列表，这非常适合模块化设计。例如，您可以决定将一个列表专用于特定的桌面选择，另一个列表专用于一组相关的软件包，这些软件包可以同样容易地用于不同的桌面之上。这使您可以在最小的麻烦下尝试不同的包组合，在不同的实时映像项目之间共享通用列表。

此目录中存在的软件包列表需要具有.list后缀才能被处理，然后是附加的阶段后缀，.chroot或.binary以指示列表用于哪个阶段。

.list.chroot_install列表中的软件包同时存在于实时系统和已安装系统中。

**注意：** 如果您不指定阶段后缀，列表将用于两个阶段。通常，您希望指定.list.chroot，以便软件包仅安装在实时文件系统中，而不会在介质上放置.deb的额外副本。

#### 8.2.4 本地二进制软件包列表

要制作二进制阶段列表，请在config/package-lists/中放置一个以.list.binary结尾的文件。这些软件包不会安装在实时文件系统中，但会包含在实时介质下的pool/中。您通常会将此类列表与非实时安装程序变体之一一起使用。如上所述，如果您希望此列表与您的chroot阶段列表相同，只需使用.list后缀即可。

#### 8.2.5 生成的软件包列表

有时，最好的方法是通过脚本生成列表。任何以感叹号开头的行表示在构建映像时在chroot中执行的命令。例如，可以在软件包列表中包含行`! grep-aptavail -n -sPackage -FPriority standard | sort`以生成具有Priority: standard的可用软件包的排序列表。

实际上，使用grep-aptavail命令（来自dctrl-tools软件包）选择软件包非常有用，以至于live-build提供了一个Packages助手脚本作为便利。此脚本接受两个参数：字段和模式。因此，您可以创建一个包含以下内容的列表：

```
$ lb config
$ echo '! Packages Priority standard' > config/package-lists/standard.list.chroot
```

#### 8.2.6 在软件包列表中使用条件

存储在config/*中的任何live-build配置变量（减去LB_前缀）都可以在软件包列表中的条件语句中使用。通常，这意味着任何lb config选项大写并将破折号更改为下划线。但实际上，只有那些影响软件包选择的选项才有意义，例如DISTRIBUTION、ARCHITECTURES或ARCHIVE_AREAS。

例如，如果指定了--architectures amd64，则安装ia32-libs：

```
#if ARCHITECTURES amd64
ia32-libs
#endif
```

您可以测试多个值中的任何一个，例如，如果指定了--architectures i386或--architectures amd64，则安装memtest86+：

```
#if ARCHITECTURES i386 amd64
memtest86+
#endif
```

您还可以测试可能包含多个值的变量，例如，如果通过--archive-areas指定了contrib或non-free，则安装vrms：

```
#if ARCHIVE_AREAS contrib non-free
vrms
#endif
```

不支持条件的嵌套。

#### 8.2.7 在安装时删除软件包

您可以在config/package-lists目录中列出带有.list.chroot_live和.list.chroot_install后缀的软件包。如果同时存在实时和安装列表，则在安装后（如果用户使用安装程序）通过钩子删除.list.chroot_live列表中的软件包。.list.chroot_install列表中的软件包同时存在于实时系统和已安装系统中。这是安装程序的特殊调整，如果您在配置中设置了--debian-installer live，并希望在安装时删除实时系统特定的软件包，这可能会很有用。

#### 8.2.8 总结

下表显示了实现所需软件包可用性所需的配置文件。

| 功能描述                           | X.chroot | X.chroot_live | X | X.binary |
|----------------------------------|----------|---------------|---|----------|
| 软件包安装在实时系统中              | 是       | 是            | 是| 否       |
| 安装实时系统后删除软件包            | 否       | 是            | 否| 不适用   |
| 可以从实时系统安装软件包而无需网络  | 不适用   | 不适用        | 是| 是       |

*1：因为安装程序需要此软件包

```
X = config/package-lists/custom_name.list
```

#### 8.2.9 桌面和语言任务

桌面和语言任务是需要一些额外规划和配置的特殊情况。在这方面，实时映像与Debian安装程序映像不同。在Debian安装程序中，如果介质是为特定桌面环境风格准备的，则相应的任务将自动安装。因此，内部有gnome-desktop、kde-desktop、lxde-desktop和xfce-desktop任务，这些任务都没有在tasksel的菜单中提供。同样，语言任务也没有菜单条目，但用户在安装期间的语言选择会影响相应语言任务的选择。

在开发桌面实时映像时，映像通常直接启动到工作桌面，桌面和默认语言的选择是在构建时而不是在运行时进行的，就像在Debian安装程序的情况下那样。这并不是说不能构建支持多个桌面或多种语言并为用户提供选择的实时映像，但这不是live-build的默认行为。

由于没有自动为语言任务提供的规定，包括语言特定的字体和输入法软件包，如果您需要它们，您需要在配置中指定它们。例如，包含德语支持的GNOME桌面映像可能包括这些任务元软件包：

```
$ lb config
$ echo "task-gnome-desktop task-laptop" >> config/package-lists/my.list.chroot
$ echo "task-german task-german-desktop task-german-gnome-desktop" >> config/package-lists/my.list.chroot
```

#### 8.2.10 内核风格和版本

根据架构，默认情况下将包含一个或多个内核风格在您的映像中。您可以通过--linux-flavours选项选择不同的风格。每个风格都附加到默认存根linux-image以形成每个元软件包名称，反过来依赖于要包含在映像中的确切内核软件包。

因此，默认情况下，amd64架构映像将包括linux-image-amd64风格元软件包，而i386架构映像将包括linux-image-586元软件包。

当在您配置的存档中有多个内核软件包版本可用时，您可以通过--linux-packages选项指定不同的内核软件包名称存根。例如，假设您正在构建amd64架构映像并添加实验性存档以进行测试，以便您可以安装linux-image-3.18.0-trunk-amd64内核。您将按如下方式配置该映像：

```
$ lb config --linux-packages linux-image-3.18.0-trunk
$ echo "deb http://deb.debian.org/debian/ experimental main" > config/archives/experimental.list.chroot
```

#### 8.2.11 自定义内核

只要它们集成在Debian软件包管理系统中，您就可以构建和包含自己的自定义内核。live-build系统不支持未构建为.deb软件包的内核。

部署自己的内核软件包的正确和推荐的方法是遵循内核手册中的说明。请记住适当地修改ABI和风格后缀，然后在您的存储库中包含linux和匹配的linux-latest软件包的完整构建。

如果您选择在没有匹配元软件包的情况下构建内核软件包，您需要指定一个适当的--linux-packages存根，如内核风格和版本中所述。正如我们在安装修改或第三方软件包中解释的那样，最好将自定义内核软件包包含在您自己的存储库中，尽管该部分中讨论的替代方案也可以使用。

本文件超出了如何自定义内核的范围。但是，您至少必须确保您的配置满足以下最低要求：

- 使用初始ramdisk。
- 包含联合文件系统模块（即通常是OverlayFS）。
- 包含配置所需的任何其他文件系统模块（即通常是squashfs）。

### 8.3 安装修改或第三方软件包

虽然这与实时系统的理念相悖，但有时可能需要使用修改版本的软件包构建实时系统，这些软件包在Debian存储库中。这可能是为了修改或支持其他功能、语言和品牌，甚至是删除现有软件包中不需要的元素。同样，"第三方"软件包可能用于添加定制和/或专有功能。

本节不涉及有关构建或维护修改软件包的建议。然而，Joachim Breitner的"如何私下分叉"方法可能会引起兴趣，网址为‹http://www.joachim-breitner.de/blog/archives/282-How-to-fork-privately.html›。定制软件包的创建在Debian新维护者指南中有所介绍，网址为‹https://www.debian.org/doc/manuals/maint-guide/›及其他地方。

有两种方法可以安装修改的自定义软件包：

- packages.chroot
- 使用自定义APT存储库

使用packages.chroot更容易实现，适用于"一次性"自定义，但有一些缺点，而使用自定义APT存储库则需要更多时间来设置。

#### 8.3.1 使用packages.chroot安装自定义软件包

要安装自定义软件包，只需将其复制到config/packages.chroot/目录中。此目录中的软件包将在构建期间自动安装到实时系统中 - 您无需在其他地方指定它们。

软件包必须按规定的方式命名。一个简单的方法是使用dpkg-name。

使用packages.chroot安装自定义软件包有以下缺点：

- 无法使用安全APT。
- 您必须在config/packages.chroot/目录中安装所有适当的软件包。
- 它不适合将实时系统配置存储在版本控制中。

#### 8.3.2 使用APT存储库安装自定义软件包

与使用packages.chroot不同，使用自定义APT存储库时，您必须确保在其他地方指定软件包。有关详细信息，请参阅选择要安装的软件包。

虽然创建APT存储库以安装自定义软件包似乎不必要的努力，但基础设施可以在以后轻松重用，以提供修改软件包的更新。

APT存储库不一定需要在线，您可以使用本地存储库。但是，在这两种情况下，存储库都需要签名。

示例：

```
$ gpg --armor --output config/archives/custom_repo.gpg.key${EXTENSION} --export-options export-minimal --export ${SIGNING_KEY}
$ cat << EOF > config/archives/custom_repo.list${EXTENSION}
deb [signed-by=/etc/apt/trusted.gpg.d/custom_repo.gpg.key${EXTENSION}.asc] ${URI} ${SUITE} ${COMPONENTS}
EOF
$ echo "${PACKAGES_FROM_REPOSITORY}" > config/package-lists/custom_repo.list${EXTENSION}
```

其中：

- ${EXTENSION}：可选的阶段后缀，请参阅摘要
- ${SIGNING_KEY}：存储库签名的密钥ID
- ${URI}：存储库的URI，例如http://deb.debian.org/debian/或file://$(pwd)/my_local_repository
- ${SUITE}：存储库中的套件，例如my-debian-based-distro
- ${COMPONENTS}：存储库中的组件，例如main
- ${PACKAGES_FROM_REPOSITORY}：要安装的软件包名称（依赖项也将自动安装） 

#### 8.3.3 自定义软件包和APT

live-build 使用 APT 将所有软件包安装到实时系统中，因此将继承该程序的行为。一个相关的例子是（假设默认配置）在两个不同的存储库中提供具有不同版本号的软件包时，APT 将选择安装版本号较高的软件包。

因此，您可能希望在自定义软件包的 debian/changelog 文件中增加版本号，以确保安装您修改后的版本而不是官方 Debian 存储库中的版本。这也可以通过更改实时系统的 APT 固定首选项来实现 - 有关更多信息，请参阅 APT 固定。

### 8.4 在构建时配置 APT

您可以通过一些仅在构建时应用的选项来配置 APT。（运行实时系统时使用的 APT 配置可以通过 config/includes.chroot/ 以正常方式配置实时系统内容。）有关完整列表，请查找 lb_config 手册页中以 apt 开头的选项。

#### 8.4.1 选择 apt 或 aptitude

您可以选择在构建时安装软件包时使用 apt 或 aptitude。使用哪个实用程序由 lb config 的 --apt 参数决定。选择实现首选软件包安装行为的方法，显著的区别在于如何处理缺失的软件包。

apt：使用此方法，如果指定了缺失的软件包，软件包安装将失败。这是默认设置。
aptitude：使用此方法，如果指定了缺失的软件包，软件包安装将成功。

#### 8.4.2 使用代理与 APT

一个常见的 APT 配置是处理在代理后面构建映像。您可以根据需要使用 --apt-http-proxy 选项指定您的 APT 代理，例如：

```
$ lb config --apt-http-proxy http://proxy/
```

#### 8.4.3 调整 APT 以节省空间

您可能会发现需要在映像介质上节省一些空间，在这种情况下，以下一个或两个选项可能会引起兴趣。

如果您不想在映像中包含 APT 索引，您可以通过以下方式省略这些：

```
$ lb config --apt-indices false
```

这不会影响 /etc/apt/sources.list 中的条目，但仅仅是 /var/lib/apt 是否包含索引文件。权衡是 APT 需要这些索引才能在实时系统中运行，因此在执行 apt-cache search 或 apt-get install 之前，用户必须先执行 apt-get update 以创建这些索引。

如果您发现推荐软件包的安装使您的映像过于臃肿，前提是您准备好处理下面讨论的后果，您可以通过以下方式禁用 APT 的默认选项：

```
$ lb config --apt-recommends false
```

关闭推荐的最重要后果是 live-boot 和 live-config 本身推荐一些软件包，这些软件包为大多数实时配置提供重要功能。

您很可能需要重新添加的两个软件包是：

user-setup，live-config 推荐用于创建实时用户。
sudo，live-config 推荐用于在实时映像中获取 root 访问权限，这在关闭计算机时需要。

```
$ lb config --apt-recommends false
$ echo "user-setup sudo" > config/package-lists/recommends.list.chroot
```

在除最特殊的情况下，您需要将至少一些这些推荐重新添加到您的软件包列表中，否则您的映像将无法按预期工作，甚至无法工作。查看每个包含在构建中的 live-* 软件包的推荐软件包，如果您不确定可以省略它们，请将它们重新添加到您的软件包列表中。

更一般的后果是，如果您不为任何给定的软件包安装推荐的软件包，即"在所有但不寻常的安装中与此软件包一起找到的软件包"（Debian 政策手册，第 7.2 节），则可能会遗漏您的实时系统用户实际需要的一些软件包。因此，我们建议您查看关闭推荐对您的软件包列表的影响（请参阅 lb build 生成的 binary.packages 文件），并在您的列表中重新包含任何您仍然希望安装的缺失软件包。或者，如果您发现只想省略少量推荐的软件包，请启用推荐并在选定的软件包上设置负 APT 固定优先级以防止它们被安装，如 APT 固定中所述。

#### 8.4.4 向 apt 或 aptitude 传递选项

如果没有 lb config 选项以您需要的方式更改 APT 的行为，请使用 --apt-options 或 --aptitude-options 将任何选项传递给您配置的 APT 工具。有关详细信息，请参阅 apt 和 aptitude 的手册页。请注意，这两个选项都有默认值，您需要在提供的任何覆盖之外保留这些默认值。因此，例如，假设您已从 snapshot.debian.org 包含了一些内容以进行测试，并希望指定 Acquire::Check-Valid-Until=false 以使 APT 对过期的 Release 文件满意，您可以按照以下示例进行操作，在默认值 --yes 之后附加新选项：

```
$ lb config --apt-options "--yes -oAcquire::Check-Valid-Until=false"
```

请查看手册页以充分了解这些选项以及何时使用它们。这只是一个示例，不应被视为以这种方式配置映像的建议。此选项不适用于，例如，实时映像的最终发布。

对于涉及 apt.conf 选项的更复杂的 APT 配置，您可能希望创建一个 config/apt/apt.conf 文件。另请参阅其他 apt-* 选项以获取一些常用选项的便捷快捷方式。

#### 8.4.5 APT 固定

作为背景，请首先阅读 apt_preferences(5) 手册页。APT 固定可以在构建时或运行时配置。对于前者，创建 config/archives/*.pref、config/archives/*.pref.chroot 和 config/apt/preferences。对于后者，创建 config/includes.chroot/etc/apt/preferences。

假设您正在构建一个 trixie 实时系统，但需要在构建时从 sid 安装所有最终进入二进制映像的实时软件包。您需要将 sid 添加到您的 APT 源中，并将来自它的实时软件包固定在比默认优先级更高的位置，但来自它的所有其他软件包固定在更低的位置。因此，只有您想要的软件包在构建时从 sid 安装，所有其他软件包都从目标系统发行版 trixie 中获取。以下将实现这一点：

```
$ echo "deb http://mirror/debian/ sid main" > config/archives/sid.list.chroot
$ cat >> config/archives/sid.pref.chroot << EOF
Package: live-*
Pin: release n=sid
Pin-Priority: 600

Package: *
Pin: release n=sid
Pin-Priority: 1
EOF
```

负固定优先级将阻止安装软件包，例如在您不希望安装由另一个软件包推荐的软件包的情况下。假设您正在使用 config/package-lists/desktop.list.chroot 中的 task-lxde-desktop 构建 LXDE 映像，但不希望用户被提示将 wifi 密码存储在密钥环中。此元软件包依赖于 lxde-core，后者推荐 gksu，gksu 又推荐 gnome-keyring。因此，您希望省略推荐的 gnome-keyring 软件包。这可以通过将以下节添加到 config/apt/preferences 来实现：

```
Package: gnome-keyring
Pin: version *
Pin-Priority: -1
``` 