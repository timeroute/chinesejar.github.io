---
layout: page
title: 一个简单有趣的微信聊天机器人
date: 2017-06-26 12:16:54
tags:
- Wechat
- Python
---

> 微信已经成了中国人生活中基本的通讯工具(除了那些自由开源人士以外)，前两天发现微信机器人的项目，其实早就有了。想着自己也做一个吧，顺便加了一些小小的功能。

## 释放我的机器人

微信扫一扫加他，跟他尬聊吧，把他拽到群里调戏他。

![机器人](MakeAWechatBot/qrcode.jpg)

**具体功能下面会介绍。**

## 工具

- 手机

    微信登陆必须得有手机端登陆才能使用网页登陆，因为要扫一扫
- Python 平台

    该项目基于 Python 开发，所以至少得来个嵌入式的开发板，或者电脑，或者...云服务器 ;-)，如果要保证长时间开启与话，最好使用云服务器。

## 开发微信机器人

该项目基于 Github 上的 [wxpy](https://github.com/youfou/wxpy)，使用文档在 [这里](http://wxpy.readthedocs.io/zh/latest/)。中文版的，所以我就不介绍这个怎么使用了。简单描述一下

### 创建机器人

```python
from wxpy import *
bot = Bot()
```

### 注册消息回复

机器人对好友、群聊中 `at` 他的人进行回复，在群聊中同时统计每个人的发言次数和第一次发言的时间，将这些信息实时存储在本地，以防程序错误导致数据丢失。

消息回复中的机器人使用 [图灵机器人](http://www.tuling123.com), 可免费申请 API，调用他。也可以使用 [小 I 机器人](http://cloud.xiaoi.com/)。这两个都是深度整合在项目里的。

```python
@bot.register([Friend, Group])
def reply_friend(msg):
    """
    消息自动回复
    """
    print(msg)
    if isinstance(msg.chat, Group):
        group = msg.chat.name
        name = msg.member.name
        if group in stat:
            if name in stat[group]['count']:
                stat[group]['count'][name] += 1
            else:
                stat[group]['count'][name] = 1
            flag = True
            for rank in stat[group]['rank']:
                if name == rank['name']:
                    flag = False
                    break
            if flag:
                stat[group]['rank'].append({'name': name, 'time': time.strftime("%H:%M:%S", time.localtime())})
        else:
            stat[group] = {"count": {name: 1}, 'rank': [{'name': name, 'time': time.strftime("%H:%M:%S", time.localtime())}, ]}
        if msg.text == "发言排行榜":
            g = bot.groups().search(group)[0]
            if not stat[g.name]:
                return
            msg_text = ""
            index = 1
            count = stat[g.name]['count']
            for name in sorted(count, key=lambda x: count[x], reverse=True):
                # print("{}: {} {}".format(index, rank['name'], rank['time']))
                msg_text += "{}: {} 发言了 {} 次\n".format(index, name, count[name])
                index += 1
            if msg_text:
                msg_text = "发言排行榜：\n" + msg_text
                g.send(msg_text)
        if msg.text == "起床排行榜":
            g = bot.groups().search(group)[0]
            if not stat[g.name]:
                return
            msg_text = ""
            index = 1
            for rank in stat[g.name]['rank']:
                # print("{}: {} {}".format(index, rank['name'], rank['time']))
                msg_text += "{}: {} {}\n".format(index, rank['name'], rank['time'])
                index += 1
            if msg_text:
                msg_text = "起床排行榜：\n" + msg_text
                g.send(msg_text)
        with open('stat.json', 'w') as fh:
            fh.write(json.dumps(stat))
        if not msg.is_at:
            return
    return tuling_auto_reply(msg)
```

### 自动接受好友申请

```python
@bot.register(msg_types=FRIENDS)
def auto_accept_friends(msg):
    """
    自动接受好友请求
    """
    # 接受好友请求
    new_friend = msg.card.accept()
    # 向新的好友发送消息
    new_friend.send('哈哈，我们现在是超级好的好朋友了呢～～')
```

## 添加计划任务

光回复怎么够，还要做一些小小的有趣的功能，我这里添加了两个统计，一个是起床时间统计，另一个是发言统计。

当天群聊的用户第一次发言作为起床时间，虽然有些不严谨，但毕竟功能是受限制的。

然后每天的 9 点发布一次起床排行榜， 20 点发布一次发言排行榜。当然其实主动发送 “**起床排行榜**”、“**发言排行榜**” 也会回复当前的排行。

### 起床排行榜

![起床排行榜]({{ site.url }}/assets/MakeAWechatBot/rank_getup.jpg)

### 发言排行榜

![发言排行榜]({{ site.url }}/assets/MakeAWechatBot/rank_speak.jpg)


### 实现

```python
class ScheduleThread(threading.Thread):
    """
    计划任务线程
    """
    def run(self):
        global schedule_time
        global bot
        global stat
        while 1:
            time.sleep(300)
            cur_hour = time.strftime("%H", time.localtime())
            # print("cur:{}\tschedule:{}".format(cur_hour, schedule_time))
            if cur_hour == schedule_time:
                continue
            elif cur_hour == '09':
                for group in bot.groups():
                    print(group.name)
                    if not stat[group.name]:
                        continue
                    msg_text = ""
                    index = 1
                    for rank in stat[group.name]['rank']:
                        # print("{}: {} {}".format(index, rank['name'], rank['time']))
                        msg_text += "{}: {} {}\n".format(index, rank['name'], rank['time'])
                        index += 1
                    if msg_text:
                        msg_text = "排行日报\n起床排行榜：\n" + msg_text
                        group.send(msg_text)
            elif cur_hour == '20':
                for group in bot.groups():
                    print(group.name)
                    if not stat[group.name]:
                        continue
                    msg_text = ""
                    index = 1
                    count = stat[group.name]['count']
                    for name in sorted(count, key=lambda x: count[x], reverse=True):
                        # print("{}: {} {}".format(index, rank['name'], rank['time']))
                        msg_text += "{}: {} 发言了 {} 次\n".format(index, name, count[name])
                        index += 1
                    if msg_text:
                        msg_text = "排行日报\n发言排行榜：\n" + msg_text
                        group.send(msg_text)
            elif cur_hour == '00':
                stat = dict()
                with open('stat.json', 'w') as fh:
                    fh.write('')
            schedule_time = cur_hour
```

# 聊聊

展示两个机器人互相尬聊的情况是怎么样的。
![聊天]({{ site.url }}/assets/MakeAWechatBot/chat.jpg)

## 部署

创建机器人时添加一个 `console_qr` 参数， `True` 时表示在终端显示二维码，`False` 表示用图片程序打开二维码。按情况来，如果在没有界面的云服务器上，那就在终端打开，如果只能连 tty ，那最好的办法就是生成一张图片，放到指定的 FTP 或者云盘目录，然后本地打开扫描，或者建个简单的 HTTP 服务器展示图片，方法很多，根据自己情况来吧。
