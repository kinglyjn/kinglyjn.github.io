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

<br>



### 反射的一些示例代码

```java
package test;
import java.lang.annotation.Annotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import org.junit.Before;
import org.junit.Test;


/**
 * 测试类
 * @author zhangqingli
 *
 */
public class Test01 {
	private Class<?> c;
	
	@Before
	public void init() {
		Student<Integer> student = new Student<Integer>();
		c = student.getClass();
	}
	
	
	/**
	 * 获取运行时类的属性
	 */
	@Test
	public void test01() {
		Field[] fields = c.getFields(); //获取本类和所有父类中公开属性
		for (Field field : fields) {
			System.out.println(field); 
		} //public int test.Person.age  public boolean test.Animal.sex
		System.out.println();
		
		
		Field[] fields2 = c.getDeclaredFields(); //获取本类中所有属性
		for (Field field : fields2) {
			System.out.println(field);
		} //private static final long test.Student.serialVersionUID
	}
	
	
	/**
	 * 获取属性的权限修饰符、类型、属性名、属性值
	 */
	@Test
	public void test02() throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException {
		Field serialVersionUIDField = c.getDeclaredField("serialVersionUID");
		
		String modifier = Modifier.toString(serialVersionUIDField.getModifiers());
		System.out.println(modifier); //private static final
		
		Class<?> type = serialVersionUIDField.getType();
		System.out.println(type); //long
		System.out.println(type.isPrimitive()); //true 判断type是否为基本类型
		
		String name = serialVersionUIDField.getName();
		System.out.println(name); //serialVersionUID
		
		if (Modifier.isStatic(serialVersionUIDField.getModifiers())) { //判断是否是静态方法
			serialVersionUIDField.setAccessible(true);
			
			long serialVersionUID = 0;
			if (type.isAssignableFrom(long.class)) { //判断type是否是long.class的子类或父类
				serialVersionUID = (long) serialVersionUIDField.get(Student.class);
				System.out.println("类型转换成功！");
				System.out.println(serialVersionUID); //1
			}
		}
	}
	
	
	/**
	 * 获取运行时方法
	 */
	@Test
	public void test03() {
		Method[] methods = c.getMethods(); //获取本类及所有父类的公开方法
		for (Method method : methods) {
			System.out.println(method);
		}
		System.out.println();
		
		Method[] methods2 = c.getDeclaredMethods(); //获本类中所有方法
		for (Method method : methods2) {
			System.out.println(method);
		}
	}
	
	
	/**
	 * 获取方法的权限修饰符、方法名称、形参列表、返回值类型、异常、注解、以及调用方法
	 */
	@Test
	public void test04() throws NoSuchMethodException, SecurityException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
		Method method = c.getDeclaredMethod("dispaly", new Class[] {String.class});
		
		String modifier = Modifier.toString(method.getModifiers());
		System.out.println(modifier); //protected
		
		String name = method.getName();
		System.out.println(name); //dispaly
		
		Class<?>[] parameterTypes = method.getParameterTypes();
		for (Class<?> class1 : parameterTypes) {
			System.out.println(class1); //class java.lang.String
		}
		
		Class<?> returnType = method.getReturnType();
		System.out.println(returnType); //void
		System.out.println(returnType.isAssignableFrom(void.class)); //true
		
		Class<?>[] exceptionTypes = method.getExceptionTypes();
		for (Class<?> class1 : exceptionTypes) {
			System.out.println(class1); //class java.lang.Exceptio
		}
		
		Annotation[] annotations = method.getAnnotations();
		for (Annotation annotation : annotations) {
			if (annotation instanceof MyAnnotation) {
				System.out.println(annotation); //@test.MyAnnotation(value=method_dispaly)
				String value = ((MyAnnotation) annotation).value();
				System.out.println(value); 	//method_dispaly
			}
		}
		
		Student<String> student = new Student<String>();
		method.invoke(student, "张三"); //张三
		
		Method method2 = c.getDeclaredMethod("haha");
		Object obj = method2.invoke(Student.class); //haha
		System.out.println(obj); //null
	}
	
	
	/**
	 * 构造器方法
	 */
	@Test
	public void test05() throws Exception {
		Constructor<?> constructor = c.getConstructor();
		Student<?> student = (Student<?>) constructor.newInstance();
		student.dispaly("李四");
	}
	
	
	/**
	 * 获取父类、父类的泛型
	 */
	@Test
	public void test06() {
		Class<?> superclass = c.getSuperclass();
		System.out.println(superclass); //获取父类  class test.Person
		
		Type genericSuperclass = c.getGenericSuperclass();
        //获取父类并且带带泛型  test.Person<java.lang.String>
		System.out.println(genericSuperclass); 
		
		Type genericSuperclass2 = c.getGenericSuperclass();
		ParameterizedType paramType = (ParameterizedType) genericSuperclass2;
		Type[] actualTypeArguments = paramType.getActualTypeArguments();
		for (Type type : actualTypeArguments) {
			System.out.println((Class<?>)type); //获取父类的泛型  class java.lang.String
		}
	}
	
	
	/**
	 * 获取接口 及 接口的泛型
	 */
	@Test
	public void test07() {
		Class<?>[] interfaces = c.getInterfaces();
		for (Class<?> class1 : interfaces) {
            //获取本类接口  interface java.lang.Comparable   interface test.MyInterface
			System.out.println(class1);
		}
		
		Type[] genericInterfaces = c.getGenericInterfaces();
		for (Type type : genericInterfaces) {
            //获取本类接口并带泛型   
            //java.lang.Comparable<test.Student<T>>  interface test.MyInterface
			System.out.println(type); 
		}
		
		Type[] genericSuperclass2 = c.getGenericInterfaces();
		ParameterizedType paramType = (ParameterizedType) genericSuperclass2[0];
		Type[] actualTypeArguments = paramType.getActualTypeArguments();
		for (Type type : actualTypeArguments) {
			System.out.println(type); //获取本类接口的泛型  test.Student<T>
		}
	}
	
	
	/**
	 * 获取包
	 */
	@Test
	public void test08() {
		Package package1 = c.getPackage();
		System.out.println(package1.getName()); //test
	}
	
	/**
	 * 获取注解
	 */
	@Test
	public void test09() {
		Annotation[] annotations = c.getAnnotations();
		
		for (Annotation annotation : annotations) {
			System.out.println(annotation); //@test.MyAnnotation(value=t_student)
		}
	}
	
}


@MyAnnotation("t_student")
class Student<T> extends Person<String> implements Comparable<Student<T>>, MyInterface {
	private static final long serialVersionUID = 1L;
	
	public Student() {
		System.out.println("无参构造器");
	}
	

	@Override
	public int compareTo(Student<T> o) {
		return 0;
	}
	
	@MyAnnotation("method_show")
	public void show() {
		System.out.println(name);
	}
	
	@MyAnnotation("method_dispaly")
	protected void dispaly(String name) throws Exception {
		System.out.println(name);
	}
	
	public static final void haha() {
		System.out.println("haha");
	}
	
	class Address {
		private String province;
		private String city;
		public String getProvince() {
			return province;
		}
		public void setProvince(String province) {
			this.province = province;
		}
		public String getCity() {
			return city;
		}
		public void setCity(String city) {
			this.city = city;
		}
	}
}

class Person<T> extends Animal {
	protected T name;
	public int age;

	public T getName() {
		return name;
	}
	public void setName(T name) {
		this.name = name;
	}
}

class Animal {
	public boolean sex = true;
}



@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@interface MyAnnotation {
	String value();
}


package test;
import java.io.Serializable;
public interface MyInterface extends Serializable {
	
}
```





