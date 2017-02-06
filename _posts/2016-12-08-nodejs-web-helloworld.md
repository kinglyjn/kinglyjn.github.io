---
layout: post
title:  "NodeJs Web - Hello World"
desc: "NodeJs Web - Hello World"
keywords: "NodeJs Web - Hello World,nodejs"
date: 2016-12-08
categories: [Other]
tags: [Other]
icon: fa-plus-circle
---


> 开发环境Nodeclipse V8

```js
 /**
 * 主程序
 * http://usejsdoc.org/
 */
var server = require("./server");
var router = require("./router");
var requestHandlers = require("./requestHandlers");
var handle = {};
handle["/"] = requestHandlers.start;
handle["/start"] = requestHandlers.start;
handle["/exectest"] = requestHandlers.exectest;
handle["/upload"] = requestHandlers.upload;
server.start(router.route, handle);



/**
 * 服务器
 * http://usejsdoc.org/
 */
var http = require("http");
var url = require("url");
function start(route, handle) {
 http.createServer(function(request, response){
  var pathname = url.parse(request.url).pathname;
  var params = url.parse(request.url, true).query;
  console.log("Request for " + pathname + " received!"); //请求路径
  console.log("Request params " + params); //请求参数 object [若parse函数参数为false则解析为被&分割的字符串]
  
  //request设置
  var postData = "";
  request.setEncoding("utf-8");
  //接收到一小块post请求数据时触发
  request.addListener("data", function(postDataChunk){ 
   postData += postDataChunk;
   console.log("Received POST data chunk '"+ postDataChunk + "'");
  });
  //接收完数据时触发（没有则直接被触发）
  request.addListener("end", function(){ 
   //路由到具体的处理函数
   route(pathname, handle, response, postData);
  });
 }).listen(8080);
 
 console.log("server has start!");
}
exports.start = start; 



/**
 * 路由器
 * http://usejsdoc.org/
 */
function route(pathname, handle, response, postData) {
 console.log("About to route a request for " + pathname);
 if (typeof handle[pathname] === 'function') {
  handle[pathname](response, postData);
 } else {
  console.log("No request handler found for " + pathname);
  response.writeHead(404, {"Content-Type":"text/plain"});
  response.write("404 Not Found!");
  response.end();
 }
}
exports.route = route; 



/**
 * 请求处理器
 * http://usejsdoc.org/
 */
var exec = require("child_process").exec;
var querystring = require("querystring");
function start(response, postData) {
 console.log("Request handler 'start' was called!");
 response.writeHead(200, {"Content-Type":"text/html"});
 var body = '<html>'+
       '<head><meta http-equiv="Content-Type" content="text/html;charset=UTF-8" /></head>'+
       '<body>'+
        '<form action="/upload" method="post">'+
        '<textarea name="text" rows="20" cols="60"></textarea><br/>'+
        '<input type="submit" value="Submit text" />'+
        '</form>'+
       '</body>'+
       '</html>';
 response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}
function exectest(response, postData) {
 console.log("Request handler 'exectest' was called!");
 //EXEC非阻塞方法测试
 exec("ping www.baidu.com", {timeout:10000, maxBuffer:2*1024*1024}, function(error, stdout, stderr){
  console.log(error + " " + stdout + " " + stderr);
  response.writeHead(200, {"Content-Type":"text/plain"});
  response.write(stdout);
  response.end();
 });
}
function upload(response, postData) {
 console.log("Request handler 'upload' was called!");
 console.log(formidable);
 response.writeHead(200, {"Content-Type":"text/plain"});
 response.end();
}
exports.start = start;
exports.exectest = exectest;
exports.upload = upload; 
```

运行效果：<br>

<img src="http://img.blog.csdn.net/20170123141059590?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br>
