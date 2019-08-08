---
layout: page
title: Python MongoDB 一些聚合查询方法
date: 2019-04-14 17:28:33
tags:
- Python
- MongoDB
---

> MongoDB 的聚合查询语法一直让我难以很好的入门，如果不是因为项目需要，我很少会用到它，但是用多了之后，会越来越喜欢它，尤其是接触了一些聚合查询方法后，我发现 MongoDB 真的在业务中提高了不少效率。总之，MongoDB 真香～～～

下面是我的一些平时使用聚合查询的记录

data 集合数据格式

```json
{
    "_id" : ObjectId("5caef7f2c0cd2730919a038f"),
    "sn" : "1904010010000001",
    "dev_id" : 200,
    "dt" : ISODate("2036-02-07T14:29:00.000Z"),
    "data" : {
        "BT" : 20.0,
        "CSQ" : 23,
        "GPSLati" : 39.8679244,
        "GPSLongti" : 116.6568387,
        "Humidity" : 0.0,
        "Temprature" : 0.0,
        "Voltage" : 0.0
    }
}
```

### 查询所有 sn 下的最新一条数据

```python
sn = ['1904010010000001', '1904010010000002', '1904010010000003']
pipeline = [
    {'$match': {'sn': {'$in': sn}}},
    {'$group': {'_id': "$sn", "data": {'$last': "$data"}, "dt": {'$last': "$dt"}}},
    {'$sort': {"dt": 1}}]

db.data.aggregate(pipeline)
```

返回结果(避免数据过长，仅显示一个数据)

```python
[
    {
        '_id': '1812010009000100',
        'data': {
            'Ap': 1009.7, 'BT': 20.0, 'CSQ': 24, 
            'GPSLati': 39.8681678, 'GPSLongti': 116.6591262, 
            'Humidity': 31.400000000000002, 'Temprature': 21.5, 
            'Voltage': 0.98, 'WindDir': 0, 'WindSpeed': 0.0
        }, 
        'dt': datetime.datetime(2019, 4, 14, 17, 44)
    }
]
```

### 查询某个 sn 10 小时内每隔 10 分钟统计的平均值

```python
sn = '1904010010000001'
pipeline = [
    {'$project': {'date': {'$substr': ["$dt", 0, 15]}, 'data': '$data'}},
    {'$group': {
        '_id': "$date",
        'temprature': {'$avg': '$data.Temprature'},
        'humidity': {'$avg': '$data.Humidity'},
        'wind_speed': {'$avg': '$data.WindSpeed'},
        'wind_dir': {'$avg': '$data.WindDir'}
    }},
    {'$limit': 60},
    {'$sort': {'_id': -1}}
]

db.data.aggregate(pipeline)
```

返回结果(避免数据过长，仅显示一个数据)

```python
[
    {
        '_id': '2019-04-14T01:3', 
        'temprature': 10.861538461538462, 
        'humidity': 18.70769230769231, 
        'wind_speed': 0.49230769230769234, 
        'wind_dir': 167.6153846153846
    }
]
```
