---
layout: page
title: 一个关于 Django Rest Framework 的梦
date: 2017-07-06 16:00:05
tags:
- Django Rest Framework
- 梦
---

Django Rest Framework 这个框架的相关资料确实少，碰到问题比较难以解决，连睡觉都在思考解决的问题。

最近有一个关于 Django Rest Framework 的问题一直困扰我，在使用视图集 ViewSet 的时候，有时候序列化模型的读写不一定完全一样，这时候该如何输出呢？

所谓日有所思，夜有所梦。昨晚做了一个奇怪的梦。

梦的大概是这样的：

> 我发现了 Django Rest Framework 在 Response 的时候可以定义一个 Response Serializer Model，这样就可以省去一些不必要显示的字段。

我将信将疑，今天进它的官网查找了一下到底有没有这个 API，发现确实.......果然.........没有。

不过按这个思路还是给了我解决方法，可能这个解决方法有些糙。

比如用户信息的注册和获取，包含以下字段：

- username
- password

注册的时候需要 username 、 password，然而获取的时候只需要 username 不需要 password，而 serializer 的 fields 又是定义了 username 、 password，这样很矛盾。

解决方法是创建了一个额外的 serializer 模型，fields 里包含 username 字段。然后重写 create 方法，在 Response 之前将原先的序列化模型转换成自定义的序列化模型，然后 Response。

### 具体解决方法

定义两个 `User` 序列化模型，一个用于注册，一个用于显示

```python
class RegisterSerializer(serializers.ModelSerializer):

    class Meta:
        model = User
        fields = ('id', 'username', 'password')

class DisplaySerializer(serializers.ModelSerializer):

    class Meta:
        model = User
        fields = ('id', 'username')
```

重写 create 方法，注册后返回用户的基本信息，而不包含密码

```python
def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        headers = self.get_success_headers(serializer.data)
        display_serializer = DisplaySerializer(data=serializer.data)
        display_serializer.is_valid(raise_exception=False)
        return Response(droider_serializer.data, status=status.HTTP_201_CREATED, headers=headers)
```

display_serializer 的 `is_valid()` 方法里 raise_exception 必须为 `False`， 因为 `self.perform_create()` 方法已经创建了用户，所以再次验证的时候会抛出 **用户已存在** 的异常，所以用 `False` 忽略这个异常。

### 返回结果

POST data
```
{
    "username": "username",
    "password": "password"
}
```

response data
```
{
    "username": "username"
}
```

其他方法也同理。
