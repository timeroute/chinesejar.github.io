---
layout: page
title: Docker 运行 Scrapy 爬虫
date: 2018-01-21 16:01:45
tags:
- Docker
- Scrapy
---

`Docker` 运行 `Scrapy` 与运行 `Django` 的方式几乎一致，首先创建 `Scrapy` 项目，或者不想在本机上安装 `Scrapy` 的话，可以创建一个空的项目，然后使用 `Docker` 运行 `Scrapy` 创建项目的命令。

在项目中配置如下文件：

#### Dockerfile

```
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
COPY pip.conf /etc/pip.conf
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/
```

#### docker-compose.yml

配置文件中的 `command` 为 `scrapy` 项目启动的命令。
如果在你只想启动 `scrapy` 项目中创建的一个 `name` 为 `demo` 的 `spide` 类，则在配置文件中的命令如下。

```
version: "3"
services:

  web:
    build: .
    command: scrapy crawl demo
    volumes:
      - .:/code
```

#### requirements.txt

此文件为 `scrapy` 项目中所用到的库，如果你喜欢用 `BeatifulSoup` 则自行添加

```
scrapy
...
```

#### pip.conf

此文件为 `pip` 的配置文件，由于国内网络的原因，使用此文件可将 `pip` 源改为清华的源。速度快~

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

### 启动

执行以下命令即可启动 `scrapy` 程序，是不是很简单呢？

```
docker-compose up
```
