---
layout: post
title:  "html基本常见特效设置"
desc: "html基本常见特效设置"
keywords: "html基本常见特效设置,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [html]
icon: fa-html5
---

### 私有名称前缀

为了兼容老版本的写法。例如：比较新版本的浏览器都支持直接写：border-radius。因为制定HTML和CSS标准的组织W3C动作是很慢的。
通常，有w3c组织成员提出一个新属性，比如说圆角border-radius，大家都觉得好，但w3c制定标准，要走很复杂的程序，审查等。而浏览器商市场推广时间紧，如果一个属性已经够成熟了，就会在浏览器中加入支持。为避免日后w3c公布标准时有所变更，加入一个私有前缀，比如-webkit-border-radius，通过这种方式来提前支持新属性。等到日后w3c公布了标准，border-radius的标准写法确立之后，再让新版的浏览器支持border-radius这种写法。<br>

简单的说，浏览器私有前缀，是浏览器对于新CSS属性的一个提前支持。-webkit- 表示 webkit内核，-moz- 代表 Firefox Gecko内核，moz代表的是Firefox的开发商Mozilla。

* -moz代表firefox浏览器私有属性
* -ms代表ie浏览器私有属性
* -webkit代表safari、chrome私有属性

<br>


### 动画效果

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>CSS3_页面特效_动画</title>
    <style type="text/css">
    /*
	Chrome和Safari 需要前缀 -webkit-
	Firefx需要前缀：-moz-
	Opera：-o-
	IE：-ms-
	*/
    div {
	    width:100px;
	    height:100px;
	    background-color: red;
	    position: relative; /*可做不占位移动操作*/
	    animation:anim 5s;  /*动画 5s*/
	    -webkit-animation: anim 5s infinite alternate; /*chrome safari支持 无限交替循环执行*/
	}
	@keyframes anim,@-webkit-keyframes anim{
	    0% {
	        background-color: red;
	        left: 0;
	        top: 0;
	    }
	    25% {
	        background-color: blue;
	        left: 200px;
	        top: 0;
	    }
	    50% {
	        background-color: green;
	        left: 200px;
	        top: 200px;
	    }
	    75% {
	        background-color: orangered;
	        left: 0;
	        top: 200px;
	    }
	    100% {
	        background-color: red;
	        left: 0;
	        top: 0;
	    }
	}
	@-webkit-keyframes anim {
	    0% {
	        background-color: red;
	        left: 0;
	        top: 0;
	    }
	    25% {
	        background-color: blue;
	        left: 200px;
	        top: 0;
	    }
	    50% {
	        background-color: green;
	        left: 200px;
	        top: 200px;
	    }
	    75% {
	        background-color: orangered;
	        left: 0;
	        top: 200px;
	    }
	    100% {
	        background-color: red;
	        left: 0;
	        top: 0;
	    }
	}
    </style>
</head>
<body>
    <div>效果展示</div>
</body>
</html>
```

效果图：<br>

<img src="http://img.blog.csdn.net/20161229110133916?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br><br>


### 过度效果

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>CSS3_页面特效_过渡</title>
    <style type="text/css">
    /*过渡效果：
	  transition  设置四个过渡属性
	  transition-property  过度的名称
	  transition-duration  过渡效果所花费的时间
	  transition-timing-function  过渡效果的时间曲线
	  transition-delay  过渡效果的开始时间
	*/
	div {
	    width:100px;
	    height:100px;
	    background-color: green;
	    -webkit-transition: width 2s, height 2s, transform 2s;
	    -moz-transition: width 2s, height 2s, transform 2s;
	    -ms-transition: width 2s, height 2s, transform 2s;
	    -o-transition:width 2s, height 2s, transform 2s;
	    transition: width 2s, height 2s, transform 2s;
	    -webkit-transition-delay: 1s; /*延时1s执行*/
	    -moz-transition-delay: 1s;
	    -ms-transition-delay: 1s;
	    -o-transition-delay: 1s;
	    transition-delay: 1s;
	}
	div:hover {
	    width:200px;
	    height: 200px;
	    transform: rotate(720deg);
	    -webkit-transition: rotate(360deg);
	} 
    </style>
</head>
<body>
    <div>效果展示</div>
</body>
</html>
```

效果图：<br>

<img src="http://img.blog.csdn.net/20161229111742737?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/><br><br>


### 移动、旋转、缩放、倾斜、矩阵、投影

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>CSS3_页面特效</title>
    <style type="text/css">
    /*转换方法
	  translate 移动 
	  rotate 旋转
      scale 缩放
      skew 倾斜
      matrix 矩阵
      rotateX(Y) 旋转投影
    */
	
	div {
	    width: 100px;
	    height: 100px;
	    background-color: green;
	}
	
	/*移动*/
	.div2 {
	    transform: translate(200px, 100px);
	    -webkit-transform: translate(200px, 100px); /*chrome和safari支持*/
	    -ms-transform: translate(200px, 100px); /*ie支持*/
	    -o-transform: translate(200px, 100px); /*opera支持*/
	    -moz-transform: translate(200px, 100px); /*firefox支持*/
	}
	
	/*旋转*/
	.div3 {
	    transform:rotate(30deg);
	    -webkit-transform: rotate(30deg);
	    -ms-transform: rotate(30deg);
	    -o-transform: rotate(30deg);
	    -moz-transform: rotate(30deg);
	}
	
	/*缩放*/
	.div4 {
	    transform: scale(1,2); /*宽度不变，高度为原来的2倍*/
	    -webkit-transform: scale(1, 2);
	    -ms-transform: scale(1, 2);
	    -o-transform: scale(1, 2);
	    -moz-transform: scale(1, 2);
	}
	
	/*倾斜*/
	.div5 {
	    transform: skew(20deg,30deg); /*以x轴为基准倾斜20度（高度不变，图形本身被拉伸），以y轴为基准倾斜30度*/
	    -webkit-transform: skew(20deg, 30deg);
	    -ms-transform: skew(20deg, 30deg);
	    -o-transform: skew(20deg, 30deg);
	    -moz-transform: skew(20deg, 30deg);
	}
	
	/*旋转投影 rotateX  rotateY*/
	.div6 {
	    transform: rotateX(45deg); /*绕x轴旋转45度后在垂直于纸面方向的投影*/
	    -webkit-transform: rotateX(45deg);
	    -ms-transform: rotateX(45deg);
	    -o-transform: rotateX(45deg);
	    -moz-transform: rotateX(45deg);
	}
    </style>
</head>
<body>
    <div>这是初始效果</div><br/>
    <div class="div2">移动后的效果</div>
    <div class="div3">旋转后的效果</div>
    <div class="div4">缩放后的效果</div>
    <div class="div5">倾斜后的效果</div>
    <div class="div6">3D转换后的效果</div>
</body>
</html>
```

效果图：<br>

<img src="http://img.blog.csdn.net/20161229112229082?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:40%"/><br><br>







