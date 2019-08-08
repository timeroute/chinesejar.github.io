---
layout: page
title: Docker 部署 MongoDB 和 RabbitMQ 的集群
date: 2018-06-08 13:59:06
tags:
---

> 备份一下 Docker 上 MongoDB 和 RabbitMQ 集群的简短教程。

[](#MongoDB-集群 "MongoDB 集群")MongoDB 集群
--------------------------------------

### [](#说明 "说明")说明

使用 Docker 官方的 MongoDB 镜像启动 3 个容器，一个主服务器两个备份服务器。

### [](#配置文件 "配置文件")配置文件

# docker-compose.yml  
```  
version: '3'  
services:  
 mongo1:  
 image: mongo  
 container_name: "mongo1"  
 ports:  
 - "30001:27017"  
 volumes:  
 - /data/mongodb/mongo1:/data/db  
 command: mongod --dbpath /data/db --replSet mongoreplset  
 mongo2:  
 image: mongo  
 container_name: "mongo2"  
 ports:  
 - "30002:27017"  
 volumes:  
 - /data/mongodb/mongo2:/data/db  
 command: mongod --dbpath /data/db --replSet mongoreplset  
 mongo3:  
 image: mongo  
 container_name: "mongo3"  
 ports:  
 - "30003:27017"  
 volumes:  
 - /data/mongodb/mongo3:/data/db  
 command: mongod --dbpath /data/db --replSet mongoreplset  
```

### [](#初始化副本集 "初始化副本集")初始化副本集

```
docker exec -it mongo1 mongo  

> rs.initiate() \# 初始化  
> rs.add('mongo2:27017') \# 加入 备份服务器 mongo2  
> rs.add('mongo3:27017') \# 加入 备份服务器 mongo3  
> rs.status() \# 查看状态  
```

[](#RabbitMQ-集群 "RabbitMQ 集群")RabbitMQ 集群
-----------------------------------------

### [](#说明-1 "说明")说明

使用 bijukunjummen 的 RabbitMQ 镜像启动 3 个容器。

### [](#配置文件-1 "配置文件")配置文件

# docker-compose.yml

```
version: '3'  
services:  
 rabbit1:  
 image: bijukunjummen/rabbitmq-server  
 hostname: rabbit1  
 ports:  
 - "5672:5672"  
 - "15672:15672"  
 environment:  
 - RABBITMQ\_DEFAULT\_USER=myuser  
 - RABBITMQ\_DEFAULT\_PASS=mypass  
 rabbit2:  
 image: bijukunjummen/rabbitmq-server  
 hostname: rabbit2  
 links:  
 - rabbit1  
 environment:  
 - CLUSTERED=true  
 - CLUSTER_WITH=rabbit1  
 - RAM_NODE=true  
 ports:  
 - "5673:5672"  
 - "15673:15672"  
  
 rabbit3:  
 image: bijukunjummen/rabbitmq-server  
 hostname: rabbit3  
 links:  
 - rabbit1  
 - rabbit2  
 environment:  
 - CLUSTERED=true  
 - CLUSTER_WITH=rabbit1  
 ports:  
 - "5674:5672"  
```

[](#验证 "验证")验证
==============

通过浏览器访问 `http://<服务器 IP>:15672`，帐号和密码默认都是 `guest`，即可在 `Nodes` 看到三个节点。

*   [docker](/tags/docker/)
*   [mongodb](/tags/mongodb/)
*   [rabbitmq](/tags/rabbitmq/)
