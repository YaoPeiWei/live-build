# 自定义二进制映像

## 11.1 启动加载程序

_live-build_ 默认使用 _syslinux_ 及其一些衍生产品（取决于映像类型）作为启动加载程序。它们可以轻松地根据您的需求进行自定义。

为了使用完整的主题，请将 /usr/share/live/build/bootloaders 复制到 config/bootloaders 并编辑其中的文件。如果您不想修改所有支持的启动加载程序配置，只需提供一个本地自定义的启动加载程序副本，例如在 config/bootloaders/isolinux 中的 **isolinux**，这也足够了，具体取决于您的使用情况。

在修改默认主题之一时，如果您想使用个性化的背景图像与启动菜单一起显示，请添加一个 640x480 像素的 splash.png 图片。然后，删除 splash.svg 文件。

有很多更改的可能性。例如，syslinux 衍生产品默认配置为超时为 0（零），这意味着它们将在其启动画面上无限期暂停，直到您按下一个键。

要修改默认 iso-hybrid 映像的启动超时，只需编辑默认的 **isolinux.cfg** 文件，指定以 1/10 秒为单位的超时。修改后的 **isolinux.cfg** 以在五秒后启动将类似于：

```
include menu.cfg  
default vesamenu.c32  
prompt 0  
timeout 50  
```

## 11.2 ISO 元数据

在创建 ISO9660 二进制映像时，您可以使用以下选项为您的映像添加各种文本元数据。这可以帮助您在不启动映像的情况下轻松识别映像的版本或配置。

- LB_ISO_APPLICATION/--iso-application NAME: 这应该描述将在映像上的应用程序。此字段的最大长度为 128 个字符。
- LB_ISO_PREPARER/--iso-preparer NAME: 这应该描述映像的准备者，通常带有一些联系信息。此选项的默认值是您使用的 _live-build_ 版本，这可能有助于以后调试。此字段的最大长度为 128 个字符。
- LB_ISO_PUBLISHER/--iso-publisher NAME: 这应该描述映像的发布者，通常带有一些联系信息。此字段的最大长度为 128 个字符。
- LB_ISO_VOLUME/--iso-volume NAME: 这应该指定映像的卷 ID。这在某些平台（如 Windows 和 Apple Mac OS）上用作用户可见的标签。此字段的最大长度为 32 个字符。 