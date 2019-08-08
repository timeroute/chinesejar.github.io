---
layout: page
title: VMWARE WORKSTATION 14 出现内存不足的错误情况
date: 2018-01-22 16:02:13
tags:
- vmware
---

## 问题

当我们在 Fedora 27 平台上的 VMware Workstation 14 启动一台虚拟机时， 会出现如下错误，而且虚拟机并未能成功启动。

但是其实系统有足够的内存。

```
Not enough physical memory is available to power on this virtual machine with its configured settings.
```


## 解决方法

`vmmon` 是虚拟机的内核监控模块。你需要按如下方法修改源文件 `hostif.c` 并且重新编译内核模块:

```
sudo su
cd /tmp
cp -p /usr/lib/vmware/modules/source/vmmon.tar /usr/lib/vmware/modules/source/vmmon.tar~backup
cp /usr/lib/vmware/modules/source/vmmon.tar .
tar xf vmmon.tar
rm vmmon.tar
wget https://raw.githubusercontent.com/mkubecek/vmware-host-modules/fadedd9c8a4dd23f74da2b448572df95666dfe12/vmmon-only/linux/hostif.c
mv -f hostif.c vmmon-only/linux/hostif.c 
tar cf vmmon.tar vmmon-only
rm -rf vmmon-only
mv -f vmmon.tar /usr/lib/vmware/modules/source/vmmon.tar 
vmware-modconfig --console --install-all
VMware Workstation should now work and the virtual machine should now start the next time you start it.
```

## 参考来源

[VMware Workstation 14: “Error: Not enough physical memory is available to power on this virtual machine with its configured settings.”](http://blog.nowherelan.com/2017/12/04/vmware-workstation-14-error-not-enough-physical-memory-is-available-to-power-on-this-virtual-machine-with-its-configured-settings/)
