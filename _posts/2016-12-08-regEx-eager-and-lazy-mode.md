---
layout: post
title:  "正则表达式的贪婪与懒惰模式"
desc: "正则表达式的贪婪与懒惰模式"
keywords: "正则表达式的贪婪与懒惰模式,正则表达式,regEx"
date: 2016-12-08
categories: [Other]
tags: [Other]
icon: fa-plus-circle
---

 当正则表达式中包含能接受重复的限定符时，通常的行为是（在使整个表达式能得到匹配的前提下）匹配尽可能多的字符。以这个表达式为例：a.*b，它将会匹配最长的以a开始，以b结束的字符串。如果用它来搜索aabab的话，它会匹配整个字符串aabab。这被称为贪婪匹配。<br>

 有时，我们更需要懒惰匹配，也就是匹配尽可能少的字符。前面给出的限定符都可以被转化为懒惰匹配模式，只要在它后面加上一个问号?。这样.*?就意味着匹配任意数量的重复，但是在能使整个匹配成功的前提下使用最少的重复。现在看看懒惰版的例子吧：<br>

a.*?b匹配最短的，以a开始，以b结束的字符串。如果把它应用于aabab的话，它会匹配aab（第一到第三个字符）和ab（第四到第五个字符）。<br>

代码/语法 说明：<br>

```default
*? 重复任意次，但尽可能少重复 
+? 重复1次或更多次，但尽可能少重复 
?? 重复0次或1次，但尽可能少重复 
{n,m}? 重复n到m次，但尽可能少重复 
{n,}? 重复n次以上，但尽可能少重复 
```
<br>

<img src="http://img.blog.csdn.net/20170122175412106?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br>

<img src="http://img.blog.csdn.net/20170122175453934?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br>

<img src="http://img.blog.csdn.net/20170122175502778?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br>
