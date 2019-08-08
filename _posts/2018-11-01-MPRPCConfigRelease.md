---
layout: page
title: 发布一个基于 mprpc_config 二次封装的 pip 包
date: 2018-11-01 16:02:33
tags:
- Python
- RPC
---

[mprpc](https://github.com/studio-ousia/mprpc) 是一个超快速的Python RPC 库，最近一直在看 RPC 的东西，考虑如何应用到公司的项目中，模仿 Celery 的方式二次封装了一下。

项目地址： [mprpc_config](https://github.com/chinesejar/mprpc_config)

## 安装

```shell
pip install mprpc_config
```

## 用法

构建一个如下的目录结构

- config.py 包含RPC配置属性（任意命名）
  - INSTALLED_APP 必填参数，list 类型，将每一个包含 RPC 接口的模块名放进去
- app_module_name RPC 接口模块（任意命名，可有多个）
  - implement.py 接口模块（任意命名，可有多个），里面包含 rpc 方法装饰的接口函数
- server.py RPC 服务器
  1. 初始化 mprpc_config 的 `Configration` 类，将 `config.py` 的模块名 `config` 作为字符串传入
  2. 导入 mprpc_config.rpc_server 的 RPCServer 和 StreamServer，启动 RPC 服务器
  3. 使用 `bind_class` 方法将初始化后的所有 RPC 接口绑定给 `RPCServer`
  4. 使用 StreamServer 启动 RPC 服务器
- client.py RPC 客户端（测试用，实际可分离）
  
  客户端启动方式可参考 [mprpc](https://github.com/studio-ousia/mprpc) 。

## 示例

> 示例代码在 `example` 目录

server.py

```python
from mprpc_config.rpc_server import RPCServer, StreamServer
from mprpc_config import rpc_config


if __name__ == '__main__':
    print('-------start server--------')
    config = rpc_config.Configuration("config")
    config.bind_class(RPCServer)
    server = StreamServer(('127.0.0.1', 6000), RPCServer)
    server.serve_forever(stop_timeout=10)

```

client.py

```python
from mprpc_config.rpc_client import RPCClient, RPCPoolClient
from mprpc_config.rpc_client import RPCPool

print('---------- client ----------')
client = RPCClient('127.0.0.1', 6000)
print('2 add 4: ', client.call('add', 2, 4))
print('2 plus 4: ', client.call('plus', 2, 4))
print('2 minus 4: ', client.call('minus', 2, 4))
print('---------- done ----------')

print('---------- client pool ----------')
client_pool = RPCPool(RPCPoolClient, dict(host='127.0.0.1', port=6000))
with client_pool.connection() as client:
    print('2 add 4: ', client.call('add', 2, 4))
    print('2 plus 4: ', client.call('plus', 2, 4))
    print('2 minus 4: ', client.call('minus', 2, 4))
print('---------- done ----------')

```

启动 server 和 client

```shell
$ python server.py

---------- server ----------

$ python client.py

---------- client ----------
2 add 4:  6
2 plus 4:  8
2 minus 4:  -2
---------- done ----------
---------- client pool ----------
2 add 4:  6
2 plus 4:  8
2 minus 4:  -2
---------- done ----------
```