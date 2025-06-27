# 自定义Debian安装程序

## 12.1 Debian安装程序的类型

Debian安装程序的三种主要类型是：

**"普通"Debian安装程序**：这是一个普通的实时系统映像，具有单独的内核和initrd（当从适当的引导加载程序中选择时）启动到标准的Debian安装程序实例，就像您下载了Debian的CD映像并启动它一样。包含实时系统和此类独立安装程序的映像通常称为"组合映像"。

在此类映像上，Debian通过使用_debootstrap_从本地媒体或某些基于网络的网络获取和安装.deb软件包，从而将默认的Debian系统安装到硬盘上。

整个过程可以通过预置和多种方式自定义；有关更多信息，请参阅Debian安装程序手册中的相关页面。一旦您有了一个有效的预置文件，_live-build_可以自动将其放入映像中并为您启用。

**"实时"Debian安装程序**：这是一个实时系统映像，具有单独的内核和initrd（当从适当的引导加载程序中选择时）启动到Debian安装程序的实例。

安装将以与上述"普通"安装相同的方式进行，但在实际的软件包安装阶段，不是使用_debootstrap_获取和安装软件包，而是将实时文件系统映像复制到目标。这是通过一个名为_live-installer_的特殊udeb实现的。

在此阶段之后，Debian安装程序将继续正常安装和配置项目，例如引导加载程序和本地用户等。

**注意：**要在同一实时介质的引导加载程序中支持普通和实时安装程序条目，您必须通过预置live-installer/enable=false来禁用_live-installer_。

**"桌面"Debian安装程序**：无论包含哪种类型的Debian安装程序，都可以通过单击图标从桌面启动d-i。在某些情况下，这对用户更友好。为了使用此功能，需要包含_debian-installer-launcher_软件包。

请注意，默认情况下，_live-build_不会在映像中包含Debian安装程序映像，需要通过lb config专门启用。此外，请注意，要使"桌面"安装程序正常工作，实时系统的内核必须与d-i用于指定架构的内核匹配。例如：

```
$ lb config --debian-installer live  
$ echo debian-installer-launcher >> config/package-lists/my.list.chroot  
```

## 12.2 通过预置自定义Debian安装程序

如Debian安装程序手册附录B中所述，"预置提供了一种在安装过程中设置问题答案的方法，而无需在安装运行时手动输入答案。这使得可以完全自动化大多数类型的安装，甚至提供一些在正常安装期间不可用的功能。"这种自定义最好通过将配置放在包含在config/includes.installer/中的preseed.cfg文件中来完成。例如，要预置将语言环境设置为en_US：

```
$ echo "d-i debian-installer/locale string en_US" \
 >> config/includes.installer/preseed.cfg  
```

## 12.3 自定义Debian安装程序内容

出于实验或调试目的，您可能希望包含本地构建的d-i组件udeb软件包。将这些放在config/packages.binary/中以将它们包含在映像中。还可以通过将材料放在config/includes.installer/中，以类似于Live/chroot本地包含的方式在安装程序initrd中包含其他或替换文件和目录。 