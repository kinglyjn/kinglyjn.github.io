---
layout: post
title:  "正则表达式中的向前匹配、向后匹配、负向前匹配、负向后匹配"
desc: "正则表达式中的向前匹配、向后匹配、负向前匹配、负向后匹配"
keywords: "正则表达式中的向前匹配、向后匹配、负向前匹配、负向后匹配,正则表达式,regEx"
date: 2016-12-08
categories: [Other]
tags: [Other]
icon: fa-plus-circle
---

比如我们要匹配下面这个语句中的“<”后面不是“br>”的“<”： <br>

```default
<div>line1</div> <br> 
```

这个正则表达式这么写：<br>

```js
/<(?!br>)/
```

<img src="http://img.blog.csdn.net/20170123102827165?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/><br>

<img src="http://img.blog.csdn.net/20170123102921674?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/><br>

如果我们只匹配后面为“br>”的“<”呢，正则表达式这么写：

```js
/<(?=br>)/
```

<img src="http://img.blog.csdn.net/20170123103326582?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/><br>

这两种语法在正则表达式中称之为：

* (?=pattern) 零宽正向先行断言
* (?!pattern) 零宽负向先行断言

断言的意思是判断是否满足，零宽的意思是它只匹配一个位置，如同^匹配开头，$匹配末尾一样，只是一个位置，不返回匹配到的字符，正向表示需要满足pattern，负向表示不能满足pattern，先行表示这个断言语句现在期望返回的匹配字符的后面。<br>


<br>

我们在来假设一个需求，如果我要匹配不在“`<br>`”中的“`>`”，也就是说只匹配“`<div>`”、“`</div>`”中的“`>`”，而不匹配“`<br>`”中的“`>`”，那么要写的正则表达式就是“匹配前面没有'`<br`'的'`>`'”，写法如下：<br>

```js
/(?<!<br)>/
```

对应的如果只匹配“`<br>`”中的“`>`”，而不匹配“`<div>`”或者“`</div>`”中的“`>`”，就这么写：<br>

```js
/(?<=<br)>/
```

这两种语法在正则表达式中称之为：<br>

* (?`<`=pattern) 零宽正向后行断言
* (?`<`!pattern) 零宽负向后行断言

与先行断言的意思一样，只不过后行断言写在需要匹配的字符的前面，表示如果前面的字符满足pattern就返回。<br>

但是很遗憾的是javascript中并不支持这种后行断言。<br>

