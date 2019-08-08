---
layout: page
title: Django Rest Framework 序列化关系模型
date: 2017-07-03 10:01:32
tags:
- Django
- Python
- REST
---

> 这两天一直在学习 Django Rest Framework 这个框架，这是一个非常流行的 REST API 框架，深度整合 Django。但与传统 MVC 模式的不同， Django REST Framework 在使用过程中，需要理解一些新的东西。结合官方 API 分享一下框架中关于序列化关系模型的理解。

## 序列化模型与序列化关系模型

序列化模型，顾名思义，即对 models 里的数据模型作序列化。而序列化关系模型则是对 models 里数据模型中带有关系的如 `ForeignKey`, `ManyToManyField` 和 `OneToOneField` 字段作序列化。Django Rest Framework 提供了灵活的序列化关系模型，让开发者可以自由定制序列化数据模型。

## 序列化关系模型

根据官方的例子来看一下每一个关系模型的介绍。

数据模型如下：
```python
class Album(models.Model):
    album_name = models.CharField(max_length=100)
    artist = models.CharField(max_length=100)

class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    order = models.IntegerField()
    title = models.CharField(max_length=100)
    duration = models.IntegerField()

    class Meta:
        unique_together = ('album', 'order')
        ordering = ['order']

    def __unicode__(self):
        return '%d: %s' % (self.order, self.title)
```

### StringRelatedField

使用 `StringRelatedField` 将返回一个对应关系 model 的 `__unicode__()` 方法的字符串。

这个字段是只读的。

参数：

- `many` 如果应用于多对多关系，则应将此参数设置为 `True`

序列化模型如下

```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.StringRelatedField(many=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```

序列化结果如下：

```javascript
{
    'album_name': 'Things We Lost In The Fire',
    'artist': 'Low',
    'tracks': [
        '1: Sunflower',
        '2: Whitetail',
        '3: Dinosaur Act',
        ...
    ]
}
```

### PrimaryKeyRelatedField

使用 `PrimaryKeyRelatedField` 将返回一个对应关系 model 的主键。

参数：

- `queryset` 用于在验证字段输入时模型实例查找。 关系必须明确设置 `queryset`，或设置 `read_only = True`
- `many` 如果是对应多个的关系，就设置为 `True`
- `allow_null` 如果设置为 `True`，则该字段将接受 `None` 的值或为空的关系的空字符串。默认为 `False`
- `pk_field` 设置为一个字段以控制主键值的序列化/反序列化。例如，`pk_field = UUIDField（format ='hex'）` 将UUID主键序列化为紧凑的十六进制表示。

序列化模型如下

```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.PrimaryKeyRelatedField(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```

序列化结果如下：

```javascript
{
    'album_name': 'Undun',
    'artist': 'The Roots',
    'tracks': [
        89,
        90,
        91,
        ...
    ]
}
```

### HyperlinkedRelatedField

使用 `HyperlinkedRelatedField` 将返回一个超链接，该链接指向对应关系 model 的详细数据，`view-name` 是必选参数，为对应的视图生成超链接。

参数：

- `view_name` 用作关系目标的视图名称。如果使用的是标准路由器类，那么它的格式为 `<modelname>-detail` 的字符串
- `queryset` 验证字段输入时用于模型实例查询的查询器。关系必须明确设置 `queryset`，或设置 `read_only = True`
- `many` 如果应用于多对多关系，则应将此参数设置为 `True`
- `allow_null` 如果设置为 `True`，则该字段将接受 `None` 的值或为空的关系的空字符串。默认为 `False`
- `lookup_field` 应该用于查找的目标上的字段。应该对应于引用视图上的 `URL` 关键字参数。默认值为 `pk`
- `lookup_url_kwarg` 与查找字段对应的 `URL conf` 中定义的关键字参数的名称。默认使用与 `lookup_field` 相同的值
- `format` 如果使用 `format` 后缀，超链接字段将对目标使用相同的 `format` 后缀，除非使用 `format` 参数进行覆盖。


序列化模型如下

```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.HyperlinkedRelatedField(
        many=True,
        read_only=True,
        view_name='track-detail'
    )

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```

序列化结果如下：

```javascript
{
    'album_name': 'Graceland',
    'artist': 'Paul Simon',
    'tracks': [
        'http://www.example.com/api/tracks/45/',
        'http://www.example.com/api/tracks/46/',
        'http://www.example.com/api/tracks/47/',
        ...
    ]
}
```

### SlugRelatedField

使用 `SlugRelatedField` 将返回一个指定对应关系 model 中的字段，需要擦参数 `slug_field` 中指定字段名称。

参数：

- `slug_field` 应该用于表示目标的字段。这应该是唯一标识任何给定实例的字段。例如 `username` 。这是必选参数
- `queryset` 验证字段输入时用于模型实例查询的查询器。 关系必须明确设置 `queryset`，或设置 `read_only = True`
- `many` 如果应用于多对多关系，则应将此参数设置为 `True`
- `allow_null` 如果设置为 `True`，则该字段将接受 `None` 的值或为空的关系的空字符串。默认为 `False`

序列化模型如下

```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.SlugRelatedField(
        many=True,
        read_only=True,
        slug_field='title'
     )

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```

序列化结果如下：

```javascript
{
    'album_name': 'Dear John',
    'artist': 'Loney Dear',
    'tracks': [
        'Airport Surroundings',
        'Everything Turns to You',
        'I Was Only Going Out',
        ...
    ]
}
```

### HyperlinkedIdentityField

使用 `HyperlinkedIdentityField` 将返回指定 `view-name` 的超链接的字段。

参数：

- `view_name` 应该用作关系目标的视图名称。如果您使用的是标准路由器类，则它将是格式为 `<model_name>-detail` 的字符串。必选参数
- `lookup_field` 应该用于查找的目标上的字段。应该对应于引用视图上的 `URL` 关键字参数。默认值为 `pk`
- `lookup_url_kwarg` 与查找字段对应的 `URL conf` 中定义的关键字参数的名称。默认使用与 `lookup_field` 相同的值
- `format` 如果使用 `format` 后缀，超链接字段将对目标使用相同的 `format` 后缀，除非使用 `format` 参数进行覆盖

序列化模型如下

```python
class AlbumSerializer(serializers.HyperlinkedModelSerializer):
    track_listing = serializers.HyperlinkedIdentityField(view_name='track-list')

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'track_listing')
```

序列化结果如下：

```javascript
{
    'album_name': 'The Eraser',
    'artist': 'Thom Yorke',
    'track_listing': 'http://www.example.com/api/track_list/12/',
}
```

### 嵌套序列化关系模型

在序列化模型中指定嵌套序列化关系模型将返回一个该嵌套序列化关系模型对应的数据模型中序列化的数据。
读起来有些拗口，看例子吧。

参数：

- `many` 如果应用于多对多关系，则应将此参数设置为 `True`

序列化模型如下

```python
class TrackSerializer(serializers.ModelSerializer):
    class Meta:
        model = Track
        fields = ('order', 'title', 'duration')

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackSerializer(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```

序列化结果如下：

```javascript
 {
    'album_name': 'The Grey Album',
    'artist': 'Danger Mouse',
    'tracks': [
        {'order': 1, 'title': 'Public Service Announcement', 'duration': 245},
        {'order': 2, 'title': 'What More Can I Say', 'duration': 264},
        {'order': 3, 'title': 'Encore', 'duration': 159},
    ],
}
```
