---
layout: post
title:  "css3基本样式汇总"
desc: "css3基本样式汇总"
keywords: "css3基本样式汇总,css3,css,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [js]
icon: fa-html5
---


背景

```css
background-attachment /*背景图像是否固定或随着页面的奇遇部分滚动 fixed*/
background-color
background-image      /*url("xxx.jpg")*/
background-repeat     /*设置背景图片是否及如何重复 repeat no-repeat repeat-x repat-y*/
background-position   /*center center   50% 50%   100px 100px*/
```

文本

```css 
color            /*颜色*/
derection        /*文本方向*/
line-height      /*行高*/
letter-spacing   /*字符间距*/
text-align       /*对齐元素中的文本  center left right	*/
text-decoration  /*向文本添加修饰*/
text-indent      /*缩进元素中文本的首行 2em  -2em  16px  -16px*/
text-transform   /*元素中的字符  单词首字母大写capitalize 全小写lowercase 全大写uppercase*/
white-space      /*元素中空白的处理方式*/
word-spacing     /*字间距*/
text-shadow      /*文本阴影  2px 2px 5px green  阴影距左位置 阴影距上位置 阴影清晰度的设置[模糊半径blur] 背景颜色*/
text-wrap        /*需配合长宽使用  正常换行normal*/ 
```


字体

```css
font-family
font-size     /*20px*/
font-style    /*字体风格*/
font-variant  /*以小型大写字体或正常字体显示文本*/
font-weight   /*设置字体的粗细 */
```


引用自定义字体文件

```css
@font-face {
    font-family: myfont;
    src:url("字体文件的路径");
}
p {
    font-family: myfont;
}
```


a标记的四种状态

```css
a:link {text-decoration:none;...}
a:hover {}
a:active {}
a:visited {}
```


列表

```css
ul li{
	list-style:circle;   /*简写列表项设置 */
	/*list-style-type: circle;  列表类型  空心圆效果 */
	/*list-style-image:url("image1.jpg");  列表项图片位置 */
	/*list-style-position:inherit; /*inherit inside outside */
}
```


表格

```css
border:1px solid blue;    /*边框*/
border-left:0;
border-collapse:collapse; /*单线折叠边框*/
width:400px;
background-color:brown;
```


轮廓

```css
outline        /*设置轮廓属性*/
outline-color  /*轮廓颜色*/
outline-style  /*轮廓样式*/
outline-width  /*轮廓宽度*/
```

