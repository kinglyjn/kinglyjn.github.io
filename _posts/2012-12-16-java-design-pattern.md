---
layout: post
title:  "java开发常见设计模式"
desc: "java开发常见设计模式"
keywords: "java开发常见设计模式,设计模式,java,kinglyjn"
date: 2012-11-18
categories: [Java]
tags: [java]
icon: fa-coffee
---

### 责任链模式

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
责任链模式代码执行过程: <br>
<img src="http://img.blog.csdn.net/20161228143659330?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br>


### 监听者模式

```java
/**
 * 监听者接口
 */
public interface Listener {
	void action(Event e);
}

public class GrandPa implements Listener {
	@Override
	public void action(Event e) {
		System.out.println("grandFather hug child!  " + e);
	}
}
public class Dad implements Listener {
	@Override
	public void action(Event e) {
		System.out.println("dad feed child!  " + e);
	}
}
public class Dog implements Listener {
	@Override
	public void action(Event e) {
		System.out.println("wang wang!  " + e);
	}
}


/**
 * 事件顶层父类
 */
public abstract class Event {
}

/**
 * 具体事件
 */
import java.util.Date;
public class WakeupEvent extends Event {
	private Date time;
	private String location;
	
	public WakeupEvent(Date time, String location) {
		this.time = time;
		this.location = location;
	}

	public Date getTime() {
		return time;
	}
	public void setTime(Date time) {
		this.time = time;
	}
	public String getLocation() {
		return location;
	}
	public void setLocation(String location) {
		this.location = location;
	}

	@Override
	public String toString() {
		return "WakeupEvent [time=" + time + ", location=" + location + "]";
	}
}


/**
 * 事件源
 */
import java.util.ArrayList;
import java.util.List;
public class Child implements Runnable {
	private WakeupEvent event;
	private List<Listener> listeners = new ArrayList<Listener>();

	public WakeupEvent getEvent() {
		return event;
	}
	public void setEvent(WakeupEvent event) {
		this.event = event;
	}
	public List<Listener> getListeners() {
		return listeners;
	}
	public void setListeners(List<Listener> listeners) {
		this.listeners = listeners;
	}

	//
	public Child addListener(Listener listener) {
		listeners.add(listener);
		return this;
	}
	
	@Override
	public void run() {
		System.out.println("睡着...");
		try {
			Thread.sleep(5000);
			System.out.println("醒了!!!");
			for (Listener listener : listeners) {
				listener.action(event);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}


/**
* main
*/
public static void main(String[] args) {
	Child child = new Child();
	WakeupEvent event = new WakeupEvent(new Date(), "卧室");
	
	//
	child.setEvent(event);
	child.addListener(new GrandPa()).addListener(new Dad()).addListener(new Dog());
	
	//
	new Thread(child).start();
}


//运行结果：
睡着...
醒了!!!
grandFather hug child!  WakeupEvent [time=Wed Dec 28 15:35:14 CST 2012, location=卧室]
dad feed child!  WakeupEvent [time=Wed Dec 28 15:35:14 CST 2012, location=卧室]
wang wang!  WakeupEvent [time=Wed Dec 28 15:35:14 CST 2012, location=卧室]
```
<br>










