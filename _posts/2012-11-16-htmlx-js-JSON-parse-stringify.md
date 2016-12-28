---
layout: post
title:  "js中的JSON对象（包含parse和stringify方法）"
desc: "js中的JSON对象（包含parse和stringify方法）"
keywords: "js中的JSON对象,JSON,parse,stringify,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [js]
icon: fa-html5
---

### js中的JSON对象

在异步应用程序中发送和接收信息时，可以选择以纯文本和 XML 作为数据格式。为了更好的使用ajax,我们将学习一种有用的数据格式 JavaScript Object Notation（JSON），以及如何使用它更轻松地在应用程序中移动数据和对象。JSON是一种简单的数据交换格式，在某些方面，它的作用与XML非常类似，但比XML更为简单，JSON的语法简化了数据交换的难度，而且提供了一种伪对象的方式。<br>

简单地说，JSON可以将 JavaScript 对象中表示的一组数据转换为字符串（伪对象），然后就可以在函数之间轻松地传递这个字符串，
或者在异步应用程序中将字符串从Web 客户端传递给服务器端程序。这个字符串看起来有点儿古怪（稍后会看到几个示例），但是 JavaScript 很容易解释它，而且 JSON 可以表示比名称/值对更复杂的结构。例如，可以表示数组和复杂的对象，而不仅仅是键和值的简单列表。<br>

关于JSON对象:

1、使用JavaScript语法创建对象

```js
//定义一个函数，作为构造函数
fucntion person(name,sex)
{
  this.name=name;
  this.sex=sex;
}
//创建一个实例
var p=new Person('张三','男');
//输出Person实例
alert(p.name);
//注意：通过该方式创建的对象是一般的脚本对象
```

2、从JavaScript1.2开始创建对象有了一种更快捷的语法(Json的语法)，如下：

```js
var obj={name:"张三","sex":'男'};    
alert(obj.sex);
```

关于数组:

1、早期的JavaScript数组

```js
var arr=new Array();
arr[0]='a';
arr[1]='bbc'

//我们也可以通过如下方式创建数组
var arr=new Array('a','bbc');
```

2、使用JSON语法，则可以通过如下方式创建数组

```js
var arr=['a','bbc'];
```

### 在JavaScript中使用JSON

```js
function showJSON() {   
    var user =    
    {    
        "username":"andy",   
        "age":20,   
        "info": { "tel": "123456", "cellphone": "98765"},   
        "address":   
            [   
                {"city":"beijing","postcode":"222333"},   
                {"city":"newyork","postcode":"555666"}   
            ]   
    }   
       
    alert(user.username);   
    alert(user.age);   
    alert(user.info.cellphone);   
    alert(user.address[0].city);   
    alert(user.address[0].postcode);   
       
    user.username = "Tom";   
    alert(user.username);   
} 
```

### js对象和json文本字符串的相互转换

```js
function myEval() {   
    var str = '{ "name": "张三", "sex": "男" }';   
    var obj = JSON.parse(str);  //或者str.parseJSON()    //或者eval('(' + str + ')') 
    alert(JSON.stringify(obj)); //或者obj.toJSONString()
}
```
<br>


