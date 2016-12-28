---
layout: post
title:  "html5本地数据库"
desc: "html5本地数据库"
keywords: "html5本地数据库,web sql database,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [js]
icon: fa-html5
---

### 概述

Web SQL数据库API实际上不是HTML5规范的组成部分，而是单独的规范。它通过一套API来操纵客户端的数据库。Safari、Chrome、Firefox、Opera等主流浏览器都已经支持Web SQL Database。<br>

虽然Html5已经提供了功能强大的localStorage和sessionStorage，但是他们两个都只能提供存储简单数据结构的数据，对于复杂的Web应用的数据却无能为力。逆天的是Html5提供了一个浏览器端的数据库支持，允许我们直接通JS的API在浏览器端创建一个本地的数据库，而且支持标准的SQL的CRUD操作，让离线的Web应用更加方便的存储结构化的数据。接下里介绍一下本地数据的相关API和用法。<br>

操作本地数据库的最基本的步骤是：<br>

1. 第一步：openDatabase方法：创建一个访问数据库的对象。
2. 第二步：使用第一步创建的数据库访问对象来执行transaction方法，通过此方法可以设置一个开启事务成功的事件响应方法，在事件响应方法中可以执行SQL.
3. 第三步：通过executeSql方法执行查询，当然查询可以是：CRUD。 

<br>

接下来分别介绍一下相关的方法的参数和用法：<br>

openDatabase方法：

```default
//获取或者创建一个数据库，如果数据库不存在那么创建之
var dataBase = openDatabase("student", "1.0", "学生表", 1024 * 1024, function () { });  

//openDatabase方法打开一个已经存在的数据库，如果数据库不存在，它还可以创建数据库。几个参数意义分别是： 
1. 数据库名称。
2. 数据库的版本号，目前来说传个1.0就可以了，当然可以不填；
3. 对数据库的描述。
4. 设置分配的数据库的大小（单位是kb）。
5. 回调函数(可省略)。  初次调用时创建数据库，以后就是建立连接了。
```

db.transaction方法：

```default
db.transaction方法可以设置一个回调函数，此函数可以接受一个参数就是我们开启的事务的对象。然后通过此对象可以进行执行Sql           脚本，跟下面的步骤可以结合起来。
```

executeSql方法：

```default
通过executeSql方法执行查询。
ts.executeSql(sqlQuery,[value1,value2..],dataHandler,errorHandler)

参数说明：
sqlQuery：
需要具体执行的sql语句，可以是create、select、update、delete；

[value1,value2..]：
sql语句中所有使用到的参数的数组，在executeSql方法中，将sql语句中所要使用的参数先用“?”代替，
然后依次将这些参数组成数组放在第二个参数中

dataHandler：
执行成功是调用的回调函数，通过该函数可以获得查询结果集；

errorHandler：
执行失败时调用的回调函数；
```
<br>

### 通过本地素具库实现简单的记事本功能

```html
<!--html代码-->
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>HTML5本地数据库</title>
</head>
<body onload="init()">
    <input type="button" value="清空数据" onclick="deleteAllData()"><br/><br/>

    <table>
        <tr>
            <td>姓名</td>
            <td><input type="text" id="name"></td>
        </tr>
        <tr>
            <td>留言</td>
            <td><input type="text" id="memo"></td>
        </tr>
        <tr>
            <td colspan="2"><input type="button" value="保存" onclick="saveData()"></td>
        </tr>
    </table>
    <hr/>

    <table id="datatable" border="1">
    </table>
    <p id="msg"></p>

    <script src="appWeb.js"></script>
</body>
</html>
```

```js
//appWeb.js

//创建访问数据库的对象
//使用数据处理
var datatable = null;
var db = openDatabase("MyData", "", "my database", 1024*100);

function init() {
    datatable = document.getElementById("datatable");
    showAllData();
}

function removeAllData() {
    for (var i=datatable.childNodes.length-1; i>=0; i--) {
        datatable.removeChild(datatable.childNodes[i]);
    }
    var tr = document.createElement("tr");
    var th1 = document.createElement("th");
    var th2 = document.createElement("th");
    var th3 = document.createElement("th");
    th1.innerHTML = "姓名";
    th2.innerHTML = "留言";
    th3.innerHTML = "时间";
    tr.appendChild(th1);
    tr.appendChild(th2);
    tr.appendChild(th3);
    datatable.appendChild(tr);
}

function showData(row) {
    var tr = document.createElement("tr");
    var td1 = document.createElement("td");
    td1.innerHTML = row.name;
    var td2 = document.createElement("td");
    td2.innerHTML = row.message;
    var td3 = document.createElement("td");
    var t = new Date();
    t.setTime(row.time);
    td3.innerHTML = t.toLocaleDateString() + " " + t.toLocaleTimeString();
    tr.appendChild(td1);
    tr.appendChild(td2);
    tr.appendChild(td3);
    datatable.appendChild(tr);
}

function showAllData() {
    db.transaction(function(tx){
        tx.executeSql("create table if not exists MsgData(name TEXT, message TEXT, time INTEGER)", []);
        tx.executeSql("select * from MsgData", [], function(tx, rs){
            removeAllData();
            for (var i=0; i<rs.rows.length; i++) {
                showData(rs.rows.item(i));
            }
        });
    });
}

function addData(name, message, time) {
    db.transaction(function(tx){
        tx.executeSql(
            "insert into MsgData values(?,?,?)",
            [name, message, time],
            function(tx, rs){
                alert("插入成功！");
            },
            function(tx, error) {
                alert(error.source +"::" + error.message);
            }
        );
    });
}

function saveData() {
    var name = document.getElementById("name").value;
    var memo = document.getElementById("memo").value;
    var time = new Date().getTime();
    addData(name, memo, time);
    showAllData();
}

function deleteAllData() {
    db.transaction(function(tx){
        tx.executeSql("delete from MsgData",[],function(tx,rs){
            alert("清除成功！");
        });
    });
    showAllData();
}
```

执行效果：<br>

<img src="http://img.blog.csdn.net/20161228182414594?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br>






