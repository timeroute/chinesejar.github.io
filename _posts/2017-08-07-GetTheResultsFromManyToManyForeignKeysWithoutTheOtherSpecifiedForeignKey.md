---
layout: page
title: 获取一些包含多对多外键但是不包含特定另一个外键的结果
date: 2017-08-07 11:45:12
tags:
- Django
- Pyhton
- ORM
---

> 前天碰到了一个情况，思考了好久没能想出问题，昨天下午在 stackoverflow 上问了一下，有一个好心人跟我沟通了一下，了解清楚我的问题以后，到了晚上给出了答案，我操作了一下果然是正确的，让我感动不已。他是一个好人，好人一生平安。

## 问题

开始描述我的问题吧，标题不能描述清楚我的问题，我应该在正文详细描述一下

有 **用户( User )** 、**发布状态( Feed )** 、**评论( Comment )** 三个 Model，分别定义如下
```
class User(models.Model):
    name = models.CharField(max_length=20)


class Feed(models.Model):
    user = models.ForeignKey(User, related_name="feed_user")


class Comment(models.Model):
    user = models.ForeignKey(User, related_name="comment_user")
    feed = models.ForeignKey(Feed, related_name="comment_feed")
```

数据库目前的记录

#### User

| id | name |
|---|---|---|
| 1 | Jack |
| 2 | Bruce |

#### Feed

| id | user |
|---|---|
| 1 | 1 |

#### Comment

| id | user | feed |
|---|---|---|
| 1 | 2 | 1 |
| 2 | 1 | 1 |

我需要获取一个用户的所有 Feed ，每一个 Feed 包含它所有的 Comment ，但是每一个 Comment 对应的 user 不能是 Feed 对应的 user 。

说白了我就是想获取不是我自己的所有 Comment 。

## 解决方法

我能想到的就是使用 exclude 、 Q 这些方法。所以一开始我的获取方式如下：

### 获取 Jack 的所有 Feed

从上面的数据库记录来看， Jack 发了一条 Feed， 这条 Feed 有两条 Comment，一条是 Bruce 评论的，还有一条是 Jack 自己评论的。

```
Feed.objects.filter(user=1)
```

序列化结果如下

```
[
    {
        "feed": {
            "id": 1,
            "user": {
                "id": 1
            },
            "comment": [
                {
                    "id": 1,
                    "user": {
                        "id": 2
                    },
                    "feed": {
                        "id": 1,
                        "user": {
                            "id": 1
                        }
                    }
                },
                {
                    "id": 2,
                    "user": {
                        "id": 1
                    },
                    "feed": {
                        "id": 1,
                        "user": {
                            "id": 1
                        }
                    }
                }
            ]
        }
    }
]
```

### 使用 exclude 方法过滤 Jack 自己评论的结果

```
Feed.objects.filter(user=1).exclude(feed_user__user=1)
```

这种方法获取的序列化结果

```
[
    {
        "feed": {
            "id": 1,
            "user": {
                "id": 1
            },
            "comment": []
        }
    }
]
```

此时的序列化结果竟然把整个 comment 过滤掉了

通过查看 queryset 的 SQL 语句

```
SELECT `feed_feed`.`id`, `feed_feed`.`user_id` FROM `feed_feed` WHERE NOT (`feed_feed`.`id` IN (SELECT U1.`feed_id_id` AS Col1 FROM `feed_comment` U1 WHERE U1.`user_id` = 1)) ORDER BY `feed_feed`.`create_time` DESC
```

发现它是通过判断 Jack 是否出现在所有评论中的某一个， 因此如果出现了就认为整个都不要了，这种方式不可取。


> prefetch_related 返回一个 QuerySet ，为每个关系单独查找，并在 Python 中“加入”。这允许它预取多对多和多对一对象，除了外键和一对一关系。

```
Feed.objects.filter(user=1).prefetch_related(
    Prefetch(
        "feed_comment",
        queryset=Comment.objects.exclude(
            user=1
        ),
        to_attr="all_comment"
    )
)
```

此时的序列化结果如下

```
[
    {
        "feed": {
            "id": 1,
            "user": {
                "id": 1
            },
            "comment": [
                {
                    "id": 1,
                    "user": {
                        "id": 2
                    },
                    "feed": {
                        "id": 1,
                        "user": {
                            "id": 1
                        }
                    }
                },
                {
                    "id": 2,
                    "user": {
                        "id": 1
                    },
                    "feed": {
                        "id": 1,
                        "user": {
                            "id": 1
                        }
                    }
                }
            ]，
            "all_comment": [
                {
                    "id": 1,
                    "user": {
                        "id": 2
                    },
                    "feed": {
                        "id": 1,
                        "user": {
                            "id": 1
                        }
                    }
                }
            ]
        }
    }
]
```

结果在 Jack 所有的 Feed 基础上，添加了一个 **all_comment** 字段，而这个字段正好是过滤了 Jack 的评论，是我要的最终结果。
