---
layout: post
title:  "java泛型小结"
desc: "java泛型小结"
keywords: "java泛型小结,泛型,java,kinglyjn"
date: 2012-11-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

### 泛型类

```java
public class Fanxinglei<T> {
	private T obj;

	public T getObj() {
		return obj;
	}

	public void setObj(T obj) {
		this.obj = obj;
	}
	
	@Override
	public String toString() {
		return "Fanxinglei [obj=" + obj + "]";
	}

	public static void main(String[] args) {
		Fanxinglei<String> fxl = new Fanxinglei<String>();
		fxl.setObj("哈哈哈");
		System.out.println(fxl);
	}
}

//运行结果
Fanxinglei [obj=哈哈哈]
```
<br>


### 泛型方法

```java
public <T extends Number> double add(T t1, T t2) {
	double sum = 0.0;
	sum = t1.doubleValue() + t2.doubleValue();
	return sum;
}

public static <T> T getObj(Class<T> c) throws Exception {
	return c.newInstance();
}
```
<br>


### 泛型接口

```java
public class Fanxing {
	public static void main(String[] args) {
		InterImpl<String> interImpl = new InterImpl<String>();
		interImpl.show("哈哈哈");
	}
}

interface Inter<T> { //interface
	void show(T t);
}

class InterImpl<T> implements Inter<T> { //interafceImpl
	@Override
	public void show(T t) {
		System.out.println(t);
	}
}
```

