---
layout: post
title:  "java线程相关测试"
desc: "java线程相关测试"
keywords: "java线程相关测试,线程,java,kinglyjn"
date: 2012-11-17
categories: [Java]
tags: [java]
icon: fa-coffee
---

### 基本线程

```java
package threadtest;

/**
 * 基本线程
 * @author zhangqingli
 *
 */
public class BaseThread01 {
	public static void main(String[] args) {
		for (int i = 1; i < 10; i++) {
			Calculator01 c = new Calculator01(i);
			new Thread(c).start();
		}
	}
}

class Calculator01 implements Runnable {
	private int number;

	public Calculator01(int number) {
		this.number = number;
	}

	@Override
	public void run() {
		for (int i = 1; i < 10; i++) {
			System.out.println(Thread.currentThread().getName() + ": " + number + ": " + i+"*"+number+"="+i*number);
		}
	}
}	


//测试结果
Thread-0: 1: 1*1=1
Thread-4: 5: 1*5=5
Thread-5: 6: 1*6=6
Thread-3: 4: 1*4=4
Thread-6: 7: 1*7=7
Thread-6: 7: 2*7=14
...
...
Thread-8: 9: 9*9=81
...
Thread-1: 2: 8*2=16
Thread-0: 1: 9*1=9
Thread-1: 2: 9*2=18
```
<br><br>


### 获取和设置线程信息

```java
package threadtest;

import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.lang.Thread.State;

/**
 * 获取和设置线程信息
 * ID: 每个线程的独特标识。 
 * Name: 线程的名称。 
 * Priority: 线程对象的优先级。优先级别在1-10之间，1是最低级，10是最高级。不建议改变它们的优先级，但是你想的话也是可以的。 
 * Status: 线程的状态。在Java中，线程只能有这6种中的一种状态： new, runnable, blocked, waiting, time waiting, terminated.
 */
public class ThreadInfo02 {
	
	public static void main(String[] args) throws IOException {
		//
		Thread[] threads = new Thread[10];
		Thread.State status[] = new Thread.State[10];
		//
		for (int i=0; i<10; i++){
		   threads[i]=new Thread(new Calculator02(i));
		   if ((i%2)==0){
		      threads[i].setPriority(Thread.MAX_PRIORITY);
		   } else {
		      threads[i].setPriority(Thread.MIN_PRIORITY);
		   }
		   threads[i].setName("Thread "+i);
		}
		//
		PrintWriter pw = new PrintWriter(new FileWriter("/Users/zhangqingli/Desktop/log.txt"));
		for (int i=0; i<10; i++){
		   pw.println("Main : Status of Thread "+i+" : " +threads[i].getState());
		   pw.flush();
		   status[i]=threads[i].getState();
		}
		//
		for (int i = 0; i < 10; i++) {
			threads[i].start();
		}
		
		//
		boolean finish=false;
		while (!finish) {
		   for (int i=0; i<10; i++){
		      if (threads[i].getState()!=status[i]) {
		          writeThreadInfo(pw, threads[i], status[i]);
		          status[i]=threads[i].getState();
		      }
		   }
		   //
		   finish=true;
		   for (int i=0; i<10; i++){
		      finish=finish && (threads[i].getState()==State.TERMINATED);
		   }
		}
	}
	
	private static void writeThreadInfo(PrintWriter pw, Thread thread, Thread.State state) {
	   pw.printf("Main : Id %d - %s\n",thread.getId(),thread.getName());
	   pw.printf("Main : Priority: %d\n",thread.getPriority());
	   pw.printf("Main : Old State: %s\n",state);
	   pw.printf("Main : New State: %s\n",thread.getState());
	   pw.printf("Main : ************************************\n");
	   pw.flush();
	}
}

class Calculator02 implements Runnable {
	private int number;
	
	public Calculator02(int number) {
	   this.number=number;
	}
	
	@Override
	public void run() {
		for (int i=1; i<=10; i++){
			try {
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		   System.out.printf("%s: %d * %d = %d\n", Thread.currentThread().getName(), number, i, i*number);
		}
	}
}

//执行结果
Thread 1: 1 * 1 = 1
Thread 0: 0 * 1 = 0
Thread 3: 3 * 1 = 3
Thread 2: 2 * 1 = 2
Thread 4: 4 * 1 = 4
Thread 6: 6 * 1 = 6
Thread 7: 7 * 1 = 7
Thread 9: 9 * 1 = 9
Thread 5: 5 * 1 = 5
Thread 8: 8 * 1 = 8
...
Thread 4: 4 * 9 = 36
Thread 9: 9 * 9 = 81
Thread 3: 3 * 9 = 27
...


log.txt
Main : Status of Thread 0 : NEW
Main : Status of Thread 1 : NEW
Main : Status of Thread 2 : NEW
Main : Status of Thread 3 : NEW
Main : Status of Thread 4 : NEW
Main : Status of Thread 5 : NEW
Main : Status of Thread 6 : NEW
Main : Status of Thread 7 : NEW
Main : Status of Thread 8 : NEW
Main : Status of Thread 9 : NEW
Main : Id 8 - Thread 0
Main : Priority: 10
Main : Old State: NEW
Main : New State: TIMED_WAITING
Main : ************************************
Main : Id 9 - Thread 1
Main : Priority: 1
Main : Old State: NEW
Main : New State: TIMED_WAITING
Main : ************************************
...
Main : ************************************
Main : Id 15 - Thread 7
Main : Priority: 1
Main : Old State: TIMED_WAITING
Main : New State: RUNNABLE
Main : ************************************
Main : Id 16 - Thread 8
Main : Priority: 10
Main : Old State: RUNNABLE
Main : New State: TIMED_WAITING
Main : ************************************
Main : Id 9 - Thread 1
Main : Priority: 1
Main : Old State: BLOCKED
Main : New State: TIMED_WAITING
Main : ************************************
...
```
<br><br>


### 线程的中断

```java
package threadtest;
/**
 * 线程的中断
 * 1. java中提供 thread.interrupt() 和 thread.isInterrupted() 来控制线程的中断，不过可能会出现难以预料的结果，一般不用这种方法
 * 2. 线程的中断我们一般自己写程序判断和控制
 */
public class ThreadInterrupt03 {
	
	public static void main(String[] args) {
		PrimeGenerator03 pg = new PrimeGenerator03();
		Thread t = new Thread(pg);
		t.start();
		
		//等待5秒，然后中断PrimeGenerator线程
		try {
			Thread.sleep(5000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		pg.interrupted = true;
	}
}

class PrimeGenerator03 implements Runnable {
	volatile boolean interrupted = false;
	
	@Override
	public void run() {
		long num = 1L;
		while (true) {
			if (isPrime(num)) {
				System.out.printf("Num %d is Prime\n", num);
			}
			
			if (this.interrupted) {
				System.out.printf("The Prime Generator has been Interrupted\n");
				return;
			}
			num++;
			
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	private boolean isPrime(long num) { //判断一个数是否是质数
		if (num < 2) {
			return true;
		} 
		for (int i = 2; i < num; i++) {
			if (num%i==0) {
				return false;
			}
		}
		return true;
	}
	
}

//执行结果
Num 1 is Prime
Num 2 is Prime
Num 3 is Prime
Num 5 is Prime
Num 7 is Prime
Num 11 is Prime
The Prime Generator has been Interrupted
```
<br><br>


### 线程的睡眠与恢复

```java
package threadtest;

import java.util.Date;
import java.util.concurrent.TimeUnit;

/**
 * 线程的睡眠与恢复
 * @author zhangqingli
 *
 */
public class ThreadSleep04 {
	public static void main(String[] args) {
		new Thread(new FileClock04()).start();
	}
}

class FileClock04 implements Runnable {

	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(new Date());
			try {
				TimeUnit.SECONDS.sleep(1); //1s钟睡一次 等价于Thread.sleep
			} catch (InterruptedException e) {
				e.printStackTrace();
			} 
		}
	}
}

// 执行结果
Tue Mar 21 12:56:38 CST 2017
Tue Mar 21 12:56:39 CST 2017
Tue Mar 21 12:56:40 CST 2017
Tue Mar 21 12:56:41 CST 2017
Tue Mar 21 12:56:42 CST 2017
Tue Mar 21 12:56:43 CST 2017
Tue Mar 21 12:56:44 CST 2017
Tue Mar 21 12:56:45 CST 2017
Tue Mar 21 12:56:46 CST 2017
Tue Mar 21 12:56:47 CST 2017
```
<br><br>


### 线程的插入

```java
package threadtest;

import java.util.Date;
import java.util.concurrent.TimeUnit;

/**
 * 线程的插入 
 * thread.join(): 在当前线程中调用该方法时，当前线程会被暂停，直到调用join方法的线程执行完毕！
 * 
 */
public class ThreadJoin05 {
	//
	public static void main(String[] args) {
		Thread t1 = new Thread(new DataSourceLoader05());
		Thread t2 = new Thread(new NetworkConnectionsLoader());
		t1.start();
		t2.start();
		
		for (int i = 0; i < 10; i++) {
			System.out.println("["+ i +"] " + Thread.currentThread().getName());
			if (i==5) {
				try {
					t1.join();
					t2.join();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}

class DataSourceLoader05 implements Runnable {

	@Override
	public void run() {
		System.out.println("["+ new Date() +"] datasource loading...");
		try {
			TimeUnit.SECONDS.sleep(10); //资源加载10s
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("["+ new Date() +"] datasource loading has finished!");
	}
}


class NetworkConnectionsLoader implements Runnable {
	@Override
	public void run() {
		System.out.println("["+ new Date() +"] networkConnections loading...");
		try {
			TimeUnit.SECONDS.sleep(15); //资源加载15s
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("["+ new Date() +"] networkConnections loading has finished!");
	}
}


// 执行结果
[0] main
[Tue Mar 21 12:58:04 CST 2017] networkConnections loading...
[Tue Mar 21 12:58:04 CST 2017] datasource loading...
[1] main
[2] main
[3] main
[4] main
[5] main
[Tue Mar 21 12:58:14 CST 2017] datasource loading has finished!
[Tue Mar 21 12:58:19 CST 2017] networkConnections loading has finished!
[6] main
[7] main
[8] main
[9] main
```
<br><br>


### 守护线程的创建的运行

```java
package threadtest;
import java.util.ArrayDeque;
import java.util.Date;
import java.util.Deque;
import java.util.concurrent.TimeUnit;
/**
 * 守护线程的创建和运行
 * Java有一种特别的线程叫做守护线程。这种线程的优先级非常低，通常在程序里没有其他线程运行时才会执行它。
 * 当进程中不存在非守护线程了，则守护线程自动销毁。典型的守护线程就是垃圾回收线程，当进程中没有非守护
 * 线程了，则垃圾回收线程也就没有存在的必要了，自动销毁。用个比较通俗的比喻来解释一下守护线程，就是：
 * 任何一个守护线程都是整个JVM非守护线程的“保姆”。
 * 
 * 根据这些特点，守护线程通常用于在同一程序里给普通线程（也叫使用者线程）提供服务。
 * 它们通常无限循环的等待服务请求或执行线程任务。它们不能做重要的任务，因为我们不
 * 知道什么时候会被分配到CPU时间片，并且只要没有其他线程在运行，它们可能随时被终止。
 * JAVA中最典型的这种类型代表就是垃圾回收器。
 * 
 * 在这个测试中, 我们将学习如何创建一个守护线程，开发一个用2个线程的例子；
 * 我们的使用线程会写事件到queue, 守护线程会清除queue里10秒前创建的事件。
 */
public class ThreadDaemon06 {
	public static void main(String[] args) {
		//工作线程向双队列中写事件
		Deque<Event> deque = new ArrayDeque<Event>();
		for (int i = 0; i < 3; i++) {
			new Thread(new WriterTask(deque)).start();
		}
		//守护线程清除10s之前的时间
		CleanerTask cleanerTask = new CleanerTask(deque);
		cleanerTask.start();
	}
}


class Event {
	Date date;
	String event;

	public Event(Date date, String event) {
		this.date = date;
		this.event = event;
	}
}

class WriterTask implements Runnable {
	private Deque<Event> deque; //双向队列，可实现栈的数据结构
	
	public WriterTask(Deque<Event> deque) {
		this.deque = deque;
	}

	@Override
	public void run() {
		for (int i = 0; i < 100; i++) {
			Event event = new Event(new Date(), String.format("The thread %s has generated an event", Thread.currentThread().getId()));
			deque.addFirst(event);
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}

class CleanerTask extends Thread {
	private Deque<Event> deque;

	public CleanerTask(Deque<Event> deque) {
		this.deque = deque;
		//设置为守护线程
		//只能在start()方法之前可以调用setDaemon()方法。因为一旦线程运行了，就不能修改守护状态。
		super.setDaemon(true);
	}

	@Override
	public void run() {
		while (true) {
			Date date = new Date();
			clean(date);
		}
	}

	//它获取最后的事件，如果它在10秒前被创建，就删除它并查看下一个事件。
	//如果一个事件被删除，它会写一个事件信息和queue的新的大小，为了让你看到变化过程
	private void clean(Date date) {
		if (deque.size()==0) {
			return;
		}
		long time = 0;
		boolean deleteFlag = false;
		
		do {
			Event event = deque.getLast();
			time = date.getTime() - event.date.getTime();
			if (time > 10000) {
				System.out.printf("Cleaner: %s\n", event.event); 
				deque.removeLast();
				deleteFlag = true;
			}
		} while (time > 10000);
		
		if (deleteFlag) {
			System.out.printf("Cleaner: Size of the queue: %d\n", deque.size());
		}
	}
}


// 运行结果
...
Cleaner: Size of the queue: 19
Cleaner: The thread 10 has generated an event
Cleaner: The thread 8 has generated an event
Cleaner: The thread 9 has generated an event
Cleaner: Size of the queue: 19
Cleaner: The thread 9 has generated an event
Cleaner: The thread 8 has generated an event
Cleaner: Size of the queue: 18
Cleaner: The thread 8 has generated an event
Cleaner: The thread 10 has generated an event
Cleaner: Size of the queue: 19
Cleaner: The thread 10 has generated an event
Cleaner: The thread 8 has generated an event
Cleaner: Size of the queue: 19
Cleaner: The thread 9 has generated an event
Cleaner: The thread 8 has generated an event
Cleaner: Size of the queue: 18
Cleaner: The thread 10 has generated an event
Cleaner: Size of the queue: 19
...
```
<br><br>


### 在线程里处理运行时异常（未检查异常）

```java
package threadtest;
import java.lang.Thread.UncaughtExceptionHandler;
/**
 * 在线程里处理不受控制的异常
 * 
 * 当在一个线程里抛出一个异常，但是这个异常没有被捕获（这肯定是非检查异常），JVM 检查线程的相关方法是否有
 * 设置一个未捕捉异常的处理者 。如果有，JVM 使用Thread 对象和 Exception 作为参数调用此方法 。如果线程
 * 没有捕捉未捕获异常的处理者， 那么 JVM会把异常的 stack trace 写入操控台并结束任务。
 * 
 * Thread 类有其他相关方法可以处理未捕获的异常。静态方法 setDefaultUncaughtExceptionHandler() 
 * 为应用里的所有线程对象建立异常 handler 。当一个未捕捉的异常在线程里被抛出，JVM会寻找此异常的3种
 * 可能潜在的处理者（handler）。
 * 首先, 它寻找这个未捕捉的线程对象的异常handle，如我们在在这个指南中学习的。如果这个handle 不存在，
 * 那么JVM会在线程对象的ThreadGroup里寻找非捕捉异常的handler，如此例。如果此方法不存在，那么 JVM 
 * 会寻找默认非捕捉异常handle。如果没有一个handler存在, 那么 JVM会把异常的 stack trace 写入操控
 * 台并结束任务。
 * 
 */
public class TaskWithUncaughtException07 implements Runnable {

	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println("[" + i + "]");
			if (i==5) {
				Integer.parseInt("TTT"); //未检查异常（RuntimeException），这里没有将它捕获处理
			}
		}
	}
	
	public static void main(String[] args) {
		Thread t = new Thread(new TaskWithUncaughtException07());
		
		//为线程设置异常处理器
		t.setUncaughtExceptionHandler(new UncaughtExceptionHandler() {
			@Override
			public void uncaughtException(Thread t, Throwable e) {
				System.out.println("---------------异常信息-----------------");
				System.out.println("Thread: " + t.getId() + " " + t.getState());
				System.out.println("Exceprion: " + e.getClass().getName() + " " + e.getMessage());
				e.printStackTrace(System.out);
			}
		});
		
		t.start();
	}
}


// 运行结果：
[0]
[1]
[2]
[3]
[4]
[5]
---------------异常信息-----------------
Thread: 8 RUNNABLE
Exceprion: java.lang.NumberFormatException For input string: "TTT"
java.lang.NumberFormatException: For input string: "TTT"
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Integer.parseInt(Integer.java:492)
	at java.lang.Integer.parseInt(Integer.java:527)
	at threadtest.TaskWithUncaughtException07.run(TaskWithUncaughtException07.java:28)
	at java.lang.Thread.run(Thread.java:745)
```
<br><br>



### 使用本地线程变量

```java
package threadtest;
import java.util.Date;
import java.util.concurrent.TimeUnit;
/**
 * 使用本地线程变量
 *
 */
public class ThreadLocalTest08 {
	
	public static void main(String[] args) {
		//Task task = new UnsafeTask();
		Task task = new SafeTask();
		for (int i = 0; i < 10; i++) {
			new Thread(task).start();
			//
			try { 
				TimeUnit.SECONDS.sleep(2);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

		}
	}
	
}


abstract class Task implements Runnable {}

//此类startDate为非本地线程变量
//如果你创建一个类对象，实现Runnable接口，然后多个Thread对象使用同样的Runnable对象，全部的线程都共享同样的属性。
//这意味着，如果你在一个线程里改变一个属性，全部的线程都会受到这个改变的影响。
class UnsafeTask extends Task implements Runnable {
	private Date startDate;

	@Override
	public void run() {
		startDate = new Date();
		System.out.printf("Starting Thread: %s : %s\n",Thread. currentThread().getId(),startDate);
		
		try {
			TimeUnit.SECONDS.sleep( (int)Math.rint(Math.random()*10));
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.printf("Thread Finished: %s : %s\n",Thread. currentThread().getId(),startDate);
	}
}

//此类startDate为本地线程变量
//有时，你希望程序里的各个线程的属性不会被共享。 Java 并发 API提供了一个很清楚的机制叫本地线程变量。
//本地线程类还提供 remove() 方法，删除存储在线程本地变量里的值。
//另外，Java 并发 API 包括 InheritableThreadLocal 类提供线程创建线程的值的遗传性 。
//如果线程A有一个本地线程变量，然后它创建了另一个线程B，那么线程B将有与A相同的本地线程变量值。 
//你可以覆盖 childValue() 方法来初始子线程的本地线程变量的值。 它接收父线程的本地线程变量作为参数。
class SafeTask extends Task implements Runnable {
	private ThreadLocal<Date> startDate = new ThreadLocal<Date>() {
		@Override
		protected Date initialValue() {
			return new Date();
		}
	};
	
	@Override
	public void run() {
		System.out.printf("Starting Thread: %s : %s\n",Thread. currentThread().getId(),startDate.get());
		
		try {
			TimeUnit.SECONDS.sleep( (int)Math.rint(Math.random()*10));
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.printf("Thread Finished: %s : %s\n",Thread. currentThread().getId(),startDate.get());
	}
}


//使用UnsafeTask执行结果
Starting Thread: 8 : Tue Mar 21 14:30:31 CST 2017
Starting Thread: 9 : Tue Mar 21 14:30:33 CST 2017
Starting Thread: 10 : Tue Mar 21 14:30:35 CST 2017
Thread Finished: 8 : Tue Mar 21 14:30:35 CST 2017
Starting Thread: 11 : Tue Mar 21 14:30:37 CST 2017
Thread Finished: 9 : Tue Mar 21 14:30:37 CST 2017
Thread Finished: 10 : Tue Mar 21 14:30:37 CST 2017
Starting Thread: 12 : Tue Mar 21 14:30:39 CST 2017
Thread Finished: 11 : Tue Mar 21 14:30:39 CST 2017
Starting Thread: 13 : Tue Mar 21 14:30:41 CST 2017
Starting Thread: 14 : Tue Mar 21 14:30:43 CST 2017
Thread Finished: 13 : Tue Mar 21 14:30:43 CST 2017
Starting Thread: 15 : Tue Mar 21 14:30:45 CST 2017
Thread Finished: 14 : Tue Mar 21 14:30:45 CST 2017
Starting Thread: 16 : Tue Mar 21 14:30:47 CST 2017
Thread Finished: 12 : Tue Mar 21 14:30:47 CST 2017
Starting Thread: 17 : Tue Mar 21 14:30:49 CST 2017
Thread Finished: 17 : Tue Mar 21 14:30:49 CST 2017
Thread Finished: 15 : Tue Mar 21 14:30:49 CST 2017
Thread Finished: 16 : Tue Mar 21 14:30:49 CST 2017

//使用SafeTask执行结果
Starting Thread: 8 : Tue Mar 21 15:11:10 CST 2017
Starting Thread: 9 : Tue Mar 21 15:11:12 CST 2017
Thread Finished: 9 : Tue Mar 21 15:11:12 CST 2017
Starting Thread: 10 : Tue Mar 21 15:11:14 CST 2017
Starting Thread: 11 : Tue Mar 21 15:11:16 CST 2017
Thread Finished: 11 : Tue Mar 21 15:11:16 CST 2017
Starting Thread: 12 : Tue Mar 21 15:11:18 CST 2017
Thread Finished: 10 : Tue Mar 21 15:11:14 CST 2017
Thread Finished: 12 : Tue Mar 21 15:11:18 CST 2017
Thread Finished: 8 : Tue Mar 21 15:11:10 CST 2017
Starting Thread: 13 : Tue Mar 21 15:11:20 CST 2017
Starting Thread: 14 : Tue Mar 21 15:11:22 CST 2017
Thread Finished: 13 : Tue Mar 21 15:11:20 CST 2017
Starting Thread: 15 : Tue Mar 21 15:11:24 CST 2017
Starting Thread: 16 : Tue Mar 21 15:11:26 CST 2017
Thread Finished: 15 : Tue Mar 21 15:11:24 CST 2017
Starting Thread: 17 : Tue Mar 21 15:11:28 CST 2017
Thread Finished: 14 : Tue Mar 21 15:11:22 CST 2017
Thread Finished: 17 : Tue Mar 21 15:11:28 CST 2017
Thread Finished: 16 : Tue Mar 21 15:11:26 CST 2017
```
<br><br>


### 线程工厂

```java
package threadtest;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.TimeUnit;
/**
 * 线程工厂
 * @author zhangqingli
 *
 */
public class MyThreadFactory09 implements ThreadFactory {
	private String name;
	private int threadNum;
	private List<String> status;
	
	public MyThreadFactory09(String name) {
		this.name = name;
		this.threadNum = 0;
		status = new ArrayList<String>();
	}

	@Override
	public Thread newThread(Runnable r) {
		Thread t = new Thread(r, name+"-Thread-"+threadNum);
		threadNum++;
		status.add(String.format("created thread %d with name %s on %s\n",
				t.getId(),t.getName(),new Date()));
		return t;
	}
	
	public String getStatus() {
		return status.toString();
	}
	
	
	public static void main(String[] args) {
		MyThreadFactory09 tf = new MyThreadFactory09("MyThreadFactory");
		MyTask task = new MyTask();
		System.out.println("Starting the Threads\n");
		for (int i = 0; i < 10; i++) {
			tf.newThread(task).start();
		}
		
		System.out.printf("Factory stats:\n"); 
		System.out.printf("%s\n",tf.getStatus());
	}
}


class MyTask implements Runnable {
	@Override
	public void run() {
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
}


//执行结果
Starting the Threads

Factory stats:
[created thread 8 with name MyThreadFactory-Thread-0 on Tue Mar 21 15:16:41 CST 2017
, created thread 9 with name MyThreadFactory-Thread-1 on Tue Mar 21 15:16:41 CST 2017
, created thread 10 with name MyThreadFactory-Thread-2 on Tue Mar 21 15:16:41 CST 2017
, created thread 11 with name MyThreadFactory-Thread-3 on Tue Mar 21 15:16:41 CST 2017
, created thread 12 with name MyThreadFactory-Thread-4 on Tue Mar 21 15:16:41 CST 2017
, created thread 13 with name MyThreadFactory-Thread-5 on Tue Mar 21 15:16:41 CST 2017
, created thread 14 with name MyThreadFactory-Thread-6 on Tue Mar 21 15:16:41 CST 2017
, created thread 15 with name MyThreadFactory-Thread-7 on Tue Mar 21 15:16:41 CST 2017
, created thread 16 with name MyThreadFactory-Thread-8 on Tue Mar 21 15:16:41 CST 2017
, created thread 17 with name MyThreadFactory-Thread-9 on Tue Mar 21 15:16:41 CST 2017
]
```
<br><br>


### 安全共享数居（安全同步数据）

```java
package threadtest;
import java.util.Random;
import java.util.concurrent.TimeUnit;
/**
 * 安全共享数居（安全同步数据）
 * 只有操作公共资源的原子操作，才是需要加锁的，如果不是就没有加锁的必要。
 */
public class SynchronizedTest10 implements Runnable {
	private int count = 10;
	
	@Override
	public void run() {
		while (count>0) {
			//一个原子操作
			synchronized (SynchronizedTest10.class) {
				System.out.println(Thread.currentThread().getName() + ": " + count);
				count--;
			}
			
			try {
				TimeUnit.SECONDS.sleep(new Random().nextInt(3));
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		SynchronizedTest10 s = new SynchronizedTest10();
		
		for (int i = 0; i < 3; i++) {
			new Thread(s).start();
		}
	}
}


//加锁执行结果：
Thread-0: 10
Thread-2: 9
Thread-1: 8
Thread-1: 7
Thread-1: 6
Thread-0: 5
Thread-2: 4
Thread-1: 3
Thread-2: 2
Thread-0: 1

//不加锁执行结果：
Thread-0: 10
Thread-2: 10
Thread-1: 10
Thread-0: 7
Thread-0: 6
Thread-2: 5
Thread-1: 5
Thread-1: 3
Thread-0: 2
Thread-2: 1
```
<br><br>



### 多个对象多个锁

```java 
package threadtest;
/**
 * 多个对象多个锁
 */
public class SynchronizedTest11 {
	public static void main(String[] args) {
		/*
		LoginServlet loginServlet = new LoginServlet();
		new Thread(new LoginTask(loginServlet, "a")).start();
		new Thread(new LoginTask(loginServlet, "b")).start();
		//执行结果
		login success!
		username: a
		login fail!
		username: b
		*/
		
		LoginServlet loginServlet01 = new LoginServlet();
		LoginServlet loginServlet02 = new LoginServlet();
		new Thread(new LoginTask(loginServlet01, "a")).start();
		new Thread(new LoginTask(loginServlet02, "b")).start();
		//执行结果
		//login success!
		//login fail!
		//username: b
		//username: a
		//[分析]
		//两个线程分别访问同一个类的两个不同实例的相同名称的同步方法，效果是以异步方式执行的。
		//本类由于是创建了两个业务对象，在系统中产生了两个锁，所以运行的结果是异步的。
		//[注]
		//如果把loginServlet的login方法改成synchronized static类型的，那么这个方法的锁
		//类型就是类对象LoginServlet.class，由于它在JVM中只有一份，因此在该例中执行的效果
		//仍是同步的。
	}
}


class LoginTask implements Runnable {
	private LoginServlet loginServlet;
	private String username;
	
	public LoginTask(LoginServlet loginServlet, String username) {
		this.loginServlet = loginServlet;
		this.username = username;
	}

	@Override
	public void run() {
		loginServlet.login(username);
	}
}

class LoginServlet {
	public synchronized void login(String username) {
		try {
			if ("a".equals(username)) {
				System.out.println("login success!");
				Thread.sleep(2000);
			} else {
				System.out.println("login fail!");
			}
			System.out.println("username: " + username);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
<br><br>



### 可重入锁

```java
package threadtest;
import java.util.concurrent.TimeUnit;
/**
 * 可重入锁
 * 可重入锁的概念是：自己可以再次获取自己的内部锁，比如线程A获得了某个对象锁，此时
 * 它的这个对象锁还没有释放，当它再次想获取这个对象锁的时候是可以获取的。
 * 
 * 父子可重入锁
 * 当存在父子类继承关系时，子类完全可以通过可重入锁调用父类的同步方法。
 * 下面的就是一个父子可重入锁的演示。
 *
 */
public class SynchronizedTest12 {
	public static void main(String[] args) {
		B b = new B();
		new Thread(new MyThread(b)).start();
		new Thread(new MyThread(b)).start();
		//执行结果
		//Thread: 8: b.bbb
		//Thread: 8: a.aaa
		//Thread: 9: b.bbb
		//Thread: 9: a.aaa
	}
}

class A {
	public synchronized void aaa() {
		System.out.println("Thread: " + Thread.currentThread().getId() + ": a.aaa");
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

class B extends A {
	public synchronized void bbb() {
		System.out.println("Thread: " + Thread.currentThread().getId() + ": b.bbb");
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		super.aaa();
	}
}

class MyThread implements Runnable {
	private B b;
	
	public MyThread(B b) {
		this.b = b;
	}

	@Override
	public void run() {
		b.bbb();
	}
}
```
<br><br>


### volatile关键字

```java
package threadtest;
import java.util.concurrent.TimeUnit;
/**
 * 1. volatile关键字解决的是数据在多个线程之间的可见性（不保证被操作数据的原子一致性），
 *    而synchronized关键字解决的是操作的原子性，而且synchronized除了保证数据操作的
 *    原子性，也将线程工作内存中的私有变量与主内存中的公共变量同步。
 * 
 * 2. 主内存（=物理内存），线程工作内存（=CPU高速缓存）
 * 	  被volatile修饰的变量保证了数据在多个不同线程之间的同步性，原理如下：
 * 	  volatile修饰符使被它修饰的变量对应的CPU高速缓存区（L1、L2）失效，
 * 	  当volatile变量在线程A中被修改后，线程A会将其工作内存中的被修改
 *    后的volatile变量写入主内存中，然后发送信号通知其他线程从主内存中
 *    同步最新的值到工作内存，这样就保证了线程间变量数据的可见性。
 *    
 */
public class VolatileTest13 {
	public static void main(String[] args) {
		MyTask02 task = new MyTask02();
		for (int i = 0; i < 10; i++) {
			new Thread(task).start();
		}
		//
		try {
			TimeUnit.SECONDS.sleep(5);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		//
		task.setInterrupted(true);
	}
}

class MyTask02 implements Runnable {
	private volatile boolean isInterrupted = false;
	
	public boolean isInterrupted() {
		return isInterrupted;
	}
	public void setInterrupted(boolean isInterrupted) {
		this.isInterrupted = isInterrupted;
	}

	@Override
	public void run() {
		while (true) {
			System.out.println("Thread: " + Thread.currentThread().getId() + " running!");
			if (isInterrupted) {
				break;
			}
		}
		System.out.println("Thread: " + Thread.currentThread().getId() + " end!");
	}
}
```
<br><br>


### 


```java
package threadtest;
/**
 * 验证volatile不具有操作原子性
 *
 */
public class VolatileTest14 {
	
	public static void main(String[] args) {
	
		MyThread03 thread03 = new MyThread03();
		
		for (int i = 0; i < 10; i++) {
			new Thread(thread03).start();
		}
	}
}

class MyThread03 implements Runnable {
	public volatile int count = 0;

	public synchronized void incr() {
		try {
			count++;
			System.out.println(Thread.currentThread().getId() + ": " + count);
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	@Override
	public void run() {
		incr();
	}
}


//加锁执行结果：
8: 1
17: 2
16: 3
15: 4
14: 5
13: 6
12: 7
11: 8
10: 9
9: 10

//不加锁执行结果：
9: 2
12: 5
13: 6
11: 4
10: 3
8: 2
14: 7
15: 8
16: 9
17: 10
```
<br><br>


