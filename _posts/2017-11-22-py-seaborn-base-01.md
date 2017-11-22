---
layout: post
title:  "seaborn可视化库分析库基础01 - 布局、参数、色板等"
desc: "seaborn可视化库分析库基础01 - 布局、参数、色板等"
keywords: "seaborn可视化库分析库基础01 - 布局、参数、色板等,kinglyjn,张庆力"
date: 2017-11-22
categories: [Py]
tags: [Py]
icon: fa-cubes
---



### Seaborn库简介

[Seaborn库官网](http://seaborn.pydata.org/index.html) 
<br>
正如你所知道的，Seaborn是比Matplotlib更高级的免费库，特别地以数据可视化为目标，但他要比这一切更进一步：他解决了用Matplotlib的2个最大问题，正如Michael Waskom所说的：Matplotlib试着让简单的事情更加简单，困难的事情变得可能，那么Seaborn就是让困难的东西更加简单。用Matplotlib最大的困难是其默认的各种参数，而Seaborn则完全避免了这一问题。
<br>

### Seaborn库api测试

布局设置

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib as mpl
import matplotlib.pylab as plt

def sinplot(flip=1):
    x = np.linspace(0,14,100)
    for i in range(1,7):
        plt.plot(x, np.sin(x+i*0.5)*(7-i)*flip)

# 使用seaborn默认的绘图风格
sns.set()
# seaborn的五种绘图风格 dark、darkgrid、white、whitegrid、ticks
#sns.set_style("whitegrid")
#sinplot()

# wiht中 和 with外 使用不同的绘图风格
sns.set(rc={"figure.figsize":(8,4)})
with sns.axes_style("darkgrid"):
    plt.subplot(211)
    sinplot()
plt.subplot(212)
sinplot(-1) 
```

![这里写图片描述](http://img.blog.csdn.net/20171122193655164?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>

设置画板上下文参数

```python
# sns上下文绘图参数的设置 paper、talk、poster、notebook
sns.set_context("talk", font_scale=1, rc={"lines.linewidth":2.5})
plt.figure(figsize=(8,4))
sinplot()
```

![这里写图片描述](http://img.blog.csdn.net/20171122193836559?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>

调色板

```python
# 调色板
# 颜色很重要
# color_palette()能传入任何Matplotlib 所支持的颜色
# color_palette()不写参数则默认颜色
# set_palette()设置所有图的颜色

#分类色板（有6个默认的颜色循环主题：deep、muted、pastel、bright、dark、colorblind）
sns.palplot(sns.color_palette())
#连续色板
sns.palplot(sns.color_palette("Blues"))
sns.palplot(sns.color_palette("Blues_r"))
sns.palplot(sns.light_palette("green"))
sns.palplot(sns.dark_palette("green"))
#色调线性变换，start和rot指定颜色区间
sns.palplot(sns.color_palette("cubehelix", 8))
sns.palplot(sns.cubehelix_palette(8, start=0.5, rot=-0.75))

#使用连续色板绘图
plt.figure(figsize=(6,6))
x,y = np.random.multivariate_normal([0,0], [[1,-0.5], [-0.5,1]], size=300).T
pal = sns.dark_palette("green", as_cmap=True)
sns.kdeplot(x, y, cmap=pal)


#圆形色板
#当你有六个以上的分类需要区分时，最简单的方法是在一个圆形的颜色空间中画出均匀间隔的颜色（这样的颜色会保持亮度和饱和度不变）。
#这是大多数使用比大年默认颜色循环中设置的颜色更多时的默认方案，最常用的方法时使用hls的颜色空间，这是RGB值的一个简单转换。
sns.palplot(sns.color_palette("hls", 12))
#l表示亮度，s表示饱和度
sns.palplot(sns.hls_palette(12, l=0.3, s=0.8))
#生成成对的颜色
sns.palplot(sns.color_palette("Paired", 12) )


#使用圆形色板绘图
plt.figure(figsize=(6,6))
data = np.random.normal(size=(20,8)) + np.arange(8)/2
sns.boxplot(data=data, palette=sns.color_palette("hls", 12))
```

![这里写图片描述](http://img.blog.csdn.net/20171122194125544?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20171122194137099?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20171122194150471?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>

使用xkcd颜色画图

```python
#使用xkcd颜色画图
#xkcd包含了一套针对RGB颜色的命名，产生了954个可以随时通过xkcd_rgb字典调用的颜色
plt.figure(figsize=(5,5))
plt.plot([0,1], [0,1], sns.xkcd_rgb["pale red"], lw=3)
plt.plot([0,1], [0,2], sns.xkcd_rgb["medium green"], lw=3)
plt.plot([0,1], [0,3], sns.xkcd_rgb["denim blue"], lw=3)
```

![这里写图片描述](http://img.blog.csdn.net/20171122194254944?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>