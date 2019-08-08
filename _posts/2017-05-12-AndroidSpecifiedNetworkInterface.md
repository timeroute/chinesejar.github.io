---
layout: page
title: Android 指定网络接口收发 Socket 数据
date: 2017-05-12 20:47:40
tags:
- Android
- Socket
- Java
---

> 上次写了一篇 Android Ping IPv6 地址的教程，这个工作的目的就是想通过 Volte 网络发送 SIP 消息。但是 Android 默认的网络环境是 Wifi、2G/3G/4G， 不会默认通过 Volte 网络发送数据。因此需要通过一些方法来指定网络接口。

### 指定网络接口

首先介绍下 **NetworkInterface** 这个类

> NetworkInterface 这个类表示由名称组成的网络接口和分配给这些网络接口的 IP 地址列表。用于标识所在多播组的本地接口。

因此，Android 获取所有网络接口就可以通过 NetworkInterface 的 getNetworkInterfaces() 、 getInetAddress() 这个方法来实现

- getNetworkInterfaces() 方法返回本机上的所有接口。枚举至少包含一个元素，可能只显示了一个本地回环接口。
- getInetAddress() 方法返回绑定某个网络接口下的所有 IP 地址。

NetworkInterface.getNetworkInterfaces() 在调试中显示的结果如下
![networkinterfaces]({{ site.url }}/assets/AndroidSpecifiedNetworkInterface/networkinterfaces.png)

图中可以看出所有的网口列表，展开第一个显示了它的一些属性。其中 name 、 interfaceIndex 都是网络接口的唯一标识符号。
此时，如果我们需要指定网络接口，就可以根据它的属性来实现。

例子：获取 rmnet_data7 网口的 IPv6 地址

```
// 获取所有网口
List<NetworkInterface> interfaces = Collections.list(NetworkInterface.getNetworkInterfaces());
for(NetworkInterface iface : interfaces){
  if(iface.getDisplayName().equals("rmnet_data7")){ // 判断网口名称
    Enumeration<InetAddress> nifAddresses = iface.getInetAddresses();
    // 遍历 rmnet_data7 下所有 IP 地址
    while(nifAddresses.hasMoreElements()){
      InetAddress ni = nifAddresses.nextElement();
      Log.i("interface",ni.toString());
    }
  }
}
```

运行可以看到 log 中打印了 rmnet_data7 的 IP 地址

### 测试指定网口

接下来看看数据到底是不是通过这个网络接口发送出去了。
我通过代码建立了一个 TCP 连接，然后发送数据给 SIP 服务器，由于这不是重点，我就不贴代码了。
运行程序，通过 **tcpdump** 抓包后显示发送成功，并抓到了返回数据

![wireshark]({{ site.url }}/assets/AndroidSpecifiedNetworkInterface/wireshark.png)

黑色标记行为发送数据， 橙色标记行为接收数据。

