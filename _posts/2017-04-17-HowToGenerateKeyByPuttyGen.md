---
layout: page
title: 如何使用云服务器提供的ssh证书导入Putty免密码登录
date: 2017-04-17 21:37:03
tags:
- ssh
- putty
- linux
---

>PuTTY是一个Telnet、SSH、rlogin、纯TCP以及串行端口连接软件。较早的版本仅支持Windows平台，后陆续增加对各类Unix平台和Mac OSX的支持。除了官方版本外，有许多第三方的团体或个人将PuTTY移植到其他平台上，像是以Symbian为基础的移动电话。PuTTY为一开放源代码软件，主要由Simon Tatham维护，使用MIT许可证授权。


现在大部分云服务厂商提供了ssh证书的登录方式，这样极大的增加了安全性。当然用户也可以自行生成ssh密钥，对于Linux用户来说，最好的ssh客户端非Putty莫属，但是Putty对于ssh证书的格式有一定的要求，所以一般的ssh证书或者密钥都需要经过它自有的工具(Puttygen)转换才可以使用。

##### 方法
puttygen命令可以生成一个ssh证书或者从另一个证书生成适合putty的新证书。
puttygen使用方式如下：
```bash
$ puttygen -h
puttygen: Release 0.67
Usage: puttygen ( keyfile | -t type [ -b bits ] )
                [ -C comment ] [ -P ] [ -q ]
                [ -o output-keyfile ] [ -O type | -l | -L | -p ]
  -t    specify key type when generating (rsa, dsa, rsa1)
  -b    specify number of bits when generating key
  -C    change or specify key comment
  -P    change key passphrase
  -q    quiet: do not display progress bar
  -O    specify output type:
           private             output PuTTY private key format
           private-openssh     export OpenSSH private key
           private-sshcom      export ssh.com private key
           public              standard / ssh.com public key
           public-openssh      OpenSSH public key
           fingerprint         output the key fingerprint
  -o    specify output file
  -l    equivalent to `-O fingerprint'
  -L    equivalent to `-O public-openssh'
  -p    equivalent to `-O public
```
因此从云服务器上下载下来的ssh证书通过以下命令即可转换成putty的证书：
```bash
$ puttygen [file] -o [output_file]
```
然后打开putty将新生成的证书导入
![putty]({{ site.url }}/assets/HowToGenerateKeyByPuttyGen/putty.png "putty")
点击Broswer弹出证书导入对话框
![putty_key]({{ site.url }}/assets/HowToGenerateKeyByPuttyGen/putty_key.png "putty_key")
我的证书命名为authorized_keys，然后确定即可！

看！从此免密码登录，还安全！
![putty_login]({{ site.url }}/assets/HowToGenerateKeyByPuttyGen/putty_login.png "putty_login")