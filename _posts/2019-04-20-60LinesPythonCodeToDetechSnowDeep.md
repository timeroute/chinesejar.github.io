---
layout: page
title: Python 60行代码使用 OpenCV 识别雪深
date: 2019-04-20 19:42:52
tags:
- Python
- OpenCV
---

前两天跟一个朋友吃饭，聊到他在做的图像识别测量雪深，对此深感兴趣，找时间就把 OpenCV 了解一下。

识别标杆上红色刻度的数量。

研究了一下午，话不多说，直接开始演示吧。

```python
import cv2
# 读取图片
img = cv2.imread("./snow.jpeg")
```

首先，将红色部分提取，则需要将原图进行颜色空间转换，转换类型使用 BGR2HSV 方法。

> [HSV](https://zh.wikipedia.org/wiki/HSL%E5%92%8CHSV%E8%89%B2%E5%BD%A9%E7%A9%BA%E9%97%B4) 是一种将RGB色彩模型中的点在圆柱坐标系中的表示法。H 为色相，是色彩的基本属性，S 为饱和度，V 为明度。

从网上查了下，红色区域的 H 值在 [0,10] 和 [170,180]，使用 inRange 方法将红色范围内外的颜色区分开

```
hsv_img = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
mask1 = cv2.inRange(hsv_img, np.array([0, 70, 50]), np.array([10, 255, 255]))
mask2 = cv2.inRange(hsv_img, np.array([170, 70, 50]), np.array([180, 255, 255]))
mask = mask1 | mask2
```

mask 显示效果如下

![mask]({{ site.url }}/assets/60LinesPythonCodeToDetechSnowDeep/mask.jpg)


此时，图像上除了刻度外，还有些地方呈现白色，需要将这些杂质过滤掉，同时也要将垂直部分的白色去掉，需要经过先膨胀再腐蚀再膨胀三个过程。为什么要这样呢？因为这样才能过滤掉杂质以及垂直方向的红线部分，以致达到效果，具体看下面的代码和图。

```python
dilated = cv2.dilate(mask, cv2.getStructuringElement(cv2.MORPH_RECT, (3, 3)), iterations=2)
# 创建一个水平的结构元素，进行腐蚀和膨胀
hline = cv2.getStructuringElement(cv2.MORPH_RECT, (int(dilated.shape[1] / 32), 1), (-1, -1))
# 腐蚀掉多余的白色部分
temp = cv2.erode(dilated, hline)
# 使白色部分膨胀
dst_img = cv2.dilate(temp, hline)
```

效果如下：

![mask]({{ site.url }}/assets/60LinesPythonCodeToDetechSnowDeep/提取红色部分并去掉垂直部分.jpg)

得到提取后的部分，发现还有一个问题，左右刻度有些连结在了一起，此时需要分割。分割的方式是先计算一下宽度，得出中点宽度值，在此原图对应的中点宽度画一条黑线（不过效率有点低啊😊

```python
def get_mid_width(mask):
    """
    获取白色轮廓区域的中间宽度
    """
    min_width = mask.shape[1]
    max_width = 0
    for line in mask:
        """
        处理图片为白色轮廓区域，计算轮廓区域宽度的中间值
        """
        indexes = list(filter(lambda i: line[i] != 0, range(len(line))))
        if len(indexes) != 0:
            if min_width > indexes[0]:
                min_width = indexes[0]
            if max_width < indexes[-1]:
                max_width = indexes[-1]
        else:
            continue
    mid_width = int((min_width + max_width) / 2)
    return mid_width

mid_width = get_mid_width(dst_img)
# 在图片上画一条黑线，用来分割左右红线区域，避免膨胀的时候连在一起
cv2.line(img, (mid_width, 0), (mid_width, img.shape[0]), (0, 0, 0), 20)
```

得到如下图：

![mask]({{ site.url }}/assets/60LinesPythonCodeToDetechSnowDeep/blackline.jpg)

然后重复上面的提取红色部分并过滤的步骤，得到如下图：

![mask]({{ site.url }}/assets/60LinesPythonCodeToDetechSnowDeep/filtered_img.jpg)

此时已经完成90%了，剩下的就是获取每个轮廓，以及把轮廓在原图上描绘出来

```python
contours, _ = cv2.findContours(img, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
# 获取轮廓
# 画上所有轮廓
cv2.drawContours(img_source, contours, -1, (0, 0, 0), 3)
imS = cv2.resize(img_source, (540, 960))
cv2.imshow('result', imS)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

最终效果

![mask]({{ site.url }}/assets/60LinesPythonCodeToDetechSnowDeep/complete.jpg)

是不是很简单呢？！整理完才60行代码，不过这只是简单的实现，一旦涉及到有比较大的杂质或者标杆倾斜以及其他情况，都会影响识别率。
