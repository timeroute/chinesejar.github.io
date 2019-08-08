---
layout: page
title: 美化我的 Fedora 28 Gnome 桌面环境
date: 2018-08-24 13:53:16
tags:
- Fedora
- GNOME
---

> 之前重装 Fedora 的时候 BOIS 设置成了 传统模式启动，导致开机的界面丑爆了。忍了很久，因为工作的原因，一直没有修整。最近一直想重装一下 Fedora，但是还是因为工作项目比较多，没时间备份，也没时间配置，先做个笔记记录以下目前系统的配置吧。

## 主题与图标

- Theme

  [Adapta](https://github.com/adapta-project/adapta-gtk-theme) 一个 Material Design 风格的 GTK 主题

- Icon

  [Papirus](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme) 很漂亮的一个 icon 主题，但是 Fedora Repo 速度很慢。

显示效果

![theme]({{ site.url }}/assets/BeautyMyFedora28GnomeDesktopEnvironment/theme.png)

![icon]({{ site.url }}/assets/BeautyMyFedora28GnomeDesktopEnvironment/icon.png)

## 字体与大小

说实话 GNOME 针对 2K 屏的优化不好啊，通过放大字体的方式能改善效果。

字体个人使用了 Roboto Regular，看起来很舒服。

![font]({{ site.url }}/assets/BeautyMyFedora28GnomeDesktopEnvironment/font.png)

## GNOME Shell Extensions

- [Cpu Power Manager](https://extensions.gnome.org/extension/945/cpu-power-manager/)
  CPU 性能管理器
- [Caffeine](https://extensions.gnome.org/extension/517/caffeine/)
  一款 启动 or 禁用 自动挂起与屏保的插件
- [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/)
  粘贴插件
- [Dash To Panel](https://extensions.gnome.org/extension/1160/dash-to-panel/)
  Dash 栏目与 Panel 板合并插件
- [NetSpeed](https://extensions.gnome.org/extension/104/netspeed/)
  网速监视器
- [System Monitor](https://extensions.gnome.org/extension/9/systemmonitor/)
  系统监视器
- [Topicons Plus](https://extensions.gnome.org/extension/1031/topicons/)
  运行程序图标指示器

显示效果

![extensions]({{ site.url }}/assets/BeautyMyFedora28GnomeDesktopEnvironment/extensions.png)

## ZSH

```shell
ZSH_THEME="powerlevel9k/powerlevel9k"

plugins=(
  git
  bundler
  dotenv
  osx
  rake
  rbenv
  ruby
)

POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(user dir rbenv vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status root_indicator background_jobs history time)
```

显示效果

![terminal]({{ site.url }}/assets/BeautyMyFedora28GnomeDesktopEnvironment/terminal.png)
