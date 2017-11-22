---
layout: post
title:  "可视化分析库 matplotlib 基础01 - 基本常见API"
desc: "可视化分析库 matplotlib 基础01 - 基本常见API"
keywords: "可视化分析库 matplotlib 基础01 - 基本常见API,kinglyjn,张庆力"
date: 2017-11-22
categories: [Py]
tags: [Py]
icon: fa-cubes
---



### Matplotlib库简介

[Matplotlib 官网](http://matplotlib.org/devdocs/tutorials/index.html)
Matplotlib 可能是 Python 2D- 绘图领域使用最广泛的套件。它能让使用者很轻松地将数据图形化，并且提供多样化的输出格式。通过数据绘图，我们可以将枯燥的数字转换成容易被人们接受的图表，从而让人留下更加深刻的印象。

<img src="http://img.blog.csdn.net/20171122150637833?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="weidth:70%"/>

<br>




### matplotlib api 测试

绘制简单的折线图

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

unrate = pd.read_csv("UNRATE.csv")
unrate['DATE'] = pd.to_datetime(unrate['DATE'])
print ("美国1948年各个月份失业率如下:\n", unrate.head(12))


#指定x轴和y轴数据
unrate_1948 = unrate.head(12)
plt.plot(unrate_1948['DATE'], unrate_1948['VALUE'])
#x轴数据值倾斜45显示
plt.xticks(rotation=45)
#指定x轴和y轴数据的标签
plt.xlabel('Month')
plt.ylabel('Unemployment Rate')
#指定标题
plt.title('Monthly Unemployment Trends, 1948')
#是否显示辅助方格线
plt.grid(False)

plt.show()
```

![这里写图片描述](http://img.blog.csdn.net/20171122150932438?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>



绘制子图

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

#创建画图区域，指定长和宽
fig = plt.figure(figsize=(10,5))
#画图区域指定在2*2矩阵中，第三个参数表示子图的位置
ax1 = fig.add_subplot(2,2,1)
ax2 = fig.add_subplot(2,2,2)
#以[0,1,2,3,4]为横坐标值，以5个随机数组成的向量为纵坐标值画ax1子图
ax1.plot(np.arange(5), np.random.randint(1,5,5), c='red')
#以[0-2pi]为横坐标，以sin([0-2*pi]的值为纵坐标画ax2子图)
ax2.plot(np.linspace(0,2*np.pi,8), np.sin(np.linspace(0,2*pi,8)), c='blue')

plt.show()
```

![这里写图片描述](http://img.blog.csdn.net/20171122151111857?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>



循环绘制一张图表

```python
#循环将1948-1952年的美国失业率情况绘制在统一图表中
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

unrate = pd.read_csv("UNRATE.csv")
unrate['DATE'] = pd.to_datetime(unrate['DATE'])
unrate['MONTH'] = unrate['DATE'].dt.month

fig = plt.figure(figsize=(8,4))
colors = ['red', 'blue', 'green', 'orange', 'black']
for i in range(5):
    start_index = i*12
    end_index = (i+1)*12
    subset = unrate[start_index:end_index]
    plt.plot(subset['MONTH'], subset['VALUE'], c=colors[i], label=str(1948+i))

#显示图例在最佳位置    
plt.legend(loc='best')
#显示主方格辅助线
plt.grid(True)
#xy轴标签和图表标题
plt.xlabel('Month, Integer')
plt.ylabel('Unemployment Rate, Percent')
plt.title('Monthly Unemployment Trends, 1948-1952')

plt.show()
```

![这里写图片描述](http://img.blog.csdn.net/20171122151332490?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>



绘制一个纵向柱形图

```python
#柱形图测试1
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

reviews = pd.read_csv("fandango_score_comparison.csv")
cols = ['FILM', 'RT_user_norm', 'Metacritic_user_nom', 'IMDB_norm', 'Fandango_Ratingvalue', 'Fandango_Stars']
norm_reviews = reviews[cols]
print ("样本数据如下:\n", norm_reviews[:3])

num_cols = cols[1:]
print ("第0号样本的电影打分情况：", norm_reviews.loc[0, num_cols].values)
bar_heights = norm_reviews.loc[0, num_cols].values
bar_positions = np.arange(5) + 1
# 返回一个 (figure, ax) 的元组
fig,ax = plt.subplots()
# 绘制纵向柱形图
ax.bar(bar_positions, bar_heights, 0.4) #0.4指定柱的宽度

# 设置x轴bar的位置和x轴数据显示的角度
ax.set_xticks(range(1,6))
ax.set_xticklabels(num_cols, rotation=45)

# 设置xy轴的标签和图表的标题
ax.set_xlabel('Rating Source')
ax.set_ylabel('Average Rating')
ax.set_title('Average User Rating For Avengers: Age of Ultron (2015)')

plt.show()
```

![这里写图片描述](http://img.blog.csdn.net/20171122154942901?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>



绘制一个横向柱形图

```python
#柱形图测试2
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

reviews = pd.read_csv("fandango_score_comparison.csv")
cols = ['FILM', 'RT_user_norm', 'Metacritic_user_nom', 'IMDB_norm', 'Fandango_Ratingvalue', 'Fandango_Stars']
norm_reviews = reviews[cols]
num_cols = cols[1:]
bar_heights = norm_reviews.loc[0, num_cols].values
bar_positions = np.arange(5) + 1
# 返回一个 (figure, ax) 的元组
fig,ax = plt.subplots()
ax.barh(bar_positions, bar_heights, 0.4)  ####

# 设置y轴bar的位置和y轴数据显示的角度
ax.set_yticks(range(1,6))
# 绘制横向柱形图
ax.set_yticklabels(num_cols, rotation=0)

# 设置xy轴的标签和图表的标题
ax.set_ylabel('Rating Source')
ax.set_xlabel('Average Rating')
ax.set_title('Average User Rating For Avengers: Age of Ultron (2015)')

plt.show()
```

![这里写图片描述](http://img.blog.csdn.net/20171122155036345?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>



绘制散点图测试1

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

reviews = pd.read_csv("fandango_score_comparison.csv")
cols = ['FILM', 'RT_user_norm', 'Metacritic_user_nom', 'IMDB_norm', 'Fandango_Ratingvalue', 'Fandango_Stars']
norm_reviews = reviews[cols]

fig,ax = plt.subplots()
ax.scatter(norm_reviews['Fandango_Ratingvalue'], norm_reviews['RT_user_norm'])
ax.set_xlabel('Fandango')
ax.set_ylabel('Rotten Tomatoes')

plt.show()
```

![这里写图片描述](http://img.blog.csdn.net/20171122160604640?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>



绘制散点图测试2

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

reviews = pd.read_csv("fandango_score_comparison.csv")
cols = ['FILM', 'RT_user_norm', 'Metacritic_user_nom', 'IMDB_norm', 'Fandango_Ratingvalue', 'Fandango_Stars']
norm_reviews = reviews[cols]

fig = plt.figure(figsize=(10,10) )
ax1 = fig.add_subplot(2,2,1)
ax2 = fig.add_subplot(2,2,2)

ax1.scatter(norm_reviews['Fandango_Ratingvalue'], norm_reviews['RT_user_norm'])
ax1.set_xlabel('Fandango')
ax1.set_ylabel('Rotten Tomatoes')

ax2.scatter(norm_reviews['RT_user_norm'], norm_reviews['Fandango_Ratingvalue'])
ax2.set_xlabel('Rotten Tomatoes')
ax2.set_ylabel('Fandango')

plt.show()
```

![这里写图片描述](http://img.blog.csdn.net/20171122160700807?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>



使用bins区域化x轴

```python
# 使用bins区域化x轴
# 如 将1,2,3,4,5,6,7,8的分布表示为 [1,3),[3,5),[5,7),[7,9)的分布
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

reviews = pd.read_csv("fandango_score_comparison.csv")
cols = ['FILM', 'RT_user_norm', 'Metacritic_user_nom', 'IMDB_norm', 'Fandango_Ratingvalue']
norm_reviews = reviews[cols]

fandango_distribution = norm_reviews['Fandango_Ratingvalue'].value_counts() #计算对应值出现的次数
fandango_distribution = fandango_distribution.sort_index() #按照数值的索引重新排序得到新的Series
imdb_distribution = norm_reviews['IMDB_norm'].value_counts() 
imdb_distribution = imdb_distribution.sort_index() 
print ("测试数据fandango_distribution：\n", fandango_distribution.head(10))
print ("测试数据imdb_distribution：\n", imdb_distribution.head(10))

fig,ax = plt.subplots(figsize=(8,4))
ax.hist(norm_reviews['Fandango_Ratingvalue'])
ax.hist(norm_reviews['Fandango_Ratingvalue'], bins=20)
ax.hist(norm_reviews['Fandango_Ratingvalue'], range=(4,5), bins=20)

plt.show()
```

![这里写图片描述](http://img.blog.csdn.net/20171122164944861?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>



根据已有数据画一个盒图

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

reviews = pd.read_csv("fandango_score_comparison.csv")
num_cols = ['RT_user_norm', 'Metacritic_user_nom', 'IMDB_norm', 'Fandango_Ratingvalue']
print ("测试数据示例如下：\n", reviews[num_cols].head(5))

fig,ax = plt.subplots(figsize=(6,6))
ax.boxplot(reviews[num_cols].values)
ax.set_xticklabels(num_cols, rotation=90)
ax.set_ylim(0,5) #设置y轴的取值范围

plt.show()
```

![这里写图片描述](http://img.blog.csdn.net/20171122170148371?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>

> 更多画图示例，请参考[Matplotlib 官网](http://matplotlib.org/devdocs/tutorials/index.html)