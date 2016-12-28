---
layout: post
title:  "html5的sessionStorage和localStorage"
desc: "html5的sessionStorage和localStorage"
keywords: "html5的sessionStorage和localStorage,sessionStorage,localStorage,html,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [html]
icon: fa-html5
---

### 概述

html5中的Web Storage包括了两种存储方式：sessionStorage和localStorage。sessionStorage用于本地存储一个会话（session）中的数据，这些数据只有在同一个会话中的页面才能访问并且当会话结束后数据也随之销毁。因此sessionStorage不是一种持久化的本地存储，仅仅是会话级别的存储。而localStorage用于持久化的本地存储，除非主动删除数据，否则数据是永远不会过期的。<br>

web storage和cookie的区别：<br>

1. web storage的概念和cookie相似，区别是它是为了更大容量存储设计的。cookie的大小是受限的，并且每次你请求一个新的页面的时候cookie都会被发送过去，这样无形中浪费了带宽，另外cookie还需要指定作用域，不可以跨域调用。

2. web storage拥有setItem,getItem,removeItem,clear等方法，不像cookie需要前端开发者自己封装setCookie，getCookie。

3. 但是cookie也是不可以或缺的，cookie的作用是与服务器进行交互，作为HTTP规范的一部分而存在，而web storage 仅仅是为了在本地“存储”数据而生


html5 web storage的浏览器支持情况：<br>
浏览器的支持除了IE７及以下不支持外，其他标准浏览器都完全支持(ie及FF需在web服务器里运行)。<br>
要判断浏览器是否支持localStorage可以使用下面的代码：<br>

```js
if(window.localStorage){
     alert("浏览支持localStorage") 
}else{
    alert("浏览暂不支持localStorage") 
} 

//或者 
if(typeof window.localStorage == 'undefined'){
    alert("浏览暂不支持localStorage") 
}
```
<br>


### sessionStorage和localStorage基本操作

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>HTML测试代码</title>
</head>
<body>
    <p id="msg1" style="color:green"></p>
    <input type="text" id="input1">
    <input type="button" value="保存sessionData" onclick="saveSessionStrorage('input1')">
    <input type="button" value="读取sessionData" onclick="loadSessionStorage('msg1')">
    <input type="button" value="清除sessionData" onclick="clearSessionStorage()">
    <input type="button" value="遍历sessionData" onclick="traverseSessionStorage()">
    <br/><br/>

    <p id="msg2" style="color:red"></p>
    <input type="text" id="input2">
    <input type="button" value="保存localData" onclick="saveLocalStorssge('input2')">
    <input type="button" value="读取localData" onclick="loadLocalStorage('msg2')">
    <input type="button" value="清除localData" onclick="clearLocalStorage()">
    <input type="button" value="遍历localData" onclick="traverseLocalStorage()">

    <script type="text/javascript">
        //HTML5保存、获取、删除、清除、遍历当前本地会话数据
        //[注]当前回话是指 当前访问的网页，该网页关闭，当前会话立即会消失，因此sessionStorage不是一种持久化的本地存储，仅仅是会话级别的存储
        var saveSessionStrorage = function(id) {
            var target = document.getElementById(id);
            var str = target.value;
            sessionStorage.setItem("message1", str);
        }
        var loadSessionStorage = function(id) {
            var msg = sessionStorage.getItem("message1");
            var target = document.getElementById(id);
            target.innerHTML = msg;
        }
        var removeSessionStorage = function(key) {
            sessionStorage.removeItem(key);
        }
        var clearSessionStorage = function() {
            sessionStorage.clear();
        }
        var traverseSessionStorage = function() {
            var str = "";
            for(var i=0; i<sessionStorage.length; i++) {
                var key = sessionStorage.key(i);
                str += sessionStorage.getItem(key) + "  ";
            }
            alert(str);
        }

        //HTML5保存、获取、删除、清除、遍历持久化本地存储数据
        var saveLocalStorssge = function(id) {
            var target = document.getElementById(id);
            var str = target.value;
            localStorage.setItem("message2", str);
        }
        var loadLocalStorage = function(id) {
            var msg = localStorage.getItem('message2');
            var target = document.getElementById(id);
            target.innerHTML = msg;
        }
        var removeLocalStorage = function(key) {
            localStorage.removeItem(key);
        }
        var clearLocalStorage = function() {
            localStorage.clear();
        }
        var traverseLocalStorage = function() {
            var str = "";
            for(var i=0; i<localStorage.length; i++) {
                var key = localStorage.key(i);
                str += localStorage.getItem(key) + "  ";
            }
            alert(str);
        }
    </script>
</body>
</html>
```

测试效果：<br>

<img src="http://img.blog.csdn.net/20161228165547864?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br><br>


### 使用localstorage实现简单的本地数据库功能

```html
<!--html代码-->
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>webStorage实现简单的数据库功能</title>
    <link href="mysyle.css" rel="stylesheet">
</head>
<body>
    <input type="button" value="清空本地库" onclick="clearLocalStorage()"><br/><br/>

    <input type="text" id="name" name="name" placeholder="请输入姓名"><br/>
    <input type="text" id="email" name="email" placeholder="请输入邮箱"><br/>
    <input type="text" id="tel" name="tel" placeholder="请输入电话号码"><br/>
    <input type="text" id="info" name="info" placeholder="请输入简介"><br/>
    <input type="button" value="保存" onclick="saveLocalStorage()"><br/>
    <hr/>

    <input type="search" id="search" placeholder="查询关键字">
    <input type="button" value="查询" onclick="searchLocalStorage('search', 'msg')"><br/><br/>
    <p id="msg"></p>

    <script src="appWeb.js"></script>
</body>
</html>
```

```js
//appWeb.js 

var saveLocalStorage = function() {
    var data = new Object();
    var name = document.getElementById("name").value;
    var email = document.getElementById("email").value;
    var tel = document.getElementById("tel").value;
    var info = document.getElementById("info").value;
    data.name = name;
    data.email = email;
    data.tel = tel;
    data.info = info;
    var dataJson = JSON.stringify(data);
    localStorage.setItem(name, dataJson);
    alert("保存成功！");
}

var searchLocalStorage = function(searchId, destinationId) {
    var target = document.getElementById(destinationId);
    target.innerHTML = "";
    var searchKey = document.getElementById(searchId).value;
    var dataJson = localStorage.getItem(searchKey);
    if (dataJson==null) {
        alert("暂无数据！");
        return;
    }
    var data = JSON.parse(dataJson);
    var result = "姓名：" + data.name + "<br/>"
    result += "邮箱：" + data.email + "<br/>";
    result += "电话：" + data.tel + "<br/>";
    result += "简介：" + data.info + "<br/>";
    target.innerHTML = result;
}

var clearLocalStorage = function() {
    localStorage.clear();
    alert("清除成功！");
}
```

测试效果：<br>

<img src="http://img.blog.csdn.net/20161228170652660?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br>

