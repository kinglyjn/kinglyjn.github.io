---
layout: post
title:  "seaborn可视化库分析库基础02 - API测试"
desc: "seaborn可视化库分析库基础02 - API测试"
keywords: "seaborn可视化库分析库基础02 - API测试,kinglyjn,张庆力"
date: 2017-11-22
categories: [Py]
tags: [Py]
icon: fa-cubes
---



```python
%matplotlib inline
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats, integrate

sns.set(color_codes=True)
np.random.seed(sum(map(ord, "discributions")))

x = np.random.normal(size=100)
#sns.distplot(x, kde=False)
sns.distplot(x, bins=20, kde=False, fit=stats.gamma)
```

<img src="http://img.blog.csdn.net/20171123193647418?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
%matplotlib inline
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats, integrate

mean,cov = [0,1], [(1,0.5), (0.5,1)]
data = np.random.multivariate_normal(mean, cov, 200)
df = pd.DataFrame(data, columns=["x", "y"]) #将数据转化成pandas的Dataframe格式
sns.jointplot(x="x", y="y", data=df)

x,y = np.random.multivariate_normal(mean, cov, 1000).T
with sns.axes_style("white"):
    sns.jointplot(x=x, y=y, kind="hex", color="k")
```

<img src="http://img.blog.csdn.net/20171123193753027?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:50%"/>
<br>

```python
iris = sns.load_dataset("iris")
sns.pairplot(iris)
```

<img src="http://img.blog.csdn.net/20171123193835316?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
np.random.seed(sum(map(ord, "regression")))
tips = sns.load_dataset("tips")
print (tips.head())

# 使用regplot回归分析
sns.regplot(x="total_bill", y="tip", data=tips)
# 使用rlmplot回归分析
sns.lmplot(x="total_bill", y="tip", data=tips)
```

<img src="http://img.blog.csdn.net/20171123193917305?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
%matplotlib inline
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats, integrate

np.random.seed(sum(map(ord, "regression")))
tips = sns.load_dataset("tips")
sns.regplot(data=tips, x="size", x_jitter=0.05, y="tip")
```

<img src="http://img.blog.csdn.net/20171123194000607?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
%matplotlib inline
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats, integrate

sns.set(style="whitegrid", color_codes=True)
tips = sns.load_dataset("tips")

# 对于分类的数据绘制散点图
# sns.stripplot(data=tips, x="day", y="total_bill", jitter=True)
sns.swarmplot(data=tips, x="day", y="total_bill", hue="sex")
```

<img src="http://img.blog.csdn.net/20171123194032718?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
# 盒图
# IQR即统计学概念的四分位距，第一四分位 与 第三四分位之间的距离
# N=1.5IQR 如果一个值>Q3+N 或 <Q1-N，则为离群点

%matplotlib inline
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats, integrate

tips = sns.load_dataset("tips")
sns.boxplot(data=tips, x="day", y="total_bill", hue="time")
```

<img src="http://img.blog.csdn.net/20171123194109041?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

<br>

```python
# 小提琴图（功能类似于盒图）

tips = sns.load_dataset("tips")
sns.violinplot(data=tips, x="day", y="total_bill", hue="sex", split=True)
```

<img src="http://img.blog.csdn.net/20171123194145781?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
sns.violinplot(data=tips, x="day", y="total_bill", inner=None)
sns.swarmplot(data=tips, x="day", y="total_bill", color="w", alpha=0.5)
```

<img src="http://img.blog.csdn.net/20171123194225524?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
# 柱形图
titanic = sns.load_dataset("titanic")
sns.barplot(data=titanic, x="sex", y="survived", hue="class")
```

<img src="http://img.blog.csdn.net/20171123194304056?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
# 点图 - 描述差异性
titanic = sns.load_dataset("titanic")
sns.pointplot(data=titanic, x="class", y="survived", hue="sex", 
              palette={"male":"g", "female":"m"}, markers=["^", "o"], linestyles=["-", "--"])
```

<img src="http://img.blog.csdn.net/20171123194340867?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
# 多面板分类图(hue是分组绘图参数)

tips = sns.load_dataset("tips")
sns.factorplot(data=tips, x="day", y="total_bill", hue="smoker", kind="bar")
```

<img src="http://img.blog.csdn.net/20171123194413220?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
sns.factorplot(data=tips, x="day", y="total_bill", hue="smoker", col="time", kind="swarm")
```

<img src="http://img.blog.csdn.net/20171123194450388?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/>
<br>

```python
# x,y,hue： xy数据、分组统计字段
# row,col：更多分类变量进行平铺显示
# col_wrap：每行的最高平铺数（整数）
# estimator：在每个分类中进行矢量到标量的映射（矢量）
# ci：置信区间（浮点数或None）
# n_boot：计算置信区间时使用的引导迭代次数（整数）
# utils：采样单元的标识符，用于执行多级引导和重复测量设计（数据变量或者向量数据）
# order,hue_order：对应排序列表（字符串列表）
# row_order,col_order：对应排序列表（字符串列表）
# kind：可选（默认为point）。bar柱形图，count频次，box箱体，violin提琴图，strip散点图，swarm分散点，
# size：每个面的高度[英寸]
# aspect：横纵比（标量）
# orient：方向 v/h
# color：颜色
# matplotlib：颜色
# palette：调色板
# seaborn：颜色色板或字典
# legend：hue的信息面板 (True/False)
# legent_out：是否扩展图形，并将信息框绘制在中心右边（True/False）
# share{x,y}：共享轴线 True/False

sns.factorplot(data=tips, x="time", y="total_bill", hue="smoker", col="day", kind="box", size=4, aspect=0.5)
```

<img src="http://img.blog.csdn.net/20171123194524421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/>
<br>

```python
tips = sns.load_dataset("tips")

pal = {'Lunch': 'seagreen', 'Dinner': 'gray'}
g = sns.FacetGrid(tips, hue="time", hue_kws={"marker":["o","+"]}, palette=pal, size=5)
g.map(plt.scatter, "total_bill", "tip", s=50, alpha=0.7, linewidth=0.5, edgecolor="white")
g.add_legend()
```

<img src="http://img.blog.csdn.net/20171123194555025?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
with sns.axes_style("white"):
    g = sns.FacetGrid(tips, row="sex", col="smoker", margin_titles=True, size=2.5)
g.map(plt.scatter, "total_bill", "tip", color="#334488", edgecolor="white", lw=0.5)
g.set_axis_labels("Total bill (US Dollars)", "Tip")
g.set(xticks=[10,30,50], yticks=[2,6,10])
g.fig.subplots_adjust(wspace=0.02, hspace=0.02)
```

<img src="http://img.blog.csdn.net/20171123194624369?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
iris = sns.load_dataset("iris")
print (iris.head())
g = sns.PairGrid(iris, hue="species")
g.map_diag(plt.hist)
g.map_offdiag(plt.scatter)
g.add_legend()
#g.map(plt.scatter)
```

<img src="http://img.blog.csdn.net/20171123194702940?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/>
<br>

```python
# 取数据的子集(vars)
g = sns.PairGrid(iris, vars=["sepal_length", "sepal_width"], hue="species")
g.map(plt.scatter)
```

<img src="http://img.blog.csdn.net/20171123194736822?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
# 热度图
%matplotlib inline
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

sns.set()

uniform_data = np.random.rand(3,3)
print (uniform_data)
heatmap = sns.heatmap(uniform_data)
```

<img src="http://img.blog.csdn.net/20171123194806913?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
# 指定显示的数据区间
print (uniform_data)
ax = sns.heatmap(uniform_data, vmin=0.2, vmax=0.5)
```

<img src="http://img.blog.csdn.net/20171123194838594?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
normal_data = np.random.randn(3,3)
print (normal_data)
ax = sns.heatmap(normal_data, center=0)
```

<img src="http://img.blog.csdn.net/20171123194909760?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

```python
# seaborn航班数据
flights = sns.load_dataset("flights")
print (flights.head())

flights = flights.pivot("month", "year", "passengers")
print (flights.head())

#linewidths表示元素间的间距（默认没有间距）
#annot表示是否显示数值，d表示数值以数字显示，e表示数值以科学计数法显示
ax = sns.heatmap(flights, linewidths=0.5, annot=True, fmt="d")
```

<img src="http://img.blog.csdn.net/20171123194943996?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/>
<br>

```python
# 指定热图的调色板 cmap
ax = sns.heatmap(flights, linewidths=0.5, annot=True, fmt="d", cmap="YlGnBu") 
```

<img src="http://img.blog.csdn.net/20171123195018035?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br>

