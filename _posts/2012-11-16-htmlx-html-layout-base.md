---
layout: post
title:  "html基本常见布局"
desc: "html基本常见布局"
keywords: "html基本常见布局,布局,layout,html,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [html]
icon: fa-html5
---

### 基本 banner-left-content-footer 布局

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>基本 banner-left-content-footer 布局</title>
	<style>
	body {
		background-color: #e1ddd9;
		font-size: 11px;
		font-family: Verdana, Arial, SunSans-Regular, Sans-Serif;
		color: #564b47;
		padding: 0px;
		margin: 0px;
	}
	
	.container {
		width: 70%;
		margin-bottom: 10px;
		margin-left: auto;
		margin-right: auto;
		background-color: #EBD3E0;
	}
	
	.banner {
		background-color: lightblue;
		text-align: right;
		height: 30px;
		padding: 0px;
		margin: 10px 0 0 0;
	}
	
	.content {
		background-color: #ffffff;
		padding: 0px;
		margin-left: 200px;
		margin-right: 0px;
	}
	
	div.content {
		min-height: 550px; /*auto;*/
		height: expression(this.scrollHeight > 600 ? "auto" : "600px");  /*自适应高度 */
	}
	
	.left {
		float: left;
		width: 200px;
		margin: 0px;
		padding: 0px;
	}
	
	.footer {
		height: 20px;
		clear: both;
		margin: 0px;
		padding: 0px;
		text-align: right;
	}
</style>
</head>

<body>
	<div class="container">
		<div class="banner">banner</div>
		<div class="left">left</div>
		<div class="content">
			content<br/>
		</div>
		<div class="footer">footer</div>
	</div>
</body>
</html>
```

布局效果：<br>

<img src="http://img.blog.csdn.net/20161229092957648?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br><br>


### 最基本的首页

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>CSS3_</title>
    <style type="text/css">
    /*初始化*/
    *{ 
	    margin: 0;
	    padding: 0;
	}
	
	body {
	    background-color: snow;
	}
	.wrapper {
	    width: 80%;
	    height: 1000px;
	    background-color: antiquewhite;
	    margin: 0 auto;
	}
	.heading {
	    width: 100%;
	    height:90px;
	    background-color: snow;
	    margin: 0 auto;
	}
	.heading_title {
	    float: left;
	    font-family:Arial, Helvetica, sans-serif;
	    font-size: 30px;
	    color: burlywood;
	    margin-right: 30px;
	}
	ul {
	    margin_left: 40px;
	    float: left;
	    list-style-type:none;
	    padding-top: 6px;
	    padding-bottom: 6px;
	}
	ul li {
	    padding-left:10px;
	    display: inline;
	
	}
	.heading_nav {
	    padding-top:30px;
	    padding-bottom: 30px;
	    width:100%;
	    height:30px;
	    position: relative;
	}
	a:link, a:visited {
	    font-weight: bold;
	    color: darkgray;
	    text-align: center;
	    padding: 6px;
	    text-decoration: none;
	}
	a:hover, a:active {
	    color: dimgray;
	}
	.heading_img img {
	    width: 30px;
	    height: 30px;
	    border-radius: 30px;
	    display: inline;
	    box-shadow: 0 1px 1px rgba(0,0,0,0.2);
	    float: right;
	}
	.heading_soptlight form {
	    float: right;
	    width: 100px;
	    height: 26px;
	    position: relative;
	    margin-right: 90px;
	}
	form input {
	    height:30px;
	    border-radius: 30px;
	}
	.body {
	    padding: 30px;
	    height: auto;
	    width:auto;
	}
	.body_title h3 {
	    font-size: 30px;
	    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
	    color: #333333;
	}
	.body_title p {
	    margin: 20px 0;
	}
	.footing {
	    padding:20px 0;
	    text-align: center;
	    font-size:10px;
	    color: darkgray;
	}
    </style>
</head>
<body>
    <div class="container">
        <div class="wrapper">
            <div class="heading">
                <div class="heading_nav">
                    <div class="heading_title"><em>X</em>学院</div>
                    <div class="heading_navbar">
                        <ul>
                            <li><a href="#">首页</a></li>
                            <li><a href="#">专业课程</a></li>
                            <li><a href="#">课程问答</a></li>
                            <li><a href="#">资料下载</a></li>
                        </ul>
                    </div>
                    <div class="heading_img">
                        <img src="1111.PNG">
                    </div>
                    <div class="heading_soptlight">
                        <form>
                            <input type="text">
                        </form>
                    </div>
                </div>
            </div>
            <div class="body">
                <div class="body_title">
                    <h3>X学院</h3>
                    <p>欢迎加入X学院！</p>
                </div>
                <hr/>
                <hr/>
            </div>
        </div>
        <div class="footing">
            @X学院
        </div>
    </div>
</body>
</html>
```

效果图：<br>

<img src="http://img.blog.csdn.net/20161229113513942?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br><br>








