---
title: 使用python实现图片转字符画
date: 2016-11-07 20:00:00
tags:
  - python
  - ascii
  - image
categories:
  - 技术
author: Kxrr
---


可能不少人都见过这张图……

![](http://img.my.csdn.net/uploads/201211/03/1351953208_3499.gif)

这是一张多帧合成的gif图片, 每帧图片由ascii字符组成。

今天要讲的是如何将一张静态图片转为字符, 关于多帧合成gif的方法这里不涉及。

<!--more-->

## 灰度

首先介绍一下灰度的概念。


> 灰度（Gray scale）数字图像是每个像素只有一个采样颜色的图像。这类图像通常显示为从最暗黑色到最亮的白色的灰度，尽管理论上这个采样可以任何颜色的不同深浅，甚至可以是不同亮度上的不同颜色。灰度图像与黑白图像不同，在计算机图像领域中黑白图像只有黑白两种颜色，灰度图像在黑色与白色之间还有许多级的颜色深度。

![rgb-gray](http://img.pinbot.me:8080/uploads/2016/11/7/blob_1478520980265.png "blob_1478520980265.png")

如下图所示, 左边的第一幅是原始的彩色照片, 右边的为灰度图片。

灰度图片可以用黑白的**颜色深度**来表示一张图片, 这样我们可以在程序中用较大块的字符(比如`M`或`N`)来表示深色的像素, 用小块的字符(比如`.`或`:`)表示浅色。

我们设置一个列表:

```python
CHARS = list("""MNHQ$OC?7>!:-;. """)
CHARS_N = len(CHARS)
```

从左住右, 表示的颜色深度依次递减。

## RGB->灰度->字符

那么如何将彩色图片转为灰度呢？

常见的公式是:

$$
\begin{aligned}
Gray = 0.30 * R + 0.59 * G + 0.11 * B
\end{aligned}
$$



不同的人眼对RGB颜色的感知并不相同，所以转换时候给予不同的权重。
这个原理也普遍应用于计算机图像处理系统。

这里我们使用`rgb2char`实现这个转换:

```python
def rgb2char(r, g, b, alpha=256):
    if alpha == 0:
        return ' '
    gray = float(0.30 * r + 0.59 * g + 0.11 * b)
    index = int(round(gray / 256 * (CHARS_N - 1)))
    return CHARS[index]
```

函数有一个`alpha`参数指的是图像的透明度, 为0是则表示该像素为透明像素, 我们将其转换为一个空格。


## 转换

实现每个像素的转换函数后, 需要对整个图片的每个像素进行处理, 这里我们使用python官方的`PIL`库。

```python
from PIL import Image


def get_image(path):
    im = Image.open(path)
    im = im.resize((HEIGHT, WIDTH), Image.NEAREST)
    return im
    

def image2text(im):
    text = []
    for i in range(HEIGHT):
        for j in range(WIDTH):
            text.append(rgb2char(*im.getpixel((j, i))))
        text.append('\n')
    return ''.join(text)


def convert(path):
    image = get_image(path)
    print(image2text(image))

```

--- 

至些, 基本的转换就完成了。完整的代码可以在[这里](https://gist.github.com/Kxrr/797688110cbdabb7f40a101cbcac7ef9)看到。

![pinbot-logo](http://img.pinbot.me:8080/uploads/2016/11/7/blob_1478520477235.png "blob_1478520477235.png")

另外如果要处理复杂图片的转换需要做许多优化, 大家可以自行研究。






