---
layout: post
title:  "css3基本选择器"
desc: "css3基本选择器"
keywords: "css3基本选择器,css3,css,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [css]
icon: fa-html5
---

### 属性选择器和模糊匹配

```
根据部分属性值选择：
如果需要根据属性值中的词列表的某个词进行选择，则需要使用波浪号（~）。
假设您想选择 class 属性中包含 important 的元素，可以用下面这个选择器做到这一点：
p[class~="important"] {color: red;} 
如果忽略了波浪号，则说明需要完成完全值匹配。


部分值属性选择器与点号类名记法的区别：
该选择器等价于我们在类选择器中讨论过的点号类名记法。
也就是说，p.important 和 p["important"] 应用到 HTML 文档时是等价的。
那么，为什么还要有 "~=" 属性选择器呢？因为它能用于任何属性，而不只是 class。
例如，可以有一个包含大量图像的文档，其中只有一部分是图片。对此，可以使用一个基于 title 文档的部分属性选择器，只选择这些图片：
img[title~="Figure"] {border: 1px solid gray;}
这个规则会选择 title 文本包含 "Figure" 的所有图像。没有 title 属性或者 title 属性中不包含 "Figure" 的图像都不会匹配。


子串匹配属性选择器
下面为您介绍一个更高级的选择器模块，它是 CSS2 完成之后发布的，其中包含了更多的部分值属性选择器。
按照规范的说法，应该称之为“子串匹配属性选择器”。很多现代浏览器都支持这些选择器，包括 IE7。
下表是对这些选择器的简单总结：
类型	            描述
------------------------------------------------------
[abc^="def"]	选择 abc 属性值以 "def" 开头的所有元素
[abc$="def"]	选择 abc 属性值以 "def" 结尾的所有元素
[abc*="def"]	选择 abc 属性值中包含子串 "def" 的所有元素
------------------------------------------------------
可以想到，这些选择有很多用途。
举例来说，如果希望对指向 W3School 的所有链接应用样式，不必为所有这些链接指定 class，再根据这个类编写样式，而只需编写以下规则：
a[href*="w3school.com.cn"] {color: red;}
```
<br>

### 结构性伪类选择器

```css
p:first-line{
    color:red; /*给p标签中第一行加样式*/ 
}
p:first-letter {
    color:red; /*给p标签中第一个字或字母加样式 */
}
li:before {
    content:"--"; /*给每一个li标签之前插入内容    //或：text-indent:10px; 首行文本缩进10像素*/
    font-size:10px;
    color:gray;
}  
li:after {
    content:"解释语"; /*给每一个li标签之后插入内容*/
    font-size:10px;
    color:gray; 
}  
```
<br><br>


### root、not、empty、target选择器

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>CSS3_选择器测试</title>
    
    <style type="text/css">
    :root {
	    background-color: lightgray;
	}
	
	/*如果使用了root，则body选择器只对内容区域起作用*/
	body { 
	    background-color: darkseagreen;
	}
	div *:not(h1) {
	    color: red;
	}
	
	table tr td:empty {
	    background-color: gray;
	}
	
	/*URL 带有后面跟有锚名称 #，指向文档内某个具体的元素。这个被链接的元素就是目标元素(target element)*/
	div:target {
	    background-color: darkorange;
	}
    </style>
</head>
<body>
    <div>
        <h2>h2标签</h2>
        <h1>这是标题</h1>
        <p>这是一个p标签</p>
    </div>
    <hr/>
    <table border="1">
        <tr>
            <td>内容1</td>
            <td>内容2</td>
        </tr>
        <tr>
            <td></td>
            <td>内容2</td>
        </tr>
        <tr>
            <td>内容1</td>
            <td></td>
        </tr>
    </table>
    <hr/>
    
    <!-- 锚 -->
    <a href="#a1">菜单一</a>
    <a href="#a2">菜单二</a>
    <a href="#a3">菜单三</a>
    <a href="#a4">菜单四</a>
    <div id="a1">
        <h1>菜单一</h1>
        <p>内容一</p>
    </div>
    <div id="a2">
        <h1>菜单二</h1>
        <p>内容二</p>
    </div>
    <div id="a3">
        <h1>菜单三</h1>
        <p>内容三</p>
    </div>
    <div id="a4">
        <h1>菜单四</h1>
        <p>内容四</p>
    </div>
</body>
</html> 
```
效果图：<br>

<img src="http://img.blog.csdn.net/20161229131555252?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br><br>

### 序号选择器

```css
li:first-child { /*第一个*/
    background-color: yellow;
}
li:last-child { /*最后一个*/
    background-color: lightblue;
}
li:nth-child(3) { /*从前往后数*/
    background-color: darkorange;
}
li:nth-last-child(3) { /*从后向前数*/
    background-color: darkgray;
}
```

<img src="http://img.blog.csdn.net/20161229132428559?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>


```css
li:nth-child(odd) { /*从前往后数奇数个*/
    background-color: yellow;
}
li:nth-child(even) { /*从前往后数偶数个*/
    background-color:darkgray;
}
li:nth-last-child(odd) { /*从后往前数奇数个*/
    background-color: yellow;
}
li:nth-last-child(even) { /*从后往前数偶数个*/
    background-color:darkgray;
}
```

<img src="http://img.blog.csdn.net/20161229132633825?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

```css
/*h2:nth-child(odd){ *//*计数的时候连同其同级元素计算在内*//*
    background-color: yellowgreen;
}*/

h2:nth-of-type(odd) { /*只算h2标签*/
    background-color: yellowgreen;
}
```

<img src="http://img.blog.csdn.net/20161229132736825?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

```css
li:nth-child(4n+1) { /*变量必须是n 且形式为an+b*/;
    background-color: red;
}
li:nth-child(4n+2) {
    background-color: orange;
}
li:nth-child(4n+3) {
    background-color: yellow;
}
li:nth-child(4n+4) {
    background-color: green;
    margin-bottom: 10px;
}
```

<img src="http://img.blog.csdn.net/20161229132847346?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>
<br><br>

### 伪类选择器

```css
:hover /*鼠标掠过*/
:active /*鼠标按下后*/
:focus /*获得焦点*/
:disabled /*属性disabled="disabled"时*/
:enabled /*属性disabled=“”时*/
:read-only /*属性readonly=“readonly”时*/
:checked {outline:2px solid green;} /*单选多选框选中后*/
::selection /*使被选中的文本成为红色 ::selection{color:red;}  只能向 ::selection 选择器应用少量 CSS 属性：color、background、cursor、outline*/
:invalid /*input:invalid{border:2px solid red;}  如果 input 元素中的值是非法的，设置样式为红色 */
:valid /*input:valid{border:2px solid green;}  如果 input 元素中的值是合法的，设置样式为绿色 */
:required /* input:required{background-color:yellow;}  如果 input 元素设置了 "required" 属性，设置其为黄色:*/
:optional /*选择器在表单元素是可选项时设置指定样式  optional 选择器只适用于表单元素: input, select 和 textarea*/

:in-range/out-of-range
/*
用来指定当元素的有效值被限定在一段范围之内的样式，例如：
<input type="text" min="0" max="100">
input[type='number']:in-range {
    background-color: green;
}
input[type='number']:out-of-range {
    background-color: red;
}
*/
```
<br>

### 兄弟选择器

```css
div~p{ /*处在同一个父元素中，与div同级的下文所有p元素*/
    background-color: yellow;
} 
```
<br>

### firefox不支持selected属性的解决方案

```default
使用javascript动态改变select的selected属性时，发现不起作用。 某个<option selected = 'selected' >hello</option>明明有selected = 'selected' 但是选中的却不是它只有预先写好的<option selected = 'selected' >hello</option>才会显示被选中。这个问题在firefox5下才会出现，在FF4下不会，所以我猜想是不是FF5的一个BUG。

可能的原因是FF5处于性能考虑，将一开始加载的dom元素的一些属性缓存起来了，而当使用F5刷新的时候，FF依然使用缓存中的属性，而不使用新的属性。只有使用CTRL+F5强制刷新时，才能渲染新的属性。

最终，在stackoverflow中找到了解决方法：http://stackoverflow.com/questions/6849057/firefox-5-not-using-select-selected-value-on-page-refresh-retaining-old-value，里面说只要在select标签中加autocomplete = ‘off “就行了，<select id="sportid" name="sportid" autocomplete="off"> ，试了一下，确实有用。
```

> [注]：
> input 的属性autocomplete 默认为on，其含义代表是否让浏览器自动记录之前输入的值。很多时候，需要对客户的资料进行保密，防止浏览器软件或者恶意插件获取到，可以在input中加入autocomplete="off" 来关闭记录，系统需要保密的情况下可以使用此参数。








