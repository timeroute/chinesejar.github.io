---
layout: page
title: Ubuntu 17.04 安装 VMware Workstation 12 错误解决
date: 2017-04-26 16:42:37
tags:
- ubuntu
- vmware
- linux
---

## 问题 1

VMware 在 Linux 系统上首次安装或者更新内核后会安装或重新编译 vmnet 与 vmmon 这两个模块。
在 Ubuntu 17.04 上可能会碰到无法安装 vmnet 和 vmmon 的情况，解决方法如下：

```
$ sudo su

# cd /usr/lib/vmware/modules/source
# tar xf vmmon.tar
# mv vmmon.tar vmmon.old.tar
# sed -i 's/uvAddr, numPages, 0, 0/uvAddr, numPages, 0/g' vmmon-only/linux/hostif.c
# tar cf vmmon.tar vmmon-only
# rm -r vmmon-only

# tar xf vmnet.tar
# mv vmnet.tar vmnet.old.tar
# sed -i 's/addr, 1, 1, 0/addr, 1, 0/g' vmnet-only/userif.c
# tar cf vmnet.tar vmnet-only
# rm -r vmnet-only
```

## 问题 2

另外，附上解决 OSX 虚拟机的工具， **Windows**、**Linux**、**OSX** 都支持。

[unlock工具](https://pan.baidu.com/s/1eS3kwJg)

解压后直接执行对应脚本即可。

## 问题 3

vmware 配置文件无法保存的解决方法，是用户目录下 `.vmware` 目录权限问题

```
$ sudo chown -R username:username ~/.vmware
```

**注意**： `username` 为用户名。

## 问题 4

序列号

- `5A02H-AU243-TZJ49-GTC7K-3C61N`
- `VF5XA-FNDDJ-085GZ-4NXZ9-N20E6`
- `UC5MR-8NE16-H81WY-R7QGV-QG2D8`
- `ZG1WH-ATY96-H80QP-X7PEX-Y30V4`
- `AA3E0-0VDE1-0893Z-KGZ59-QGAVF`
