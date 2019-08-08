---
layout: page
title: Linux GDM登录界面如何适应高分屏
date: 2017-04-15 23:45:12
tags:
- linux
- gdm
- scale factor
---
> GDM(GNOME Desktop Manager)是一种GNOME显示环境的管理器，它是一个运行在后台的小程序（脚本），运行你的 X 会话，显示一个登录界面并在你忘记密码的时候告诉你无法登录，GDM 比 xdm 在任何方面都做的更好，也没有 xdm 那么多的漏洞。 它没有使用任何来自xdm的代码。 它支持 XDMCP, 并事实上扩展了我认为XDMCP缺少的一部分。

### 背景介绍
Linux对于高分屏的自适应不是很好，使用过程中由于屏幕分辨率较高，系统调整缩放级别系数偏大，直接导致显示窗口过大。Google了相关资料，今天写一篇教程修改GDM登录界面和GNOME界面的缩放级别系数。
对于高分屏，GDM登录界面显示很大：
![login]({{ site.url }}/assets/LinuxGDMScaleFactor/login.png "login")
GNOME桌面偶尔也无法自适应：
![login2]({{ site.url }}/assets/LinuxGDMScaleFactor/login2.png "login2")

### 解决方法
#### GNOME桌面
顺带介绍以下GNOME桌面缩放级别修改方式。
最简单的解决方法是打开gnome-tweak-tool看窗口缩放值，将其调整为1即可。但是有时候他的值是1的情况下屏幕显示还是很大，将其调整为2没有任何改变。此时就需要使用以下命令查看scale值发现其实并不是1，而是2
```bash
$ gsettings get org.gnome.desktop.interface scaling-factor
```
显示的结果是
```bash
unit32 2
```
2则表示当前缩放级别实际是2，使用以下命令调整为1即可。

```bash
$ gsettings set org.gnome.desktop.interface scaling-factor 1
```

#### GDM登录界面
好了，重点在这。其实修改方式跟以上方法如出一辙。

配置X服务访问权限
```bash
# xhost +SI:localuser:gdm
```
打开dconf工具直接修改
如果没有dconf请先安装
```bash
$ sudo dnf install dconf-editor
$ sudo -u gdm dconf-editor
```
显示如下界面：
![dconf-editor]({{ site.url }}/assets/LinuxGDMScaleFactor/dconf-editor.png "dconf-editor")
接下去按照路径 /org/gnome/desktop/gnome/interface 进入，下拉滚动条找到 scaling-factor 选项，修改为1
![dconf-editor]({{ site.url }}/assets/LinuxGDMScaleFactor/dconf-editor-scale-factor.png "dconf-editor-scale-factor")
此时重启系统，你会发现登录界面再也不是那么丑大！！！
此时再看一下界面：
![screen]({{ site.url }}/assets/LinuxGDMScaleFactor/screen.png "screen")
是不是很细腻呢～
###### 注意： dconf-editor还可以修改GDM的GTK主题、图标主题、光标主题，背景。

### 参考
[GDM wiki]({{ site.url }}/assets/https://wiki.archlinux.org/index.php/GDM)