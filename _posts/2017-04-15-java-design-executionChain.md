---
layout: post
title:  "springmvc 的 handler 和 interceptor 责任链设计模式"
desc: "springmvc 的 handler 和 interceptor 责任链设计模式"
keywords: "springmvc 的 handler 和 interceptor 责任链设计模式,责任链设计模式,springmvc,java,kinglyjn"
date: 2017-04-15
categories: [Java]
tags: [java,springnvc,design]
icon: fa-coffee
---


### 责任链模式代码执行过程

<img src="http://img.blog.csdn.net/20161228143659330?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br>
<br>


### 相关测试源码：<br>

```java
/**
* 封装请求体
*/
public class Request {
	private String content;

	public Request(String content) {
		super();
		this.content = content;
	}

	public String getContent() {
		return content;
	}

	public void setContent(String content) {
		this.content = content;
	}
}

/**
* 封装响应体
*/
public class Response {
	private String content;

	public Response(String content) {
		super();
		this.content = content;
	}

	public String getContent() {
		return content;
	}

	public void setContent(String content) {
		this.content = content;
	}
}

/**
* 过滤器接口（过滤器和过滤器链都实现该接口）
*/
public interface Filter {
	void doFilter(Request request, Response response, FilterChain chain);
}

/**
* 过滤器链
*/
public class FilterChain implements Filter {
	private List<Filter> filters = new ArrayList<Filter>();
	private int i = 0;

	public FilterChain() {
		super();
	}
	public FilterChain(List<Filter> filters) {
		this.filters = filters;
	}

	public List<Filter> getFilters() {
		return filters;
	}
	public void setFilters(List<Filter> filters) {
		this.filters = filters;
	}

	//
	public FilterChain addFilter(Filter filter) {
		filters.add(filter);
		return this;
	}
	//
	@Override
	public void doFilter(Request request, Response response, FilterChain chain) {
		if (chain.getFilters().isEmpty() || i==chain.getFilters().size()) {
			return;
		}
		Filter filter = chain.getFilters().get(i); 
		i++; //注意顺序
		filter.doFilter(request, response, chain);
	}
}

/**
* 过滤器Filter1，Filter2，Filter3
*/
public class Filter1 implements Filter {

	@Override
	public void doFilter(Request request, Response response, FilterChain chain) {
		request.setContent(request.getContent() + " filter1");
		chain.doFilter(request, response, chain);
		response.setContent(response.getContent() + " filter1");
	}
}

/**
* main
*/
public static void main(String[] args) {
	Request request = new Request("myrequest");
	Response response = new Response("myresponse");
	
	Filter1 filter1 = new Filter1();
	Filter2 filter2 = new Filter2();
	Filter3 filter3 = new Filter3();
	
	FilterChain chain = new FilterChain();
	chain.addFilter(filter1).addFilter(filter2).addFilter(filter3);
	chain.doFilter(request, response, chain);
	
	System.out.println(request.getContent());
	System.out.println(response.getContent());
}


//运行结果：
myrequest  filter1  filter2  filter3
myresponse  filter3  filter2  fileter1
```


