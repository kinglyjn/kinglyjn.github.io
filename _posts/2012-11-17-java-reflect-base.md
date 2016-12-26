---
layout: post
title:  "java反射基础"
desc: "java反射基础"
keywords: "java反射基础,反射,java,kinglyjn"
date: 2012-11-17
categories: [Java]
tags: [java]
icon: fa-coffee
---

### 概述

反射是Java中非常重要的一个语言特性，反射的强大和完善，让Java语言在工程实践中的灵活性大大的增强，使得Java程序在运行时可以探查类的信息，动态的创建类的对象，获知对象的属性，调用对象的方法。因此，反射技术被广泛的应用在一些工具和框架的开发上。也许，并不是每一个程序员都有机会利用反射API进行他们的Java开发，但是，学习反射是一个Java程序员必须要走过的道路之一，对反射的掌握能够帮助程序员更好的理解后面很多的框架和Java工具，毕竟这些框架和工具都是采用反射作为底层技术的。<br>

### 类对象

Java中有一个类，`java.lang.Class`，这个类的对象被称为类对象。<br>

那类对象用来干什么呢？比如，以前我们写过学生类，一个学生对象都是用来保存一个学生的信息。而一个类对象呢，则用来保存一个类的信息。所谓类的信息，包括：

* 这个类继承自哪个类
* 实现了哪些接口
* 有哪些属性
* 有哪些方法
* 有哪些构造方法
* 有哪些注解...

我们之前提到过类加载的概念。当JVM第一次遇到某个类的时候，会通过CLASSPATH找到相应的.class文件，读入这个文件并把读到的类的信息保存起来。而类的信息在JVM中，则被封装在了类对象中。<br>

获取类对象的三种方式：<br>
//要获得这个类的类对象，必须要先加载这个类。Class.forName("test.Student")就会触发类加载的动作。<br>

* Class<Student> c = Student.class
* Class<Student> c = stu.getClass()
* Class<Student> c = Class.forName("test.Student") 

类对象方法：<br>

```java
c.getName() //获得类的名字，包括包名
c.getSimpleName() //获得类的名字，不包括包名
c.getSuperclass() //获得本类的父类的类对象
c.getInterfaces() //获得本类所实现的所有接口的类对象，返回值为Class[]

c.getField("result")  //
c.getDeclaredFields() //返回本类的所有属性
c.getFields()  //返回它及其父类所有[公开]属性,返回一个Field类型的数组
c.getMethod("getResult", new Class[]{String.class, int.class}) //
c.getDeclaredMethods() //返回本类的所有方法
c.getMethods() //返回它及其父类所有[公开]方法,返回一个Method类型的数组

c.newInstance() //通过类的无参构造构造方法创建一个类对象
c.getConstructor(String.class, int.class..).newInstance("str", 1, ..) //通过有参构造方法创建对象
...
```
<br>

### 使用反射修改私有属性及调用私有方法

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ReflectTest {
	public static void main(String[] args) throws Exception {
		//创建对象
		Class<?> c = Class.forName("test.Student");
		Constructor<?> cons = c.getConstructor(String.class, int.class);
		Student stu = (Student) cons.newInstance("张三", 23);
		
		//利用反射调用对象的私有方法
		Method toString = c.getDeclaredMethod("toString1", new Class[]{});
		toString.setAccessible(true); //
		String str = (String) toString.invoke(stu, new Object[]{});
		System.out.println(str);
		
		//利用反射修改私有属性
		Field age = c.getDeclaredField("age");
		age.setAccessible(true); //
		age.set(stu, 25);
		System.out.println(stu.getAge()); //25
	}
}	

class Student {
	private String name;
	private int age;
	
	public Student(String name, int age) {
		this.name = name;
		this.age = age;
	}

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}

	private String toString1() { //注意这是私有方法
		return "Student [name=" + name + ", age=" + age + "]";
	}
}
```
有上述测试我们知道，反射技术可以访问和修改一个对象的私有属性，可以调用一个对象的私有方法。那么，这样算不算破坏封装呢？严格的说，算。但是，这种对封装的破坏并不可怕。要明确的是，反射是一种非常底层的技术，而封装相对来说是一个比较高级的概念。例如，一台服务器，要防止外部的破坏，有可能会假设一道网络防火墙。防火墙这个概念就是一个相对比较高级的概念。而这个防火墙设计的再合理，如果服务器机房的钥匙被人偷走了，让人能够进入机房偷走服务器，那么防火墙设计的再好也拦不住。防火墙防止的是高层的攻击，而底层的破坏，不需要防火墙处理。封装也一样。封装防止的是程序员直接访问和操作一些私有的数据；而反射是一个非常底层的技术，利用反射，完全可以打破封装。<br>

在反射代码中，创建对象所采用的类名“Student”，以及调用方法时的方法名“study”都是以字符串的形式存在的，而字符串的值完全可以不写在程序中，比如，从文本文件中读取。这样，如果需求改变了，需要创建的对象不再是Student对象，需要调用的方法也不再是study方法，那么程序有没有可能不做任何修改呢？当然可能，你需要修改的可能是那个文本文件。反观不用反射的代码，它只能创建Student对象，只能调用study方法，如有改动则必须修改代码重新编译。明白了吧，用反射的代码，会更通用，更万能！	因此，利用反射实现的代码，可以在最大程度上实现代码的通用性，而这正是工具和框架在编写的时候所需要的。因此，反射才能在这些领域得到用武之地。<br>

当然，这里并不是鼓励大家滥用反射。反射技术有着非常显著的几个缺点。<br>

1. 运行效率与不用反射的代码相比会有明显下降。
2. 代码的复杂度大幅提升，这个从代码量上大家就能比较出来
3. 代码会变得脆弱，不易调试。使用反射，我们就在一定程度上绕开了编译器的语法检查，例如，用反射去调用一个对象的方法，而该对象没有这个方法，那么在编译时，编译器是无法发现的，只能到运行时由JVM抛出异常。

因此，反射作为一种底层技术，只适合于工具软件和框架程序的开发，在大部分不需要使用反射的场合，没有必要为了追求程序的通用性而随意使用反射。滥用反射绝对是一个坏的编程习惯。





