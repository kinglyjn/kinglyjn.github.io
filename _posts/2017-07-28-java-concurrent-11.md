---
layout: post
title:  "JAVA中的13个原子操作类"
desc: "JAVA中的13个原子操作类"
keywords: "JAVA中的13个原子操作类,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



> 当程序更新一个变量时，如果多个线程同时更新，可能得到预期之外的结果。比如变量i=1，A线程更新i+1，B线程也更新i+1，经过两个线程操作之后可能i不等于3，而是等于2。因为AB在更新变量i的时候拿到的i都是1，这就是线程不安全的更新操作，通常我们会使用synchronized来解决这个问题。而java从1.5就开始提供了java.util.concurrent.atomic包，这个包的原子操作类提供了一种用法简单、性能高效、线程安全的更新一个变量的方式。java.util.concurrent.atomic包里一共提供了四大类13个原子操作类，这四类分别是 原子更新基本类型、原子更新数组、原子更新引用、原子更新属性。java.util.concurrent.atomic包里的类基本都是使用Unsafe实现的包装类。



### 原子更新基本类型类

共有三个：

* AtomicBoolean
* AtomicInteger
* AtomicLong

以上三个类提供的方法基本都一样，所以就以AtomicInteger为例讲解。AtomicInteger常用的方法如下:

```java
//以原子的方式将输入的值与实例中的值相加并返回结果
int addAndGet(int delta)
  
//如果输入的值等于预期的值，就以原子的方式将该值设置为输入的值  
boolean compareAndSet(int expect, int update)  
 
//以原子的方式将当前的值+1，返回的是自增前的值 
int getAndIncrement()
  
//最终会设置成newValue，使用lazySet设置后，可能导致其他线程在之后的一小段时间内还是可以读到旧的值
void lazySet(int newValue)
  
//以原子的方式设置为newValue，并返回旧值
int getAndSet(int newValue)
```

使用示例代码：

```java
import java.util.concurrent.atomic.AtomicInteger;
public class AtomicIntegerTest {
	public static void main(String[] args) {
		AtomicInteger ai = new AtomicInteger(1);
		System.out.println(ai.getAndIncrement());
		System.out.println(ai);
	}
}

//运行结果：
1
2
```

那么getAndIncrement是如何实现原子操作的呢？让我们来看一下getAndIncrement的源码（java1.7）：

```java
public final int getAndIncrement() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return current;
    }
}

public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

源码中for循环体的第一步是先取得AtomicInteger里存储的值，第二步对AtomicInteger里的当前值+1，关键的第三部是调用compareAndSet方法来进行原子更新操作，该方法先检验当前数值是否等于current，等于则就意味着AtomicInteger的值没有被其他线程修改过，则AtomicInteger的值会更新成新的next的值，如果不等于则compareAndSet返回false，程序会在for循环中再次进行compareAndSet操作，直到设置成功返回。

Atomic包只提供了三种基本类型的原子更新类，但是java的基本类型里还有char、float、double等。那么问题来了。如何原子地更新其他基本类型呢？Atomic包里的类基本都是使用Unsafe类实现的，让我们看一下Unsafe的源码（java1.7）：

```java
public final native boolean compareAndSwapObject(Object paramObject1, 
                                                 long paramLong, 
                                                 Object paramObject2,
												 Object paramObject3);

public final native boolean compareAndSwapInt(Object paramObject, 
                                                  long paramLong, 
                                                  int paramInt1, 
                                                  int paramInt2);

public final native boolean compareAndSwapLong(Object paramObject, 
                                                   long paramLong1, 
                                                   long paramLong2,
                                                   long paramLong3);
```

通过源码，我们发现Unsafe只提供了3中CAS方法，再看看AtomicBoolean的源码，发现，他是先把Boolean转化为整型，再使用compareAndSwapInt进行CAS，所以原子更新char、float、double等变量也可以使用类似的思路来实现。

```java
public final boolean compareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
    return unsafe.compareAndSwapInt(this, valueOffset, e, u);
}	
```

<br>



### 原子更新数组

通过原子的方式更新数组里的某个元素，Atomic包里提供了一下四个类：

* AtomicIntegerArray
* AtomicLongArray
* AtomicDoubleArray
* AtomicReferenceArray

AtomicIntegerArray主要提供原子更新整型数组的元素，其常用的方法如下：

* int addAndGet(int i, int delta)：以原子的方式将输入值与index为i的元素相加
* boolean compareAndSet(int i, int expect, int update)：如果当前值等于预期的值，则以原子的方式将index为i的值设置成update值。

测试代码如下：

```java
import java.util.concurrent.atomic.AtomicIntegerArray;
public class AtomicIntegerTest {
	public static void main(String[] args) {
		int[] array = new int[] {1,2,3};
		AtomicIntegerArray aia = new AtomicIntegerArray(array);
		aia.getAndSet(0, 4);
		System.out.println(aia.get(0));
		System.out.println(array[0]);
	}
}

//运行结果
4
1
```

需要注意的是，数组value通过构造方法传递进去，AtomicIntegerArray会将当前数组复制一份，所以AtomicIntegerArray对内部的数组元素进行修改的时候，不会影响传入的数组。

<br>



### 原子更新引用类型

原子更新基本类型的AtomicInteger只能原子更新一个变量，如果要原子更新多个变量的话，就需要使用这个院系更新引用类型提供的类。包括如下三个：

* AtomicReference：原子更新引用类型
* AtomicReferenceFieldUpdater：原子更新引用类型里的字段
* AtomicMarkableReference：原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和引用类型。构造方法是 AtomicMarkableReference(V initailRef, boolean initailMark)

以上几个类提供的方法几乎一样，所以以AtomicReference为例：

```java
import java.util.concurrent.atomic.AtomicReference;

public class AtomicIntegerTest {
	public static void main(String[] args) {
		AtomicReference<User> atomicRefUser = new AtomicReference<User>();
		
		User user1 = new User("zhangsan", 23);
		atomicRefUser.set(user1);
		
		User user2 = new User("xiaojuan", 18);
		atomicRefUser.compareAndSet(user1, user2);
		
		System.out.println(atomicRefUser.get().getName());
		System.out.println(atomicRefUser.get().getAge());
		System.out.println(user1.getName());
		System.out.println(user1.getAge());
	}

	static class User {
		private String name;
		private int age;

		public User(String name, int age) {
			super();
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
	}
}

//运行结果
xiaojuan
18
zhangsan
23
```

<br>



### 原子更新字段类

* AtomicIntegerFieldUpdater
* AtomicLongFieldUpdater
* AtomicStampedFieldUpdater：原子更新带有版本号的引用类型，该类将整数值与引用类型关联起来，可用于原子地更新数据和数据的版本号可以解决使用CAS进行原子更新时出现的ABA问题。

> CAS看起来很爽，但是会导致“ABA问题”。CAS算法实现一个重要前提需要取出内存中某时刻的数据，而在下时刻比较并替换，那么在这个时间差类会导致数据的变化。比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。如果链表的头在变化了两次后恢复了原值，但是不代表链表就没有变化。因此前面提到的原子操作AtomicStampedReference/AtomicMarkableReference就很有用了。这允许一对变化的元素进行原子操作。

要想原子地更新字段类型需要两步：

* 第一步，因为燕子更新字段类都是抽象类，每次使用的时候必须使用静态的方法newUpdater()创建一个更新器，并且需要设置想要更新的类和属性
* 第二步，更新类的字段（属性）必须使用public volatile修饰符

以上三个类提供的方法基本一样，以AtomicIntegerFieldUpdater为例讲解：

```java
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;

public class AtomicIntegerTest {
	public static void main(String[] args) {	
		AtomicIntegerFieldUpdater<User> userAgeUpdater = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");
		
		User user = new User("zhangsan", 23);
		System.out.println(userAgeUpdater.getAndIncrement(user)); //23
		System.out.println(userAgeUpdater.get(user)); //24
		System.out.println(user.getAge()); //24
	}

	static class User {
		private String name;
		public volatile int age;

		public User(String name, int age) {
			super();
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
	}
}

//运行结果
23
24
24
```

