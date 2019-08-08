---
layout: page
title: 定制我的 Ubuntu 17.04
date: 2017-06-14 09:40:51
tags:
- linux
- gnome
- ubuntu
---

> 来来回回装了 N 次系统了，每次都要重装一大堆的工具，这次就想写这篇记录一下，以便下次再安装不用挨个找，顺便给其他 Linux 用户参考，如果他们需要的话。对于界面美化，我也是会讲究的，一款好的界面会让用户更喜爱他，不是麼 ;)

文章分 **工具篇** 和 **界面美化篇**

## 工具篇
### 科学上网工具
我想这是首先必须的。好多人没有它不能做任何事情。通过 PPA 源安装，支持 Ubuntu 14.04 或更高版本。

```shell
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```

**[项目地址](https://github.com/shadowsocks/shadowsocks-qt5)**

### Google Chrome
1. 可访问 [Google Chrome 官网](https://www.google.com/chrome/browser/desktop/index.html) 进行下载（前提：科学上网工具）
2. 通过 PPA 源安装
    首先安装密钥
    ```shell
    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
    ```
    添加 PPA 源
    ```shell
    sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
    ```
    更新源
    ```shell
    sudo apt update
    ```
    安装 stable 、 Beta 或者 Unstable 版本
    ```shell
    sudo apt-get install google-chrome-stable
    sudo apt-get install google-chrome-beta
    sudo apt-get install google-chrome-unstable
    ```

### Oh-My-ZSH
这是一个 Terminal 框架，不仅提供丰富的外观主题，还提供了丰富的功能如更强大、智能的自动补全等。需要预先安装 [zsh](https://wiki.archlinux.org/index.php/Zsh_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 工具。

```shell
sudo apt install zsh
```

一键安装 oh-my-zsh 命令

```shell
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

![on-my-zsh]({{ site.url }}/assets/BeautyYouUbuntu17-04/oh-my-zsh.png "oh-my-zsh")

相关配置信息、主题请访问 **[项目地址](https://github.com/robbyrussell/oh-my-zsh)** 

### Node.js
安装 **6.x** 版本
```shell
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```
安装 **8.x** 版本
```shell
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```

#### npm 更换淘宝镜像
```shell
npm config set registry https://registry.npm.taobao.org/
```

### Typora
这是一款 MarkDown 编辑器，功能也很强大。

```shell
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
sudo add-apt-repository 'deb https://typora.io ./linux/'
sudo apt-get update
sudo apt-get install typora
```

## 界面美化篇
对桌面环境来说，最喜欢 GNOME 了，它的高扩展性提供了丰富的定制功能。Github 上好多针对 GNOME 优化的项目，何况 Ubuntu 都放弃 Unity 了。
### 桌面主题
Github 上一大堆，我用比较热门的一款叫 Arc-Theme 的主题，包括了 GNOME Shell 主题和 GTK 主题

安装前需要以下依赖包（直接使用 `sudo apt install ` 安装即可）：
- autoconf
- automake
- pkg-config
- libgtk-3-dev
- git

#### 安装步骤

1. 从 Github 上获取项目
    ```shell
    git clone https://github.com/horst3180/arc-theme --depth 1 && cd arc-theme
    ```
2. 构建项目并安装
    ```shell
    ./autogen.sh --prefix=/usr
    sudo make install
    ```
3. 其他选项
    ```shell
    --disable-transparency     在 GTK3 主题中禁用透明度
    --disable-light            禁用 Arc Light
    --disable-darker           禁用 Arc Darker
    --disable-dark             禁用 Arc Dark
    --disable-cinnamon         禁用 Cinnamon
    --disable-gnome-shell      禁用 GNOME Shell
    --disable-gtk2             禁用 GTK2
    --disable-gtk3             禁用 GTK3
    --disable-metacity         禁用 Metacity
    --disable-unity            禁用 Unity
    --disable-xfwm             禁用 XFWM

    --with-gnome=<version>     为特定的 GNOME 版本 (3.14, 3.16, 3.18, 3.20, 3.22) 构建主题
    ```
4. 选择主题

    安装完成后打开 GNOME Tweak Tool 工具选择对应的 Arc 主题即可。

**注意** :对于高分屏，可能使用 Arc-Theme 显示 GNOME Shell 的字体过小，可通过修改 `/usr/share/themes/[对应 Arc 主题]/gnome-shell/gnome-shell.css` 修改 **stage** 的 `font-size` 。

**[项目地址](https://github.com/horst3180/arc-theme)** ，其他热门主题 [Numix](https://github.com/snwh/paper-gtk-theme) 、 [Paper](https://github.com/numixproject/numix-gtk-theme) 

#### 图标主题

**Numix**

```shell
sudo add-apt-repository ppa:numix/ppa
sudo apt-get update
sudo apt-get install numix-icon-theme
```
**[项目地址](https://github.com/numixproject/numix-icon-theme)**

**Paper**

```shell
sudo add-apt-repository ppa:snwh/pulp
sudo apt-get update
sudo apt-get install paper-icon-theme
# 同时也可以安装 GTK 和 Cursor 主题
sudo apt-get install paper-gtk-theme
sudo apt-get install paper-cursor-theme
```
**[项目地址](https://github.com/snwh/paper-icon-theme)**

**Papirus**

```shell
sudo add-apt-repository ppa:papirus/papirus
sudo apt-get update
sudo apt-get install papirus-icon-theme
```
或者下载最新的 [deb 安装包](https://launchpad.net/~papirus/+archive/ubuntu/papirus/+packages?field.name_filter=papirus-icon-theme) .

**[项目地址](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme)**


### GNOME Shell Extensions
- **[Weather](https://extensions.gnome.org/extension/613/weather/)** 天气插件

    ![weather]({{ site.url }}/assets/BeautyYouUbuntu17-04/weather.png "weather")
- **[System Monitor](https://extensions.gnome.org/extension/1064/system-monitor/)** 系统监视器

- **[Topicons Plus](https://extensions.gnome.org/extension/1031/topicons/)** 任务图标栏

    任务图标栏使用默认的图标，如何让他使用自定义的图标主题呢？

    比如使用 **Papirus** ，它支持 hardcode-tray 脚本来实现

    1. 安装 hardcode-tray

    ```shell
    sudo add-apt-repository ppa:andreas-angerer89/sni-qt-patched
    sudo apt update
    sudo apt install sni-qt sni-qt:i386 hardcode-tray
    ```

    2. 转换图标

    ```shell
    hardcode-tray --conversion-tool Inkscape
    ```


- **[Nvidia GPU Temperature Indicator](https://extensions.gnome.org/extension/541/nvidia-gpu-temperature-indicator/)** 显卡温度指示器

    以上三个的界面效果

    ![extensions]({{ site.url }}/assets/BeautyYouUbuntu17-04/extensions.png "extensions")
- **[Dash To Dock](https://extensions.gnome.org/extension/307/dash-to-dock/)** 可定制的 Dock

    可选择主题，停靠位置，个人习惯喜欢停靠在底部

    ![dock]({{ site.url }}/assets/BeautyYouUbuntu17-04/dock.png "dock")

## 结束

GNOME 可以定制的还有更多，可通过项目地址进去仔细研究。 ;)

顺便晒一下我的桌面

![desktop]({{ site.url }}/assets/BeautyYouUbuntu17-04/desktop.png "desktop")

