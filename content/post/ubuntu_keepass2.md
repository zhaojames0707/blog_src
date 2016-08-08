+++
author = "zhaojames0707"
comments = true
date = "2016-08-08T10:02:18+08:00"
draft = false
share = true
tags = ["keepass", "ubuntu"]
title = "解决 Ubuntu 下 KeePass2 中文显示为方块的问题"

+++

最近开始使用 KeePass2，遇到了中文显示为方块的问题。

<!--more-->

最近我的京东和支付宝接连被人登陆，使我不得不重视起密码安全问题，下定决心使用密码管理软件，自动生成并记录强密码。

在了解了市面上的若干个密码管理软件以后，我选择了 KeePass2，因为它支持多平台，并且开源。我在 Mac 上使用 MacPass，iPhone 上使用 MiniKeePass，Ubuntu上则使用 KeePass2。

一开始一切正常，直到我在 KeePass2 上输入了中文，中文均显示为方块。在网上参考了众多讨论后，我找到了在我电脑上的解决办法。
**注**: 我的系统是 Ubuntu 16.04 64-bit，KeePass 版本为 2.34。

#### 1. 下载 KeePass2 语言包

KeePass 的[官网](http://keepass.info/translations.html)提供了各种语言的语言包，下载中文2.x版本语言包后解压到```~/.local/share/KeePass/```目录下，重启 KeePass 后设置 View->Change Language，选择 Simplified Chinese 即可。

#### 2. 修改启动脚本

参考[博客](http://wenliangcan.github.io/blog/2014/04/20/keepass-cjk-fonts-displaying/)，修改```/usr/bin/keepass2```，加入

```
export LANG=zh_CN.utf8
```

#### 3. 修改系统字体设置

参考[FAQ](http://forum.ubuntu.org.cn/viewtopic.php?f=8&t=473678)，修改```/etc/fonts/conf.avail/65-nonlatin.conf```，添加

```xml
   <alias>
      <family>Ubuntu</family>
      <prefer>
         <family>sans-serif</family>
      </prefer>
   </alias>
```

在进行上述操作后，重启 KeePass2，应该就可以正常显示中文了。