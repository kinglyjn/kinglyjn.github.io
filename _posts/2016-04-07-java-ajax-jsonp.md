---
layout: post
title:  "Ajax 跨域技术"
desc: "Ajax 跨域技术"
keywords: "Ajax 跨域技术,ajax,跨域,kinglyjn"
date: 2016-04-07
categories: [Java]
tags: [java]
icon: fa-coffee
---



> No 'Access-Control-Allow-Origin' header is present on the requested resource.
>
> 当使用ajax访问远程服务器时，请求失败，浏览器报如上错误。这是出于安全的考虑，默认禁止跨域访问导致的。



### 一、什么是跨域访问

**举个栗子**：在A网站中，我们希望使用Ajax来获得B网站中的特定内容。如果A网站与B网站不在同一个域中，那么就出现了跨域访问问题。你可以理解为两个域名之间不能跨过域名来发送请求或者请求数据，否则就是不安全的。跨域访问违反了同源策略，同源策略的详细信息可以点击如下链接：[Same-origin_policy](https://en.wikipedia.org/wiki/Same-origin_policy)； <br>
总而言之，同源策略规定，浏览器的ajax只能访问跟它的HTML页面同源（相同域名或IP）的资源。<br><br>



### 二、解决方案

常用的解决方案有两种，可以分为客户端解决方案和服务器端解决方案。先说服务器端解决方案。<br>

#### 服务器端解决方案 

在服务器端的filter或者servlet里面添加

```java
response.setHeader("Access-Control-Allow-Origin", "*"); 
```

“Access-Control-Allow-Origin”表示允许跨域访问，“*”表示允许所有来源进行跨域访问，这里也可以替换为特定的域名或ip。 很显然，这种方式对非网站拥有人员来说是不能做到的。而且此种方式很容易受到CSRF攻击。<br><br>



#### 客户端解决方案

首先解释什么是 jsonp？<br>

```default
JSONP(JSON with Padding)是JSON的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题。
```

由于同源策略，一般来说位于 server1.example.com 的网页无法与不是 server1.example.com的服务器沟通，而 HTML 的`<script>` 元素是一个例外。利用`<script>`元素的这个开放策略，网页可以得到从其他来源动态产生的 JSON 资料，而这种使用模式就是所谓的 JSONP。用 JSONP 抓到的资料并不是 JSON，而是任意的JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析。<br>

JQuery Ajax对JSONP进行了很好的封装，我们使用起来很方便。前端示例：

```javascript
$.ajax({
  type:"GET",
  url:"http://www.deardull.com:9090/getMySeat", //访问的链接
  dataType:"jsonp",  //数据格式设置为jsonp
  jsonp:"callback",  //Jquery生成验证参数的名称
  success:function(data){  //成功的回调函数
    alert(data);
  },
  error: function (e) {
    alert("error");
  }
});
```

需要注意的地方是：<br>

- dataType，该参数必须要设置成jsonp
- jsonp，该参数的值需要与服务器端约定，详细情况下面介绍。（约定俗成的默认值为callback）

<br>

后端的配合示例:<br>

后端要配合使用jsonp，那么首先得了解Jquery Ajax jsonp的一个特点： <br>
Jquery在发送一个Ajax jsonp请求时，会在访问链接的后面自动加上一个验证参数，这个参数是Jquery随机生成的，例如链接 

```shell
http://www.deardull.com:9090/getMySeat?callback=jQuery31106628680598769732_1512186387045&_=1512186387046
```

中，参数callback=jQuery31106628680598769732_1512186387045&_=1512186387046 就是jquery自动添加的。 添加这个参数的目的是唯一标识这次请求。当服务器端接收到该请求时，需要将该参数的值与实际要返回的json值进行构造（如何构造下面讲解），并且返回，而前端会验证这个参数，如果是它之前发出的参数，那么就会接收并解析数据，如果不是这个参数，那么就拒绝接受。 <br>

需要特别注意的是这个验证参数的名字（我在这个坑上浪费了2小时），这个名字来源于前端的**jsonp**参数的值。如果把前端jsonp参数的值改为“aaa”，那么相应的参数就应该是 
aaa=jQuery31106628680598769732_1512186387045&_=1512186387046 <br>

知道了Jquery Ajax Jsonp的原理，也知道了需要接受的参数，我们就可以来编写服务器端程序了。 
为了配合json，服务器端需要做的事情可以概括为两步：<br>

```java
// 第一步、接收验证参数
// 根据与前端Ajax约定的jsonp参数名来接收验证参数，示例如下（使用SpringMVC，其他语言及框架原理类似）
@ResponseBody
@RequestMapping("/getJsonp")
public String getMySeatSuccess(@RequestParam("callback") String callback){...}

// 第二步、构造参数并返回
// 将接收的的验证参数callback与实际要返回的json数据按“callback(json)”的方式构造：
@ResponseBody
@RequestMapping("/getMySeat")
public String getMySeatSuccess(@RequestParam("callback") String callback){
  Gson gson=new Gson();
  Map<String,String> map=new HashMap<>();
  map.put("seat","1_2_06_12");
  return callback+"("+gson.toJson(map)+")"; //构造返回值
}
```

<br>

最终，前后端的相应代码应该是这样的：<br>

**前端**

```javascript
$.ajax({
  type:"GET",
  url:"http://www.deardull.com:9090/getMySeat", //访问的链接
  dataType:"jsonp",  //数据格式设置为jsonp
  jsonp:"callback",  //Jquery生成验证参数的名称
  success:function(data){  //成功的回调函数
    alert(data);
  },
  error: function (e) {
    alert("error");
  }
});
```

**后端**

```java
@ResponseBody
@RequestMapping("/getMySeat")
public String getMySeatSuccess(@RequestParam("callback") String callback){
  Gson gson=new Gson();
  Map<String,String> map=new HashMap<>();
  map.put("seat","1_2_06_12");
  logger.info(callback);
  return callback+"("+gson.toJson(map)+")";
}
```

需要注意的是：<br>

- 前端注意与后端沟通约定jsonp的值，通常默认都是用callback。
- 后端根据jsonp参数名获取到参数后要与本来要返回的json数据按“callback(json)”的方式构造。
- 如果要测试的话记得在跨域环境（两台机器）下进行。







