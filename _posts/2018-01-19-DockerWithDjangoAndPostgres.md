---
layout: page
title: Docker 运行 Django 和 PostgreSQL
date: 2018-01-19 17:24:23
tags:
 - docker
 - django
 - postgres
---

> Docker 真是个神器啊。都已经火了好几年了，我现在才开始用，可惜了我的青春。既然是新手，就应该做一些笔记。

## 项目所需配置文件

- Dockerfile

```
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/
```

- docker-compose.yml

```
version: "3"
services:
  db:
    image: postgres
    ports:
      - "5432:5432"
  web:
    build: .
    command: python3 manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    links:
      - db
```

- requirements.txt(可能本来就有)

```
django
psycopg2
...
```

## Docker 运行 Django

### 对于已有 Django 项目

如果拥有一个已经存在的 Django 项目，可以在项目根目录添加 Docker 相关的配置文件

启动项目

```python
docker-compose build
docker-compose up
```

### 对于未创建 Django 项目

如果还没有创建项目，则在需要创建项目的路径下创建目录作为 Docker 项目。

```
mkdir djangoproject
cd djangoproject
```

然后添加以上三个配置文件，并补充相应内容

```
touch Docker
touch docker-compose.yml
touch requirements.txt
```

创建 Django 项目并启动

```
docker-compose run web django-admin.py startproject django_demo .
```

启动后打开浏览器访问 localhost:8000 是否显示 it works 了呢

## Docker 运行 PostgreSQL

运行 Django 已经启动了 PostgreSQL，而且代理了 5432 端口，但是需要在 Django 中进行配置以连接上

Django 的 settings.py 修改 DATABASES 里面使其使用 PostgreSQL

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

然后重启 Django。

```
docker-compose run web python manage.py migrate
docker-compose up
```

## Docker 运行 PgAdmin4

从 Docker 镜像中拉取 pgadmin4，启动 pgadmin4 并将 9000 端口映射到 docker 的 80 端口

```
docker pull dpage/pgadmin4
docker run -p 9000:80 \
-e "PGADMIN_DEFAULT_EMAIL=user@domain.com" \
-e "PGADMIN_DEFAULT_PASSWORD=SuperSecret" \
-d dpage/pgadmin4
```

打开浏览器访问 PgAdmin4

```
http://127.0.0.1:9000/
```

登陆邮箱为启动命令中的 `PGADMIN_DEFAULT_EMAIL`

登陆密码为启动命令中的 `PGADMIN_DEFAULT_PASSWORD`
