---
layout: page
title: Django Rest Framework 对序列化 Model 字段的处理方法
date: 2017-12-09 00:45:46
tags:
- djangorestframework
- drf
---

Django Rest Framework 从模型中输出数据，如果在输出之前要针对某一个字段进行修改，可以用 `serializers.SerializerMethodField()` 方法指定一个新的字段，然后在 `get_新字段` 方法中对指定字段进行处理。

具体用法如下：

```
# models.py
class Article(models.Model):
    title = models.CharField(max_length=250)
    pub_date = models.DateField()
    content = models.TextField()
```

要输出一个 `title`、`pub_date`、`about` 序列化数据。其中 `title` 和 `pub_date` 为 models.py 中对应的字段， `about` 为 `content` 字段的二次处理后数据，其实也就截取部分数据而已。

```
# serializers.py
class ArticleSerializer(serializers.HyperlinkedModelSerializer):
    about = serializers.SerializerMethodField()

    class Meta:
        model = Article
        fields = ('title', 'pub_date', 'about')
    
    def get_about(self, obj):
        return obj.content[:100]
```

这样得到序列化的数据就是针对指定字段二次处理后的。