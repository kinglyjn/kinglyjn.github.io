---
layout: post
title:  "HTML内联框架iframe"
desc: "HTML内联框架iframe"
keywords: "HTML内联框架iframe,iframe,html,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [html,iframe]
icon: fa-html5
---

> 由于现在frame和frameset很少使用，已经过时了，已经被div+CSS代替了，所以，这里只是举例说明一下，当下还在使用的内联框架iframe。所谓的iFrame内联框架，简单理解就是在网页内部嵌套一个网页，并且可以一级一级地嵌套下去。

测试代码: <br>


```html
<!--index.html-->
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>内联框架测试</title>
</head>
<body>
    <div style="margin-left:20%;">
        <!--内联框架可以嵌套-->
        <iframe src="framec.html" frameborder="0" width="60%" height="50%"/>
    </div>

    <!--防止被内嵌js代码-->
    <script type="text/javascript">
        if (window!=top) // 判断当前的window对象是否是top对象
        top.location.href =window.location.href; // 如果不是，将top对象的网址自动导向被嵌入网页的网址
    </script>
</body>
</html>


<!--framec.html-->
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>framec</title>
</head>
<body>
    <div style="background-color: green; width:500px;height:300px;">
        <a href="http://www.baidu.com">在当前iframe打开网页</a><br>
        <a href="http://www.baidu.com" target="_parent">在父级frame中打开新网页</a><br>
        <a href="http://www.baidu.com" target="_top">在顶级frame中打开新网页</a>
    </div>
</body>
</html>
```

效果图：<br>
<img src="http://img.blog.csdn.net/20161228163646102?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br><br>

iframe元素相关属性：<br>
<img src="http://img.blog.csdn.net/20161228164007809?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br><br>


