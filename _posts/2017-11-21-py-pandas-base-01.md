---
layout: post
title:  "pandas 基础01"
desc: "pandas 基础01"
keywords: "pandas 基础01,kinglyjn,张庆力"
date: 2017-11-21
categories: [Py]
tags: [Py]
icon: fa-cubes
---



### pandas简介

[pandas官网链接](http://pandas.pydata.org/) <br>

`Pandas`（Python Data Analysis Library）是基于`NumPy` 的一种工具，该工具是为了解决数据分析任务而创建的。Pandas 纳入了大量库和一些标准的数据模型，提供了高效地操作大型数据集所需的工具。pandas提供了大量能使我们快速便捷地处理数据的函数和方法。你很快就会发现，它是使Python成为强大而高效的数据分析环境的重要因素之一。

pandas主要是用来作数据分析和处理的，如果想从事机器学习或数据挖掘相关的工作和研究，跟数据能打上交道的行业都离不开这个pandas库。pandas库给我们提供了非常简洁的函数，从数据的读取、预处理、到最后的分析，用一些很简洁的函数就能表示很复杂的操作。



### pandas api测试

读取数据

```python
'''
aaa.txt
--------------------
Year,t1,t2,t3,t4
1986,t11,t21,t31,23
1987,t12,t22,t32,121
1988,t13,t23,t33,123
1989,t14,t24,t34,125
--------------------
'''

import pandas
import re
#只要文件中的数据是以 "," 为分隔符的，都可以使用 pandas.read_csv 函数进行读取
info = pandas.read_csv('aaa.txt') 
print ("Pandas数据流矩阵（核心类）:\n", type(info))
print ("数表的类型:\n", info.dtypes)
print ("数表的列:\n", info.columns)
print ("数表的型状:\n", info.shape)
print ("数表的前3个样本:\n", info.head(3))
print ("数表的尾3个样本:\n", info.tail(3))
print ("取数表的第0个样本:\n", info.loc[0])
print ("取数表的第[2,3]个样本:\n", info.loc[2:3])
print ("只打印数表的某些行和某些列:\n", info.loc[2:3][["Year","t1"]] )
print ("只打印数表的某些列和某些行:\n", info[["Year","t1"]].loc[2:3] )

columnlist = info.columns.tolist()
print (type(columnlist), columnlist)
target_clolumns = []
target_pattern = re.compile(r't\d')

for c in target_columns:
    if target_pattern.match(c):
        target_clolumns.append(c)
print ("目标列为:\n", target_clolumns)
print ("目标列数据对应的数据为:\n", info[target_clolumns].head(3))


'''
Pandas数据流矩阵（核心类）:
 <class 'pandas.core.frame.DataFrame'>
数表的类型:
 Year     int64
t1      object
t2      object
t3      object
t4       int64
dtype: object
数表的列:
 Index(['Year', 't1', 't2', 't3', 't4'], dtype='object')
数表的型状:
 (4, 5)
数表的前3个样本:
    Year   t1   t2   t3   t4
0  1986  t11  t21  t31   23
1  1987  t12  t22  t32  121
2  1988  t13  t23  t33  123
数表的尾3个样本:
    Year   t1   t2   t3   t4
1  1987  t12  t22  t32  121
2  1988  t13  t23  t33  123
3  1989  t14  t24  t34  125
取数表的第0个样本:
 Year    1986
t1       t11
t2       t21
t3       t31
t4        23
Name: 0, dtype: object
取数表的第[2,3]个样本:
    Year   t1   t2   t3   t4
2  1988  t13  t23  t33  123
3  1989  t14  t24  t34  125
只打印数表的某些行和某些列:
    Year   t1
2  1988  t13
3  1989  t14
只打印数表的某些列和某些行:
    Year   t1
2  1988  t13
3  1989  t14
<class 'list'> ['Year', 't1', 't2', 't3', 't4']
目标列为:
 ['t1', 't2', 't3', 't4']
目标列数据对应的数据为:
     t1   t2   t3   t4
0  t11  t21  t31   23
1  t12  t22  t32  121
2  t13  t23  t33  123
'''
```

<br>



数据的预处理

```python
#数据的预处理
print ("某一列都除以这一列的最大数:\n", info['t4']/info['t4'].max())
info.sort_values("t4", inplace=True, ascending=False) #注意排序会改变原来的dataframe
print ("在原来dataframe基础上按某一行进行排序, inplace=True表示在原来的dataframe位置替换而不copy生成一个新dataframe:\n", info)
print ("可以看到排序之后，我们的索引值变得无序了，这对于我们取数据是不利的，我们希望能对索引进行重新设定，如下：")
new_info = info.reset_index(drop=True)
print ("重新设定索引后的dataframe（之前的datarame 即info并没有被改变）: \n", new_info)


'''
某一列都除以这一列的最大数:
 0    0.184
1    0.968
2    0.984
3    1.000
Name: t4, dtype: float64
在原来dataframe基础上按某一行进行排序, inplace=True表示在原来的dataframe位置替换而不copy生成一个新dataframe:
    Year   t1   t2   t3   t4
3  1989  t14  t24  t34  125
2  1988  t13  t23  t33  123
1  1987  t12  t22  t32  121
0  1986  t11  t21  t31   23
可以看到排序之后，我们的索引值变得无序了，这对于我们取数据是不利的，我们希望能对索引进行重新设定，如下：
重新设定索引后的dataframe（之前的datarame 即info并没有被改变）: 
    Year   t1   t2   t3   t4
0  1989  t14  t24  t34  125
1  1988  t13  t23  t33  123
2  1987  t12  t22  t32  121
3  1986  t11  t21  t31   23
'''
```
<br>



泰坦尼克号乘客样本分析案例：

```python
'''
titanic_train 数据示例:
----------------------
PassengerId,Survived,Pclass,Name,Sex,Age,SibSp,Parch,Ticket,Fare,Cabin,Embarked
1,0,3,"Braund, Mr. Owen Harris",male,22,1,0,A/5 21171,7.25,,S
2,1,1,"Cumings, Mrs. John Bradley (Florence Briggs Thayer)",female,38,1,0,PC 17599,71.2833,C85,C
3,1,3,"Heikkinen, Miss. Laina",female,26,0,0,STON/O2. 3101282,7.925,,S
4,1,1,"Futrelle, Mrs. Jacques Heath (Lily May Peel)",female,35,1,0,113803,53.1,C123,S
5,0,3,"Allen, Mr. William Henry",male,35,0,0,373450,8.05,,S
6,0,3,"Moran, Mr. James",male,,0,0,330877,8.4583,,Q
7,0,1,"McCarthy, Mr. Timothy J",male,54,0,0,17463,51.8625,E46,S
----------------------
'''

import pandas as pd
import numpy as np

titanic_survival = pandas.read_csv('titanic_train.csv')

#定位数表中的元素
print ("第83个样本的Age: ", titanic_survival.loc[83, "Age"])


#缺失值(NA)的处理
age = titanic_survival["Age"]
age_is_null = pd.isnull(age)
age_null_true = age[age_is_null]
age_null_true_count = len(age_null_true)
print ("Age缺失值的个数: ", age_null_true_count)


#使用普通方法计算平均值
good_ages = titanic_survival["Age"][age_is_null==False]
mean_age = sum(good_ages) / len(good_ages)
print ("年龄的平均值（普通方法计算）: ", mean_age)

#使用pandas的函数计算含有缺失值的列的平均数
print ("年龄的平均值（dataframe.mean方法计算）: ", titanic_survival["Age"].mean())


# 计算1，2，3等仓船票的平均价格
passenger_classes = [1,2,3]
fares_by_class = {}
for this_class in passenger_classes:
    pclass_rows = titanic_survival[titanic_survival["Pclass"]==this_class]
    pclass_fares = pclass_rows["Fare"]
    fare_for_class = pclass_fares.mean()
    fares_by_class[this_class] = fare_for_class
print ("各等船舱船票的平均值为: ", fares_by_class)


# 分析船舱等级和是否获救之间的关系（类似于Excel的透视表分析功能）
# pivot_table(index表示基准列，values表示基准列的分析列，aggfunc表示采用的数据透视算法)
passenger_survival = titanic_survival.pivot_table(index="Pclass", values="Survived", aggfunc=np.mean)
print ("各等级船舱的人获救的几率:\n", passenger_survival)
print ("由数据透视结果可知，船舱的等级越高，被获救的计几率也就越高！")

# 再分析船舱等级和年龄之间的关系
passenger_age = titanic_survival.pivot_table(index="Pclass", values="Age", aggfunc=np.mean)
print ("各等级船舱的人的平均年龄:\n", passenger_age)
print ("由数据透视结果可知，船舱的等级越高，该船舱人的平均年龄就越大！")

# 再来分析不同登船地点的 售票总额以及被获救的人数
port_start = titanic_survival.pivot_table(index="Embarked", values=["Fare", "Survived"], aggfunc=np.sum)
print ("不同登船地点的 售票总额以及被获救的人数如下：\n", port_start)

# 将匹配到的 axis=? 或 axis='columns' 对应的列的缺失值丢弃掉
# subset指定在哪些列中没有数据时才进行删除
# how有两个取值 'any'|'all'，表示一行数据在何种方式下被舍弃（至少一个NA还是全部NA）
# thresh=3 表示一行中至少有3个NA时才将其丢弃
drop_na_columns = titanic_survival.dropna(axis=0, how='any', thresh=None)
new_titanic_survival = titanic_survival.dropna(axis=0, subset=["Age", "Sex"])
print (id(new_titanic_survival), id(titanic_survival))


'''
第83个样本的Age:  28.0
Age缺失值的个数:  177
年龄的平均值（普通方法计算）:  29.6991176471
年龄的平均值（dataframe.mean方法计算）:  29.69911764705882
各等船舱船票的平均值为:  {1: 84.15468749999992, 2: 20.66218315217391, 3: 13.675550101832997}
各等级船舱的人获救的几率:
         Survived
Pclass          
1       0.629630
2       0.472826
3       0.242363
由数据透视结果可知，船舱的等级越高，被获救的计几率也就越高！
各等级船舱的人的平均年龄:
               Age
Pclass           
1       38.233441
2       29.877630
3       25.140620
由数据透视结果可知，船舱的等级越高，该船舱人的平均年龄就越大！
不同登船地点的 售票总额以及被获救的人数如下：
                 Fare  Survived
Embarked                      
C         10072.2962        93
Q          1022.2543        30
S         17439.3988       217
4560635328 4464760368
'''
```

<br>



自定义函数:

```python
# 自定义一个函数
def hundredth_row(column):
    return column.loc[99]

def not_null_count(column):
    column_null_true = column[pd.isnull(column)]
    return len(column_null_true)

def which_class(row):
    pclass = row['Pclass']
    if pd.isnull(pclass):
        return 'Unknown'
    elif pclass==1:
        return 'First Class'
    elif pclass==2:
        return 'Second Class'
    elif pclass==3:
        return 'Third Class'
    
def is_minor(row):
    if row["Age"] < 18:
        return True
    else:
        return False
    
def generate_age_label(row):
    age = row["Age"]
    if pd.isnull(age):
        return "Unknown"
    elif age < 18:
        return "Minor"
    else:
        return "Adult"
    
print ("使用自定义函数展示数据（默认axis为0表示列）:\n", titanic_survival.apply(hundredth_row) )
print ("使用自定义函数计算得到的每一列的缺失值个数:\n", titanic_survival.apply(not_null_count))
print ("使用自定义函数变换一个cell的值（这里axis为1表示行）:\n", titanic_survival.apply(which_class, axis=1).head(3))
print ("使用自定义函数判断一个样本是否是小孩:\n", titanic_survival.apply(is_minor, axis=1).head(3))
print ("使用自定义函数判断一个样本的年龄段:\n", titanic_survival.apply(generate_age_label, axis=1).head(3))


# 给titanic_survial这个dataframe加一个列age_labels，并透视分析该列与其他列之间的关系
age_labels = titanic_survival.apply(generate_age_label, axis=1)
titanic_survival['age_labels'] = age_labels
print ("新增age_labels列之后的titanic_survival：\n", titanic_survival.head(3))
age_group_survival = titanic_survival.pivot_table(index='age_labels', values="Survived", aggfunc=np.mean)
print ("不同年龄段人群获救的几率如下：\n", age_group_survival)
print ("有分析结果可知，小孩的获救几率比成年人的获救几率要大很多！")


'''
使用自定义函数展示数据（默认axis为0表示列）:
 PassengerId                  100
Survived                       0
Pclass                         2
Name           Kantor, Mr. Sinai
Sex                         male
Age                           34
SibSp                          1
Parch                          0
Ticket                    244367
Fare                          26
Cabin                        NaN
Embarked                       S
age_labels                 Adult
dtype: object
使用自定义函数计算得到的每一列的缺失值个数:
 PassengerId      0
Survived         0
Pclass           0
Name             0
Sex              0
Age            177
SibSp            0
Parch            0
Ticket           0
Fare             0
Cabin          687
Embarked         2
age_labels       0
dtype: int64
使用自定义函数变换一个cell的值（这里axis为1表示行）:
 0    Third Class
1    First Class
2    Third Class
dtype: object
使用自定义函数判断一个样本是否是小孩:
 0    False
1    False
2    False
dtype: bool
使用自定义函数判断一个样本的年龄段:
 0    Adult
1    Adult
2    Adult
dtype: object
新增age_labels列之后的titanic_survival：
    PassengerId  Survived  Pclass  \
0            1         0       3   
1            2         1       1   
2            3         1       3   

                                                Name     Sex   Age  SibSp  \
0                            Braund, Mr. Owen Harris    male  22.0      1   
1  Cumings, Mrs. John Bradley (Florence Briggs Th...  female  38.0      1   
2                             Heikkinen, Miss. Laina  female  26.0      0   

   Parch            Ticket     Fare Cabin Embarked age_labels  
0      0         A/5 21171   7.2500   NaN        S      Adult  
1      0          PC 17599  71.2833   C85        C      Adult  
2      0  STON/O2. 3101282   7.9250   NaN        S      Adult  
不同年龄段人群获救的几率如下：
             Survived
age_labels          
Adult       0.381032
Minor       0.539823
Unknown     0.293785
有分析结果可知，小孩的获救几率比成年人的获救几率要大很多！
'''
```

<br>





