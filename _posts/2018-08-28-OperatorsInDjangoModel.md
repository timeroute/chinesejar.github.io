---
layout: page
title: 总结 Django Model 的查询方式
date: 2018-08-28 14:43:33
tags:
- Django
---

> Django 自带的 ORM 强大而复杂。

- 如果有多个查询条件组合，则需要用到 Q 方法

```python
# and
CustomModel.objects.filter(Q(id=1) & Q(id=2))

# or
CustomModel.objects.filter(Q(id=1) | Q(id=2))
```

- 如果需要从一个不确定列表长度的数据中读取查询条件，则需要用到 reduce 方法

```python
CustomModel.objects.filter(reduce(operator.or_, map(lambda x: Q(id=x['id']), data)))
```
