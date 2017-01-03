---
layout: post
title:  "工具 - gson解析json"
desc: "工具 - gson解析json"
keywords: "工具 - gson解析json,gson,json,java,kinglyjn"
date: 2012-11-18
categories: [Java]
tags: [java,gson]
icon: fa-coffee
---

> gson是google解析json的一个开源框架,同类的框架fastJson,JackJson等。

<br>


### 使用gson处理html特殊字符

```java
public static void main(String[] args) {
	Map<String, String>	 map = new HashMap<String, String>();
	map.put("s", "\u003chtml\u003e");
	
	Gson gson = new GsonBuilder().disableHtmlEscaping().create(); //disableHtmlEscaping
	String json = gson.toJson(map);
	
	System.out.println(json);
}

//运行结果：
{"s":"<html>"}

//[注] 如果在创建gson时不使用disableHtmlEscaping，则运行结果将会是：
{"s":"\u003chtml\u003e"}
```
<br>

### 使用gson解析时格式化json结果

```java
Gson gson = new GsonBuilder().setPrettyPrinting().create();
```

示例结果：

<img src="http://img.blog.csdn.net/20161229154505514?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/><br>





