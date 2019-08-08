---
layout: page
title: Docker 运行 Mysql
date: 2018-01-30 12:22:09
tags:
- docker
- mysql
---

> 本篇作为学习笔记记录。

### 将 Mysql 镜像下拉到本地

```
docker pull mysql
```

### 创建指定存放的数据路径

```
mkdir -p /mnt/mysql/conf
cd /mnt/mysql/conf
touch my.cnf
```

这样可以直接创建 mysql 目录和 mysql 目录里的 conf 目录，如果涉及权限请加 `sudo`

### 启动 docker

```
docker run -d -p 3306:3306 -v /mnt/mysql/conf/my.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf -v /mnt/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=gaofen123456 --name gaofensql mysql
```

| 参数 | 说明 |
| -- | -- |
| `-p` | 指定映射的端口 |
| `-v` | 指定映射的文件/路径 |
| `-e` | 环境变量 |

### 查看 mysql

```
docker ps
```

返回如下表示正在运行之中

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
b731b0361808        mysql               "docker-entrypoint.s…"   2 hours ago         Up 2 hours          0.0.0.0:3306->3306/tcp   gaofensql
```
