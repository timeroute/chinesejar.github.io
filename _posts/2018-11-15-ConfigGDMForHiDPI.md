---
layout: page
title: GDM 适配 HiDPI
date: 2018-11-15 18:35:46
tags:
- GNOME
- GDM
- HiDPI
---

话说 GNOME 跟 GDM 还不是一个东西。GNOME 的优化工具是非常好用啊～不过 GDM 下的优化真的不太方便。常年使用 2K 屏的人，其实也无所谓 GDM 下显示很大很丑，反正就一登录而已。

不过这么长时间来，终于想改一下，也能立马解决问题，所以还是记录一下吧。

```shell
sudo vi /usr/share/glib-2.0/schemas/org.gnome.desktop.interface.gschema.xml
```

打开后找到 key 为 `text-scaling-factor` 的参数，该参数表示字体显示的尺寸，最小是 0.5, 最大是 3.0，将 default 里的数字改为 1.4，个人认为在 2K 分辨率下是最好的大小。

再找到 key 为 `scaling-factor` 的参数，该参数表示界面显示的尺寸，将 default 为 1，就好看多了。

改完后执行使配置生效

```shell
sudo glib-compile-schemas /usr/share/glib-2.0/schemas
```

当然，改完是看不到任何变化的，注销后也看不到（亲测），那就重启一下吧。
