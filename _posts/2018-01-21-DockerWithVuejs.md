---
layout: page
title: Docker 运行 Vuejs
date: 2018-01-21 20:42:57
tags:
- Docker
- Node
- Vue
---

> 作为 Docker 新手，总该为自己做一些入门笔记，毕竟还不熟悉，打开自己的笔记就可以立马部署了。

现在再来讲一下 Docker 运行 Vuejs 项目的方法。

### pull node

将最新版的 docker with node 环境拉下来

```
docker pull node
```

### 配置两个 Docker 配置文件

#### Dockerfile

无论 `npm` 有没配置国内源， `npm install` 过程中都需要下载烦人的 `chromedriver`，而它又无法通过国内源，所以建议在本地通过代理 `npm install` 后，将整个项目 `ADD` 到容器里去。

```dockerfile
FROM node:8
RUN mkdir /code
WORKDIR /code
ADD . /code/
```

#### docker-compose.yml

```
version: "3"
services:
  web:
    build: .
    command: "npm run dev"
    volumes:
      - .:/code
    ports:
      - "8080:8080"
```

### 修改 Vue 项目中的 HOST

默认情况下，启动 `npm run dev` 的地址为 `localhost:8080`，而这种情况通过端口映射对于容器外的网络是不起作用的，必须将 HOST 改为 `0.0.0.0`

修改 `build/webpack.dev.conf.js` 文件中的 HOST

```vue
/* const HOST = process.env.HSOT */
const HOST = "0.0.0.0"
```

### 启动

执行 `docker-compose up -d` 命令启动程序。

-d 表示在作为后台执行。

当然在生产环境中使用 `npm run dev` 是不严谨的，暂时还没开发完成，后续将 build 后的方法再记录一下。
