---
layout: page
title: Django 处理微信支付的回调
date: 2018-08-17 19:42:23
tags:
- Django
- 微信支付
---

> 最近在做的项目中需要涉及支付功能的开发，微信支付的下单和回调都是采用 XML 格式，Django Rest Framework 默认不支持 XML 渲染方式。需要安装一个第三方库来支持，并且需要做一些修改。

## 安装支持 Rest Framework 的 XML 渲染库

```
pip install djangorestframework-xml
```

## 复写 XMLRenderer 类

由于 `djangorestframework-xml` 不支持解析 XML 文件中含有 `CDATA` 的数据，需要使用 `lxml` 库来替换。

```python
from django.utils import six
from lxml.etree import CDATA, Element, tostring, SubElement
from rest_framework_xml.renderers import XMLRenderer


class WechatXMLRenderer(XMLRenderer):
    """
    微信数据 XML 渲染模块
    """

    root_tag_name = 'xml'

    def _to_xml(self, xml, data):
        if isinstance(data, (list, tuple)):
            for item in data:
                child = SubElement(xml, self.item_tag_name)
                self._to_xml(child, item)

        elif isinstance(data, dict):
            for key, value in six.iteritems(data):
                child = SubElement(xml, key)
                self._to_xml(child, value)

        elif data is None:
            # Don't output any value
            pass

        else:
            xml.text = CDATA(data)

    def render(self, data, accepted_media_type=None, renderer_context=None):
        """
        Renders `data` into serialized XML.
        """
        if data is None:
            return ''

        xml = Element(self.root_tag_name)
        self._to_xml(xml, data)

        return tostring(xml, encoding="unicode")


```

## 在微信支付回调视图里添加渲染

将上一步复写的 `XML` 渲染类放到视图的 `renderer_classes` 里。
然后复写 `post` 方法，对其进行相应的数据验证

```python
class WechatPayNotifyView(generics.CreateAPIView):
    """
    微信回调
    """

    permission_class = ()
    authentication_class = ()
    serializer_class = WechatPayNotifySerializer
    parser_classes = (XMLParser, )
    renderer_classes = (WechatXMLRenderer, )

    def post(self, request, *args, **kwargs):
        # parse_xml 为解析 XML 文件，转换为 JSON
        # check_signature 为验证 sign
        # 以上实现可使用 wechatpy 库
        data = parse_xml(request.body.decode())
        if check_signature(data) and data.get('result_code') == "SUCCESS":
            sale_record.status = True
            sale_record.save()
            serializer = self.get_serializer(data={
                'return_code': request_data.get('result_code')})
            serializer.is_valid(raise_exception=True)
            return Response(serializer.data)
```
