---
layout: page
title: 在 CentOS 7 上搭建 Redis 集群
date: 2018-06-06 14:30:29
tags:
- Redis
- CentOS
---

> 考虑到后期可能使用到 Redis 集群，参考了 Google 搜索的一些资料，自己整理了一下 Redis 集群搭建过程。

## 安装环境

### Ruby

redis-trib 是 Redis 官方推出的管理 Redis 集群的工具，使用 Ruby 实现。如果是编译最新版的 Redis，需要相应版本的 Ruby 环境。

- 版本兼容的情况下，直接通过 yum 安装

```
yum install ruby
```

- 版本不兼容的情况下，可通过 [RVM](http://rvm.io/) 工具安装。

```shell
# 安装依赖
yum install gcc-c++ patch readline readline-devel zlib zlib-devel \
    libyaml-devel libffi-devel openssl-devel make \
    bzip2 autoconf automake libtool bison iconv-devel sqlite-devel \

# 导入 GPG
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

# 安装 RVM
curl -sSL https://get.rvm.io | bash -s stable

# 加载 RVM 环境
source /etc/profile.d/rvm.sh
rvm reload

# 验证依赖
rvm requirements run
# 安装 Ruby
rvm install 2.3.3
# 设置默认版本
rvm use 2.3.5 --default
# 验证当前版本
ruby --version
# ruby 2.3.5p376 (2017-09-14 revision 59905) [x86_64-linux]
```

### Redis

```
yum install redis redis-trib
```

### Ruby 与 Redis 接口

```
gem install redis
```

## 搭建集群

- 创建集群的目录 redis-cluster，然后在其里面分别创建 6 个目录：

```
mkdir -p /data/redis-cluster/9001/
...
mkdir -p /data/redis-cluster/9006/
```

- 将 /etc/redis.conf 文件复制到每个新建的目录里面：

```
cp /etc/redis.conf /data/redis-cluster/9001/
...
cp /etc/redis.conf /data/redis-cluster/9006/
```

- 修改每一个复制的 redis.conf 配置里面的参数

```shell
# 后台运行
daemonize yes
# 端口 (900* 表示端口号一一对应)
port 900*
# 绑定 IP，只接受当前 IP，如果要接受全部 IP 则配置为 0.0.0.0
bind 192.168.1.83
# 指定数据文件存放位置，对应每一个新建的目录下
dir /data/redis-cluster/900*/
# 启动集群模式
cluster-enabled yes
# 集群配置文件，自动创建(900* 表示端口号一一对应)
cluster-config-file node-900*.conf
# 集群超时时间
cluster-node-timeout 5000
# 开启 aof 持久化模式
appendonly yes
# 每次有写操作的时候都同步
appendfsync always
```

- 启动 6 个 Redis 实例

```
redis-server /data/redis-cluster/9001/redis.conf
...
redis-server /data/redis-cluster/9006/redis.conf
```

- 将 6 个 Redis 实例加入到集群

```
redis-trib create --replicas 1 192.168.1.83:9001 192.168.1.83:9002 192.168.1.83:9003 192.168.1.83:9004 192.168.1.83:9005 192.168.1.83:9006
```

创建集群成功

```
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.1.83:9001
192.168.1.83:9002
192.168.1.83:9003
Adding replica 192.168.1.83:9004 to 192.168.1.83:9001
Adding replica 192.168.1.83:9005 to 192.168.1.83:9002
Adding replica 192.168.1.83:9006 to 192.168.1.83:9003
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 192.168.1.83:9001)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

如果出现 `ERR Slot 16011 is already busy` 错误，则是由于上一次配置集群失败留下的配置信息导致的。只要删除每个节点目录里的 node-900*.conf 文件删除并重新启动 redis-server 即可。

- 连接任意一台客户端进行验证，并查看集群信息、节点信息

```
redis-cli -c -h 192.168.1.83 -p 9001

192.168.1.83:9001> CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_sent:17000
cluster_stats_messages_received:17000

192.168.1.83:9001> CLUSTER NODES
cf427dffcc0355bde3720ac7119cbff2eac45633 192.168.1.83:9002 master - 0 1528270078637 2 connected 5461-10922
31feb2be2955c12a29052232c8334c542581a30f 192.168.1.83:9003 master - 0 1528270079651 3 connected 10923-16383
c01c9dabfcbd39e8c760a258a2ccb39b322a1970 192.168.1.83:9001 myself,master - 0 0 1 connected 0-5460
08034c8a4d995b002ce559b275606ffffe896a0b 192.168.1.83:9004 slave c01c9dabfcbd39e8c760a258a2ccb39b322a1970 0 1528270079143 4 connected
c513bf1ca862a4bdaaa3ea8d95679950d6ae8e26 192.168.1.83:9006 slave 31feb2be2955c12a29052232c8334c542581a30f 0 1528270078233 6 connected
0cae84633230a7d033888c9adf0c8c4487f71938 192.168.1.83:9005 slave cf427dffcc0355bde3720ac7119cbff2eac45633 0 1528270078637 5 connected
```
