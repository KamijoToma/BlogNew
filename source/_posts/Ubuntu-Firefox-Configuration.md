---
title: Ubuntu 22.04下FireFox的问题和配置方法
description: 回顾一下和Ubuntu 22.04的Firefox的斡旋过程中踩到的坑，并提供我本人的最佳实践。
tags:
  - 技术
  - Ubuntu
  - Linux
  - Firefox
  - 浏览器
categories:
  - 技术
date: 2024-01-25 15:24:50
updated: 2024-01-25 15:24:50
keywords:
cover:
katex:
---


## 前言

最近寒假期间参加了一个超算方面的竞赛，由于比赛需要，再加上我自己也非常想在实体机上安装尝试一下Linux系统，所以终于在2024年初给我的联想笔记本装上了Ubuntu 22.04。（为什么没安装Arch: ~~博主太懒~~ 马上就要比赛了没时间鼓捣 ~~那为什么不装Manjaro?~~）总之，我装了Ubuntu 22.04。

众所周知，Linux系统一般系统自带的浏览器是FireFox，Ubuntu也不例外。但是，Ubuntu系统的FireFox有些特殊：它不是由大家熟知的Debian系的软件包管理器APT提供的。实际上，Ubuntu的开发公司Canonical为了推广其研发的Snap软件包管理系统，从Ubuntu的某个版本起就将Firefox从APT源换成了由Snap提供。如果你现在在Ubuntu上运行`sudo apt install firefox`，你会发现`firefox`软件包是一个空包，它最终会运行`snap install firefox`从Snap安装。Ubuntu默认的软件源中已经不包含FireFox。

对于普通用户来说，这些都是软件安装方式的改变，似乎并不会影响他们正常使用软件本身。然而，事实真的是如此吗？Canonical推崇的Snap是基于容器的，每个软件都运行在单独的一个隔离容器内，类似Docker。容器化带来了隔离的安全性，但是也引入了新问题，例如**无法更换跟随系统更换字体、无法上传文件、无法与Gnome Extensions插件配合安装桌面扩展等**。因此，是时候换掉Snap FireFox了！

## 折腾记录

由于博主更换了2个版本的FireFox，每种FireFox都遇到了不同的问题，因此在这里一边描述过程一边阐述问题。

### Snap，NMSL

#### 无法安装Gnome Extensions
在全新安装Ubuntu后，博主最先遇到的问题是FireFox无法安装[Gnome Extensions](https://extensions.gnome.org/)。

> GNome Extensions提供了很多由用户创建的针对Ubuntu桌面的拓展，相当于浏览器的插件商店。
>
> 想要安装一个Extension，需要在系统中安装一个`chrome-gnome-shell`软件包，还需要在浏览器中安装一个插件。网站通过`网站 <----> 插件 <---> chrome-gnome-shell <---> 桌面`如此流程进行通信。

但是，由于SNAP先进的隔离安全性，阻止了插件和本地之间的联系，因此会出现`the native host connector cannot be detected.`的[报错](https://bugzilla.mozilla.org/show_bug.cgi?id=1661935)。

[解决的办法之一](https://www.oschina.net/news/204852/firefox-snap-install-gnome-extensions)是安装 `xdg-desktop-portal`提供中继，博主当时也用了这种方法，确实有效，于是就正常呆了几天。

然后，新的问题又出现了。

#### 无法上传文件

这个问题是在提交比赛报告的时候发现的，当时点“打开本地文件”按钮半天没反应，还以为是网站坏掉了……写到这非常无语，Canonical你这么打包的浏览器[竟然不支持上传文件](https://support.mozilla.org/gl/questions/1375307)？那你这个snap有个屁用！正好比赛打完了，于是当即决定换掉snap版FireFox。

### APT，你坏

进行了一下网上冲浪，找到了几个[有关这方面的帖子](https://askubuntu.com/questions/1399383/how-to-install-firefox-as-a-traditional-deb-package-without-snap-in-ubuntu-22)，看来大家都很讨厌Snap嘛。

按照帖子换掉了FireFox（还痛失了浏览数据）,打开崭新的浏览器，Mozilla Firefox向我问好，于是快乐的网上冲浪生活就这样开始了……吗？

#### 严重的掉帧问题

刚开始我并没有意识到有什么问题，不幸的是，我又参加了[一场比赛](https://hpcgame.pku.edu.cn/)，在切换页面的过程中，我遭遇了严重的掉帧问题，FireFox的性能真的有这么差吗？

打开Firefox的Troubleshooting信息页面(Menu->Help->More troubleshooting information)，我似乎发现了问题—— **FireFox当前使用软件渲染！** 具体就是在Graphics（图形）节的Compositing（合成）选项处，若其显示的是WebRender，则为硬件渲染，若为WebRender(Software)，则说明其为软件渲染。同样，若GPU处显示为llvmpipe，则更能说明其为软件渲染。

在网上搜索了一圈也没找到相关问题，这可能是我这里独有的问题了吧。既然APT也不好，那我就切换到Mozilla提供的二进制构建。

在这里提一下，Linux上常用的软件包发行形式有以下几种：
- 源代码发行，用户下载到软件的源代码，在本地用编译器编译（例如./configure make make install）
- 二进制发行，用户下载到的直接就是软件的可执行程序。这种分发形式免去了用户在本地编译软件所消耗的大量时间，但是很可能由于编译环境和用户运行环境的差异造成依赖库问题
- 软件包发行，用户所使用的发行版提供软件包管理器和针对此发行版和架构编译好的软件包，用户直接使用软件包管理工具即可安装，解决了依赖库问题，但是可能安装不了最新版本的软件~~说的就是你RHEL~~

### 你好，二进制

安装二进制软件非常简单，就是[下载压缩包](https://www.mozilla.org/en-US/firefox/all/#product-desktop-release)——解压压缩包——双击`firefox`运行即可。但是，每次都双击运行总感觉有点难受，有没有办法创建快捷方式呢？经过搜索，Mozilla在[这篇文章](https://support.mozilla.org/en-US/kb/install-firefox-linux#w_install-firefox-from-mozilla-builds)里提供了解决方法（和从二进制安装的全套指南），可以手动下载快捷方式安装到桌面中。

## 博主的最佳实践

1. 远离snap，会变得不幸
2. 远离llvmpipe，会变得不幸
3. 多上网查找解决方案

### 备份数据

通过登陆[Mozilla Account](https://accounts.firefox.com/)同步数据，或者通过FireFox内置的数据导出功能将浏览数据导出。需要注意的是，以上两种方式**都不能完整导出数据，至少会损失插件的配置数据（例如TamperMonkey的脚本和脚本配置）**，所以建议**应在安装完系统后尽快更换FireFox**。

### 卸载FireFox

通过下面的命令卸载Snap版FireFox，卸载前**请确保重要数据已经导出！**

```bash
sudo snap remove firefox
```

### 下载安装二进制版本FireFox并配置快捷方式

这部分请参考[Mozilla提供的官方指南](https://support.mozilla.org/en-US/kb/install-firefox-linux#w_install-firefox-from-mozilla-builds)，建议软件的安装路径遵循[FHS标准](https://zh.wikipedia.org/zh-hans/%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E6%A0%87%E5%87%86)，安装到`/opt/`目录下。

## 总结

***Canonical，你（）什么时候（）啊***
