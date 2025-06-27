# 示例

## 16.1 使用示例

要使用这些示例，您需要一个符合要求的系统来构建它们，并安装了_live-build_，如"安装live-build"中所述。

请注意，为了简洁起见，在这些示例中我们没有指定用于构建的本地镜像。如果您使用本地镜像，可以大大加快下载速度。您可以在使用lb config时指定选项，如"构建时使用的发行版镜像"中所述，或者为了方便起见，在/etc/live/build.conf中为您的构建系统设置默认值。只需创建此文件，并在其中设置相应的LB_MIRROR_*变量为您首选的镜像。构建中使用的所有其他镜像将从这些值中默认设置。例如：

```
LB_MIRROR_BOOTSTRAP="http://mirror/debian/"  
LB_MIRROR_CHROOT_SECURITY="http://mirror/debian-security/"  
LB_MIRROR_CHROOT_BACKPORTS="http://mirror/debian-backports/"  
```

## 16.2 教程1：默认映像

**用例：** 创建一个简单的第一个映像，学习_live-build_的基础知识。

在本教程中，我们将构建一个默认的ISO混合实时系统映像，其中仅包含基本软件包（无Xorg）和一些实时系统支持软件包，作为使用_live-build_的第一个练习。

这再简单不过了：

```
$ mkdir tutorial1 ; cd tutorial1 ; lb config  
```

如果愿意，可以检查config/目录的内容。您将在此处看到一个骨架配置，准备好自定义，或者在本例中立即用于构建默认映像。

现在，以超级用户身份构建映像，并在构建时使用tee保存日志。

```
# lb build 2>&1 | tee build.log  
```

假设一切顺利，过一段时间后，当前目录将包含live-image-amd64.hybrid.iso。此ISO混合映像可以直接在虚拟机中启动，如"使用Qemu测试ISO映像"和"使用VirtualBox测试ISO映像"中所述，或者如"将ISO映像刻录到物理介质"和"将ISO混合映像复制到USB闪存设备"中所述，将其映像到光学介质或USB闪存设备上。

## 16.3 教程2：网页浏览器实用程序

**用例：** 创建一个网页浏览器实用程序映像，学习如何应用自定义。

在本教程中，我们将创建一个适合用作网页浏览器实用程序的映像，作为自定义实时系统映像的介绍。

```
$ mkdir tutorial2  
$ cd tutorial2  
$ lb config  
$ echo "task-lxde-desktop firefox-esr" >> config/package-lists/my.list.chroot  
```

我们在此示例中选择LXDE是因为我们希望提供一个最小的桌面环境，因为映像的重点是我们心目中的单一用途，即网页浏览器。我们甚至可以更进一步，在config/includes.chroot/etc/iceweasel/profile/中为网页浏览器提供默认配置，或者为查看各种网页内容提供额外的支持软件包，但我们将此作为读者的练习。

再次以超级用户身份构建映像，并像教程1中一样保存日志：

```
# lb build 2>&1 | tee build.log  
```

再次验证映像是否正常并进行测试，如教程1中所述。

## 16.4 教程3：个性化映像

**用例：** 创建一个项目以构建个性化映像，包含您喜欢的软件，随时随地携带在USB闪存盘上，并随着您的需求和偏好变化而不断演变。

由于我们将在多个版本中更改我们的个性化映像，并且我们希望跟踪这些更改，尝试实验性操作并在事情不顺利时可能恢复它们，我们将使用流行的git版本控制系统来保存我们的配置。我们还将使用通过自动脚本进行自动配置的最佳实践，如"管理配置"中所述。

### 16.4.1 第一个版本

```
$ mkdir -p tutorial3/auto  
$ cp /usr/share/doc/live-build/examples/auto/* tutorial3/auto/  
$ cd tutorial3  
```

编辑auto/config以如下内容：

```
#!/bin/sh  
  
lb config noauto \  
 --distribution stable \  
 "${@}"  
```

执行lb config以使用您刚刚创建的auto/config脚本生成配置树：

```
$ lb config  
```

现在填充您的本地软件包列表：

```
$ echo "task-lxde-desktop spice-vdagent hexchat" >> config/package-lists/my.list.chroot  
```

首先，--distribution stable确保使用"stable"而不是默认的"testing"。其次，我们添加了_spice-vdagent_以便于在_qemu_中测试映像。最后，我们添加了一个初始的最爱软件包：_hexchat_。

现在，构建映像：

```
# lb build  
```

请注意，与前两个教程不同，我们不再需要输入2>&1 | tee build.log，因为这已包含在auto/build中。

一旦您测试了映像（如教程1中所述）并对其工作满意，就可以初始化我们的git存储库，仅添加我们刚刚创建的自动脚本，然后进行第一次提交：

```
$ git init  
$ cp /usr/share/doc/live-build/examples/gitignore .gitignore  
$ git add .gitignore auto config  
$ git commit -m "Initial import."  
```

### 16.4.2 第二个版本

在此版本中，我们将清理第一次构建，替换_smplayer_软件包为_vlc_软件包，重新构建、测试并提交。

lb clean命令将清除上次构建生成的所有文件，除了缓存，这样就不必重新下载软件包。这确保了后续的lb build将重新运行所有阶段以从我们的新配置重新生成文件。

```
# lb clean  
```

现在在我们的本地软件包列表中，在_lxde_软件包选择_smplayer_、_vlc_和_mplayer-gui_之前安装_vlc_软件包：

```
$ echo "vlc task-lxde-desktop spice-vdagent hexchat" >> config/package-lists/my.list.chroot  
```

再次构建：

```
# lb build  
```

测试，当您满意时，提交下一个版本：

```
$ git commit -a -m "Replacing smplayer with vlc."  
```

当然，可以对配置进行更复杂的更改，可能在config/的子目录中添加文件。当您提交新版本时，只需注意不要手动编辑或提交包含LB_*变量的config顶级文件，因为这些也是构建产品，并且始终由lb clean清理，并通过各自的自动脚本使用lb config重新创建。

我们的教程系列到此结束。虽然即使仅使用这些简单示例中探索的少数功能，也可以进行许多种自定义，但几乎可以创建无限多种不同的映像。本节中的其余示例涵盖了从实时系统用户的收集经验中得出的其他几个用例。

## 16.5 VNC展台客户端

**用例：** 使用_live-build_创建一个映像以直接引导到VNC服务器。

创建一个构建目录，并在其中创建一个骨架配置，禁用推荐以创建一个最小系统。然后创建两个初始软件包列表：第一个由_live-build_提供的名为Packages的脚本生成，第二个包括_xorg_、_gdm3_、_metacity_和_xvnc4viewer_。

```
$ mkdir vnc-kiosk-client  
$ cd vnc-kiosk-client  
$ lb config --apt-recommends false  
$ echo '! Packages Priority standard' > config/package-lists/standard.list.chroot  
$ echo "xorg gdm3 metacity xtightvncviewer" > config/package-lists/my.list.chroot  
```

如"调整APT以节省空间"中所述，您可能需要重新添加一些推荐的软件包以使您的映像正常工作。

列出推荐软件包的一个简单方法是使用_apt-cache_。例如：

```
$ apt-cache depends live-config live-boot  
```

在此示例中，我们发现必须重新包含_live-config_和_live-boot_推荐的几个软件包：user-setup以使自动登录工作，sudo作为关闭系统的基本程序。此外，添加live-tools以便能够将映像复制到RAM并弹出以最终弹出实时介质可能很方便。因此：

```
$ echo "live-tools user-setup sudo eject" > config/package-lists/recommends.list.chroot  
```

之后，在config/includes.chroot中创建目录/etc/skel，并为将启动_metacity_并启动_xvncviewer_的默认用户放置一个自定义.xsession，其中连接到192.168.1.2上的服务器的端口5901：

```
$ mkdir -p config/includes.chroot/etc/skel  
$ cat > config/includes.chroot/etc/skel/.xsession << EOF  
#!/bin/sh  
  
/usr/bin/metacity &  
/usr/bin/xvncviewer 192.168.1.2:1  
  
exit  
EOF  
```

构建映像：

```
# lb build  
```

享受。

## 16.6 适合512MB USB密钥的最小映像

**用例：** 创建一个默认映像，移除一些组件以适应512MB USB密钥，并留出一些空间供您随意使用。

在优化映像以适应某种媒体大小时，您需要了解在大小和功能之间进行的权衡。在此示例中，我们仅修剪到足以在512MB媒体大小中容纳额外材料，但不做任何破坏包完整性的事情，例如通过_localepurge_包清除语言环境数据，或其他此类"侵入性"优化。特别值得注意的是，我们使用--debootstrap-options从头创建一个最小系统，并使用--binary image hdd创建一个可以复制到USB密钥的映像。

```
$ lb config --binary-image hdd --apt-indices false --apt-recommends false --debootstrap-options "--variant=minbase" --firmware-chroot false --memtest none  
```

为了使映像正常工作，我们必须至少重新添加两个被--apt-recommends false选项遗漏的推荐软件包。请参阅"调整APT以节省空间"

```
$ echo "user-setup sudo" > config/package-lists/recommends.list.chroot  
```

此外，您将希望拥有网络访问权限，因此需要重新添加另外两个推荐软件包：

```
$ echo "ifupdown isc-dhcp-client" >> config/package-lists/recommends.list.chroot  
```

现在，以通常的方式构建映像：

```
# lb build 2>&1 | tee build.log  
```

在作者撰写本文时，以上配置在其系统上生成了一个298MiB的映像。这与在教程1中添加--binary-image hdd时默认配置生成的380MiB映像相比是有利的。

省略APT的索引使用--apt-indices false节省了相当多的空间，代价是您需要在实时系统中使用_apt_之前执行apt-get update。使用--apt-recommends false省略推荐软件包可以节省一些额外的空间，代价是省略了一些您可能期望存在的软件包。--debootstrap-options "--variant=minbase"从一开始就引导一个最小系统。不自动包含固件包使用--firmware-chroot false也节省了一些空间。最后，--memtest none防止安装内存测试器。

**注意：** 使用钩子也可以实现最小系统，例如在/usr/share/doc/live-build/examples/hooks中找到的stripped.hook.chroot钩子。它可能会剃掉额外的小空间并生成一个277MiB的映像。然而，它通过从系统上安装的软件包中删除文档和其他文件来实现，这违反了这些软件包的完整性，并且正如注释头所警告的那样，可能会产生不可预见的后果。这就是为什么使用最小_debootstrap_是实现此目标的推荐方法。

## 16.7 本地化的GNOME桌面和安装程序

**用例：** 创建一个GNOME桌面映像，本地化为瑞士并包含一个安装程序。

我们希望使用我们首选的桌面（在本例中为GNOME）制作一个iso-hybrid映像，其中包含标准Debian安装程序为GNOME安装的所有相同软件包。

我们最初的问题是发现适当的语言任务的名称。目前，_live-build_无法帮助解决此问题。虽然我们可能会幸运地通过试错法找到它，但有一个工具，grep-dctrl，可以用来从tasksel-data中的任务描述中挖掘出来，因此准备时，请确保您拥有这两样东西：

```
# apt-get install dctrl-tools tasksel-data  
```

现在我们可以首先使用以下命令搜索适当的任务：

```
$ grep-dctrl -FTest-lang de /usr/share/tasksel/descs/debian-tasks.desc -sTask  
Task: german  
```

通过此命令，我们发现任务被简单地称为german。现在找到相关任务：

```
$ grep-dctrl -FEnhances german /usr/share/tasksel/descs/debian-tasks.desc -sTask  
Task: german-desktop  
Task: german-kde-desktop  
```

在引导时，我们将生成**de_CH.UTF-8**语言环境并选择**ch**键盘布局。现在让我们把这些部分放在一起。回忆一下"使用元包"中提到的任务元包以task-为前缀，我们只需指定这些语言引导参数，然后将标准优先级软件包和我们发现的所有任务元包添加到我们的软件包列表中，如下所示：

```
$ mkdir live-gnome-ch  
$ cd live-gnome-ch  
$ lb config \  
 --bootappend-live "boot=live components locales=de_CH.UTF-8 keyboard-layouts=ch" \  
 --debian-installer live  
$ echo '! Packages Priority standard' > config/package-lists/standard.list.chroot  
$ echo task-gnome-desktop task-german task-german-desktop >> config/package-lists/desktop.list.chroot  
$ echo debian-installer-launcher >> config/package-lists/installer.list.chroot  
```

请注意，我们已包含_debian-installer-launcher_软件包以从实时桌面启动安装程序。 