+++
author = "zhaojames0707"
comments = true
date = "2016-11-18T15:46:18+08:00"
draft = false
share = true
tags = ["ubuntu", "QQ", "wine", "winetricks"]
title = "在 Ubuntu 16.04 中使用 QQ"

+++

<!--more-->

**更新于2018-02-13**

已有更方便使用的基于 AppImage 的 QQ 及 Tim，参见：https://github.com/askme765cs/Wine-QQ-TIM

以下内容不需再看。

----

工作中经常需要使用 QQ，但是众所周知 Linux 下没有官方版的 QQ。还好在 Wine + Winetricks 的帮助下，使用 QQ 不再是难事。

#### 1. 安装 Wine

由于系统源中的 Wine 版本可能比较旧，推荐使用 Wine 官方的源。

如果你使用的是64位系统，需要先启用32位架构：

    sudo dpkg --add-architecture i386

然后添加源：

    sudo add-apt-repository ppa:wine/wine-builds

更新：

    sudo apt-get update

安装 Wine：

    sudo apt-get install --install-recommends winehq-devel

以上内容来自 Wine 官方 Wiki：https://wiki.winehq.org/Ubuntu

#### 2. 安装 Winetricks-zh

Winetricks 是一个脚本，能很方便的下载 Windows 程序所需的库文件，并对已知问题提供了 work around，非常方便；Winetricks-zh 在 Winetricks 的基础上，提供了很多国内常用软件的支持（例如QQ）。

Winetricks-zh 的 GitHub 地址：https://github.com/hillwoodroc/winetricks-zh

Clone Winetricks-zh 项目：

    git clone git@github.com:hillwoodroc/winetricks-zh.git

将 wintricks-zh 放到 /usr/bin/

    sudo cp winetricks-zh/winetricks-zh /usr/bin/

#### 3. 安装 QQ

命令行输入：

    winetricks-zh qq

如果是第一次使用 winetricks-zh 的话，可能会需要安装依赖库，在弹出的窗口选择中选择安装即可。

随后 Winetricks-zh 开始安装 QQ，安装过程中会从 http://web.archive.org 下载 W2KSP4_EN.EXE 和 InstMsiW.exe，而该网站由于不可描述的原因在大陆无法访问。建议先停止安装程序，使用科学上网工具下载到本地，然后：

    cp W2KSP4_EN.EXE ~/.cache/winetricks/win2ksp4/
    cp InstMsiW.exe ~/.cache/winetricks/msls31/

完成后，再次执行：

    winetricks-zh qq

正常的话接下来会弹出 QQ 安装窗口，有可能中文会是方框，不用担心，安装即可。

安装完成后，按下 Win 键，在搜索框中输入 QQ 便会出现 QQ 的启动程序，点击启动即可运行。

至此已基本完成，但是在对话窗口的输入框内输入中文时，文字会变成方框，文字发出后恢复正常。

#### 4. 解决输入框显示问题

首先在终端中输入：

    export LC_ALL=zh_CN.UTF-8

如果有报错，则需要安装中文语言包：

    sudo apt-get -y install language-pack-zh-hans

然后编辑 QQ 的启动器文件，该文件位于```~/.local/share/applications/wine/Programs/腾讯软件/QQ/腾讯QQ.desktop```，将```Exec=env WINEPRFIX...```改为```Exce=env LC_ALL=zh_CN.UTF-8 WINEPRFIX...```，保存，然后再次启动 QQ，问题应该解决了。

已知问题：查看讨论组的历史消息，会导致 QQ 崩溃，这个问题有待以后解决。
