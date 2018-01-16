---
layout: post
title:  "java开发常见设计模式"
desc: "java开发常见设计模式"
keywords: "java开发常见设计模式,设计模式,java,kinglyjn"
date: 2012-12-16
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
<br><br>


### 遍历器模式

```java
/**
* 遍历器接口
*/
public interface Iterator<E> {
	boolean hasNext();
	E next();
}


/**
* 集合接口
*/
public interface Collection<E> {
	void add(E e);
	int size();
	Iterator<E> iterator();
}


/**
* 集合实现类ArrayList
*/
package test;

public class ArrayList<E> implements Collection<E> {
	private Object[] array = new Object[10];
	private int index = 0;

	@Override
	public void add(E e) {
		array[index] = e;
		index++;
		if (index == array.length) {
			Object[] newArray = new Object[array.length+10];
			System.arraycopy(array, 0, newArray, 0, array.length);
			array = newArray;
		}
	}

	@Override
	public int size() {
		return index;
	}

	@Override
	public Iterator<E> iterator() {
		return new Iterator<E>() {
			private int currentIndex = -1;
			
			@Override
			public boolean hasNext() {
				if (currentIndex >= index-1) {
					return false;
				}
				return true;
			}

			@SuppressWarnings("unchecked")
			@Override
			public E next() {
				currentIndex++;
				return (E) array[currentIndex];
			}
			
		};
	}
	
	//
	@SuppressWarnings("unchecked")
	public E get(int i) {
		return (E) array[i];
	}
}


/**
* 集合实现类LinkedList
*/
public class LinkedList<E> implements Collection<E> {
	private Node preHead;
	private Node head;
	private Node tail;
	private int index = 0;
	
	class Node {
		private E data;
		private Node nextNode;
		public E getData() {
			return data;
		}
		public void setData(E data) {
			this.data = data;
		}
		public Node getNextNode() {
			return nextNode;
		}
		public void setNextNode(Node nextNode) {
			this.nextNode = nextNode;
		}
	}
	
	@Override
	public void add(E e) {
		if (head == null) {
			head = tail = new Node();
			head.setData(e);
			preHead = new Node();
            preHead.setNextNode(head);
		} else {
			Node newNode = new Node();
            newNode.setData(e);
            tail.setNextNode(newNode);
            tail = newNode;
		}
		index++;
	}

	@Override
	public int size() {
		return index;
	}

	@Override
	public Iterator<E> iterator() {
		return new Iterator<E>() {
			private Node currentNode = preHead;
			
			@Override
			public boolean hasNext() {
				return currentNode.getNextNode()==null ? false : true;
			}

			@Override
			public E next() {
				currentNode = currentNode.getNextNode();
                return currentNode.getData();
			}
		};
	}
}


/**
* 测试方法
*/
public static void main(String[] args) {
	Collection<String> list = new LinkedList<String>();
	list.add("张三");
	list.add("李四");
	list.add("小娟");
	
	Iterator<String> it = list.iterator();
	while (it.hasNext()) {
		System.out.println(it.next());
	}
}

//运行结果说明  
//遍历器模式适合运用在 屏蔽同类事物的相同操作上
张三
李四
小娟
```
<br><br>


### 策略模式

```java
/**
* Comparable接口
*/
public interface Comparable<T> {
	int compareTo(T t);
}


/**
* Comparator接口
*/
public interface Comparator<T> {
	int compare(T o1, T o2);
}

/**
* Comparator接口实现类 DogWeightComparator
*/
public class DogWeightComparator implements Comparator<Dog> {
	private static final DogWeightComparator INSTANCE = new DogWeightComparator();
    private DogWeightComparator(){}
    
    public static final DogWeightComparator getInstance() {
        return INSTANCE;
    }
    @Override
    public int compare(Dog o1, Dog o2) {
        if (o1.getWeight() < o2.getWeight()) {
            return 1;
        } else if (o1.getWeight() > o2.getWeight()) {
            return -1;
        }
        return 0;
    } 
}

/**
* Comparator接口实现类 DogHeightComparator
*/
public class DogHeightComparator implements Comparator<Dog> {
    private static final DogHeightComparator INSTANCE = new DogHeightComparator();
    private DogHeightComparator(){}
    
    public static final DogHeightComparator getInstance() {
        return INSTANCE;
    }
    
    @Override
    public int compare(Dog o1, Dog o2) {
        if (o1.getHeight() < o2.getHeight()) {
            return -1;
        } else if (o1.getHeight() > o2.getHeight()) {
            return 1;
        }
        return 0;
    }
}


/**
* 对象元素Dog
*/
public class Dog implements Comparable<Dog> {
	private int weight;
    private int height;
    private static Comparator<Dog> comparator = DogWeightComparator.getInstance(); // 默认按照狗的重量排序

    public Dog(int weight, int height) {
        this.weight = weight;
        this.height = height;
    }
    
    public int getWeight() {
        return weight;
    }
    public void setWeight(int weight) {
        this.weight = weight;
    }
    public int getHeight() {
        return height;
    }
    public void setHeight(int height) {
        this.height = height;
    }
    public static Comparator<Dog> getComparator() {
        return comparator;
    }
    public static void setComparator(Comparator<Dog> comparator) {
        Dog.comparator = comparator;
    }
    @Override
    public String toString() {
        return "Dog [weight=" + weight + ", height=" + height + "]";
    }

    @Override
    public int compareTo(Dog o) {
        return comparator.compare(this, o);
    }
}


/**
* 工具类 Arrays
*/
public class Arrays {
	
	@SuppressWarnings({ "rawtypes", "unchecked" })
	public static void sort(Object[] os) {
		for (int i = 0; i < os.length; i++) {
            for (int j = 0; j < os.length-i-1; j++) {
                Comparable o1 = (Comparable) os[j];
                Comparable o2 = (Comparable) os[j+1];
                
                if (o1.compareTo(o2) < 0) {
                	Object tmp = os[j];
                    os[j] = os[j+1];
                    os[j+1] = tmp;
                }
            }
        }
	}
	
}


/**
* 测试方法
*/
public static void main(String[] args) {
	Dog[] dogs = {new Dog(7, 3), new Dog(2, 2), new Dog(5, 6)};
	//Dog.setComparator(DogWeightComparator.getInstance()); //按狗的重量排序测试
    //Dog.setComparator(DogHeightComparator.getInstance()); //按狗的身高排序策略
	
	Arrays.sort(dogs);  //要求Dog类实现Compareble接口，默认按狗的重量排序
	
	for (int i = 0; i < dogs.length; i++) {
        System.out.println(dogs[i]);
    }
}
```
<br><br>


### 桥接模式

<img src="http://img.blog.csdn.net/20161229170613706?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:50%"/><br>

```java
/**
* 桥接抽象类
*/
public abstract class Gift {
    String name;
    GiftImpl giftImpl; // 用聚合或组合代替继承
}


/**
* 桥接实现类顶层父类
*/
public class GiftImpl extends Gift {
    @Override
    public String toString() {
        return "name: " + name + " giftImpl: " + giftImpl.getClass();
    }
}


/**
* 桥接具体实现类（各种实现类的定义之间可能存在交叉）
*/
public class WarmGift extends GiftImpl {
    public WarmGift() {}
    public WarmGift(String name, GiftImpl giftImpl) {
        this.name = name;
        this.giftImpl = giftImpl;
    }
}
public class ColdGift extends GiftImpl {
    public ColdGift() {}
    public ColdGift(String name, GiftImpl giftImpl) {
        this.name = name;
        this.giftImpl = giftImpl;
    }
}
public class FlowerGift extends GiftImpl {
    public FlowerGift() {}
    public FlowerGift(String name, GiftImpl giftImpl) {
        this.name = name;
        this.giftImpl = giftImpl;
    }
}
public class RingGift extends GiftImpl {
    public RingGift() {}
    public RingGift(String name, GiftImpl giftImpl) {
        this.name = name;
        this.giftImpl = giftImpl;
    }
}


/**
* 测试方法
*/
public static void main(String[] args) {
    Gift warmGift = new WarmGift("warmGift", new FlowerGift("flower", null));
    System.out.println(warmGift); //name: warmGift giftImpl: class com.kly.bridge.FlowerGift
    
    Gift coldGift = new ColdGift("coldGift", new RingGift());
    System.out.println(coldGift); //name: coldGift giftImpl: class com.kly.bridge.RingGift
}
```
<br><br>


### 代理模式

* 代理类和被代理类实现同一个接口，代理类中聚合了被代理对象

`静态代理`<br>

```java
/**
* 代理接口
*/
public interface Movable {
    void move();
}


/**
* 被代理类
*/
public class Tank implements Movable {
    @Override
    public void move() {
        System.out.println("tank moving...");
    }
    public static void main(String[] args) {
        Movable m = new Tank();
        m.move();
    }
}


/**
* 代理类
*/
public class TankLogProxy implements Movable {
    private Movable tank;
    public TankLogProxy(Movable tank) {
        this.tank = tank;
    }
    @Override
    public void move() {
        System.out.println("tank start!");
        tank.move();
        System.out.println("tank end!");
    }
}


/**
* 代理类
*/
import java.util.Random;
public class TankTimeProxy implements Movable {
    private Movable tank;
    public TankTimeProxy(Movable tank) {
        this.tank = tank;
    }
    @Override
    public void move() {
        long start = System.currentTimeMillis();
        
        tank.move();
        
        try {
            Thread.sleep(new Random().nextInt(10000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis();
        System.out.println("运行时间：" + ((end-start)/1000.0) + " s");
    }
}


//静态代理测试方法
public static void main(String[] args) {
    TankTimeProxy tankTimeProxy = new TankTimeProxy(new Tank());
    TankLogProxy tankLogProxy = new TankLogProxy(tankTimeProxy);
    tankLogProxy.move();
}

//运行结果：
tank start!
tank moving...
运行时间：2.82 s
tank end!
```
<br>

`动态代理实现1`<br>

```java
public interface ArithmeticCaculator {
	int add(int i, int j);
	int minus(int i, int j);
	int multi(int i, int j);
	int div(int i, int j);
}

/**
 * 计算器功能实现类（保持其清纯性）
 * @author zhangqingli
 *
 */
public class ArithmeticCaculatorImpl implements ArithmeticCaculator {

	@Override
	public int add(int i, int j) {
		return i+j;
	}

	@Override
	public int minus(int i, int j) {
		return i-j;
	}

	@Override
	public int multi(int i, int j) {
		return i*j;
	}

	@Override
	public int div(int i, int j) {
		return i/j;
	}
}


import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;
/**
 * ArithmeticCaculator的java动态代理类
 * @author zhangqingli
 *
 */
public class MyArithmeticCaculatorJavaProxy {
	
	private ArithmeticCaculator target;
	public MyArithmeticCaculatorJavaProxy(ArithmeticCaculator target) {
		this.target = target;
	}

	/**
	 * 生成java动态代理对象
	 * @return
	 */
	public ArithmeticCaculator generateJavaProxy() {
		
		ClassLoader loader = this.getClass().getClassLoader();
		Class<?>[] interfaces = target.getClass().getInterfaces();
		InvocationHandler h = new InvocationHandler() {
			/**
			 * proxy: 正在返回的那个代理对象， 一般情况下invoke方法中都不使用该对象
			 * method: 正在被调用的方法
			 * args: 正在被调用方法的参数列表
			 */
			@Override
			public Object invoke(Object proxy, Method method, Object[] args)
					throws Throwable {
				Object result = null;
				try {
					System.out.println("before（前置通知）: " 
                                       + method.getName() + 
                                       " with args " + Arrays.asList(args));
					result = method.invoke(target, args);
					System.out.println("after returning（返回通知）: " 
                                       + method.getName() 
                                       + " with result " + result);
				} catch (Exception e) {
					System.out.println("after throwing（异常通知）: " 
                                       + method.getName()
                                       + " occurs exception " + e.toString());
					throw e;
				} finally {
					System.out.println("after（后置通知）: " 
                                       + method.getName() 
                                       + " ends");
				}
				return result;
			}
		};
		return (ArithmeticCaculator) Proxy.newProxyInstance(loader, interfaces, h);
	}
	
	
	/*
	 * 测试
	 */
	public static void main(String[] args) {
		ArithmeticCaculator caculator = 
          new MyArithmeticCaculatorJavaProxy(
          	new ArithmeticCaculatorImpl()).generateJavaProxy();
		//int result = caculator.add(1, 2);
		int result = caculator.div(1, 0);
		System.out.println(result);
	}
}
```

<br>

`动态代理实现2`<br>

```java
import java.lang.reflect.Method;
import java.util.Arrays;
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

/**
 * AirithmeticCaculator的cglib动态代理类
 * 在spring的AOP中，就是使用到java的动态代理和CGLib动态代理
 * 
 * java的动态代理是必须基于接口的
 * 而CGLib基于字节码创建代理类对象，不需要基于接口就可以直接创建代理类
 * 
 * @author zhangqingli
 *
 */
public class MyArithmeticCaculatorCglibProxy {
	
	/**
	 * 生成cglib动态代理类对象
	 * @param targetType
	 * @return
	 */
	public ArithmeticCaculator generateCglibProxy(Class<? extends ArithmeticCaculator> targetType) {
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(targetType);
		enhancer.setCallback(new MethodInterceptor() {
			/**
			 * obj: 动态生成的代理对象
			 * method: 实际调用的方法
			 * method: 调用方法入参列表
			 * proxy: method类的代理类，可以实现委托类对象的方法的调用，
			 *        常用invokeSuper方法，在拦截方法内可以调用多次
			 * 
			 */
			@Override
			public Object intercept(Object obj, Method method, Object[] args,
					MethodProxy methodProxy) throws Throwable {
				
				Object result = null;
				try {
					System.out.println("before（前置通知）: " 
                                       + method.getName() 
                                       + " with args " + Arrays.asList(args));
					result = methodProxy.invokeSuper(obj, args);
					System.out.println("after returning（返回通知）: " 
                                       + method.getName() 
                                       + " with result " + result);
				} catch (Exception e) {
					System.out.println("after throwing（异常通知）: " 
                                       + method.getName() 
                                       + " occurs exception " + e.toString());
					throw e;
				} finally {
					System.out.println("after（后置通知）: " 
                                       + method.getName() 
                                       + " ends");
				}
				return result;
			}
		});
		return (ArithmeticCaculator) enhancer.create();
	}
	
	/*
	 * main
	 */
	public static void main(String[] args) {
		ArithmeticCaculator caculator = new MyArithmeticCaculatorCglibProxy()
          	.generateCglibProxy(ArithmeticCaculatorImpl.class);
		//int result = caculator.add(1, 0);
		int result = caculator.div(1, 0);
		System.out.println(result);
	}
}
```

<br>



### 工厂模式

### 单例模式

### 装饰模式

<br>







