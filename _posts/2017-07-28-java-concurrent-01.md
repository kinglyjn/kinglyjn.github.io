---
layout: post
title:  "JAVA并发编程的挑战"
desc: "JAVA并发编程的挑战"
keywords: "JAVA并发编程的挑战,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



> *摘要：* 第1章并发编程的挑战 并发编程的目的是为了让程序运行的更快，但是并不是启动更多的线程，就能让程序最大限度的并发执行。在进行并发编程时，如果希望通过多线程执行任务让程序运行的更快，会面临非常多的挑战，比如上下文切换的问题，死锁的问题，以及受限于硬件和软件的资源限制问题，本章会介绍几种并发编程的挑战，以及解决方案。

<br>



### 1.1     上下文切换

即使是单核处理器也支持多线程执行代码，CPU通过给每个线程分配CPU时间片来实现这个机制。时间片是CPU分配给各个线程的时间，因为时间片非常短，所以CPU通过不停的切换线程执行，让我们感觉多个线程是同时执行的，时间片一般是几十毫秒（ms）。

CPU通过时间片分配算法来循环执行任务，当前任务执行一个时间片后会切换到下个任务，但是在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。所以任务的保存到再加载的过程就是**一次上下文切换**。

就像我们同时在读两本书，比如当我们在读一本英文的技术书时，发现某个单词不认识，于是便打开中英文字典，但是在放下英文技术书之前，大脑必需首先记住这本书读到了多少页的第多少行，等查完单词之后，能够继续读这本书，这样的切换是会影响读书效率的，同样上下文切换也会影响到多线程的执行速度。



### 1.1.1    多线程一定快吗？

下面的代码演示串行和并发执行累加操作的时间，请思考下面的代码并发执行一定比串行执行快些吗？

```java
public class ConcurrencyTest {
	/** 执行次数 */
	private static final long count = 40000l;

	public static void main(String[] args) throws InterruptedException {
		// 并发计算
		concurrency();
		// 单线程计算
		serial();
	}

	private static void concurrency() throws InterruptedException {
		long start = System.currentTimeMillis();
		Thread thread = new Thread(new Runnable() {
			@Override
			public void run() {
				int a = 0;
				for (long i = 0; i < count; i++) {
					a += 5;
				}
				System.out.println(a);
			}
		});
		thread.start();
		int b = 0;
		for (long i = 0; i < count; i++) {
			b--;
		}
		long time = System.currentTimeMillis() - start;
		thread.join();
		System.out.println("concurrency: " + time + "ms,b=" + b);
	}

	private static void serial() {
		long start = System.currentTimeMillis();
		int a = 0;
		for (long i = 0; i < count; i++) {
			a += 5;
		}
		int b = 0;
		for (long i = 0; i < count; i++) {
			b--;
		}
		long time = System.currentTimeMillis() - start;
		System.out.println("serial:" + time + "ms,b=" + b + ",a=" + a);

	}
}


//测试结果
循环次数	串行执行耗时（单位ms）	并发执行耗时	并发比串行快多少
1亿			130						77			约1倍
1千万			18						9		   约1倍
1百万			5						5		   差不多
10万			 4						 3			慢
1万			 0						 1			慢
```

从试验中我们可以发现当并发执行累加操作不超过百万次时，速度会比串行执行累加操作要慢。那么为什么并发执行的速度还比串行慢呢？因为线程有创建和上下文切换的开销。<br>



### 测试上下文切换次数和时长

下面我们来看看有什么工具可以度量上下文切换带来的消耗。

* 使用Lmbench3可以测量上下文切换的时长。
* 使用vmstat可以测量上下文切换的次数

下面是利用vmstat测量上下文切换次数的示例。

```shell
root@ubuntu:~# vmstat 2 1
procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 1  0      0 3499840 315836 3819660    0    0     0     1    2    0  0  0 100  0
 0  0      0 3499584 315836 3819660    0    0     0     0   88  158  0  0 100  0
 0  0      0 3499708 315836 3819660    0    0     0     2   86  162  0  0 100  0
 0  0      0 3499708 315836 3819660    0    0     0    10   81  151  0  0 100  0
 1  0      0 3499732 315836 3819660    0    0     0     2   83  154  0  0 100  0
```

CS（Content Switch）表示上下文切换的次数，从上面的测试结果中，我们可以看到其中上下文的每一秒钟切换1000多次。2表示每个两秒采集一次服务器状态，1表示只采集一次。

<br>



### 1.1.3    如何减少上下文切换

减少上下文切换的方法有无锁并发编程、CAS算法、单线程编程和使用协程。

- 无锁并发编程。多线程竞争锁时，会引起上下文切换，所以多线程处理数据时，可以用一些办法来避免使用锁，如将数据用ID进行Hash算法后分段，不同的线程处理不同段的数据。
- CAS算法。Java的Atomic包使用CAS算法来更新数据，而不需要加锁。
- 使用最少线程。避免创建不需要的线程，比如任务很少，但是创建了很多线程来处理，这样会造成大量线程都处于等待状态。
- 协程：在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换。

<br>



### 减少上下文切换实战

本节描述通过减少线上大量WAITING的线程，来减少上下文切换次数。

```shell
# 第一步，使用jstack命令看看pid为22924的进程里的线程都在做什么
> sudo -u ubuntu jstack  22924  > ~/test/dump22924

# 第二步，统计所有线程分别处于什么状态
> grep java.lang.Thread.State ~/test/dump22924 | awk '{print $2$3$4$5}' | sort | uniq -c
39 RUNNABLE
21 TIMED_WAITING(onobjectmonitor)
6 TIMED_WAITING(parking)
51 TIMED_WAITING(sleeping)
305 WAITING(onobjectmonitor)
3 WAITING(parking)

# 第三步，打开dump22924文件查看 WAITING(onobjectmonitor) 的线程在做什么。发现这些线程基本全是jboss的工作线程，在await。说明jboss线程池里接收到的任务太少，大量线程都空闲着。
"http-0.0.0.0-7001-97" daemon prio=10 tid=0x000000004f6a8000 nid=0x555e in Object.wait() [0x0000000052423000]
 java.lang.Thread.State: WAITING (on object monitor)
 at java.lang.Object.wait(Native Method)
 - waiting on <0x00000007969b2280> (a org.apache.tomcat.util.net.AprEndpoint$Worker)
 at java.lang.Object.wait(Object.java:485)
 at org.apache.tomcat.util.net.AprEndpoint$Worker.await(AprEndpoint.java:1464)
 - locked <0x00000007969b2280> (a org.apache.tomcat.util.net.AprEndpoint$Worker)
 at org.apache.tomcat.util.net.AprEndpoint$Worker.run(AprEndpoint.java:1489)
 at java.lang.Thread.run(Thread.java:662)


# 第四步，减少jboss的工作线程数，找到jboss线程池的配置信息，将maxThreads降低到100
<maxThreads="250" maxHttpHeaderSize="8192"
emptySessionPath="false" minSpareThreads="40" maxSpareThreads="75" maxPostSize="512000" protocol="HTTP/1.1"
enableLookups="false" redirectPort="8443" acceptCount="200" bufferSize="16384"
connectionTimeout="15000" disableUploadTimeout="false" useBodyEncodingForURI="true">


# 第五步，重启jboss，再dump线程信息，然后统计 waiting(onobjectmonitor) 的线程，发现减少了175个，waiting的线程减少了，系统切换上下文的次数就会少，因为每一次从waiting到runnable都会进行一次上下文的切换。
> grep java.lang.Thread.State ~/test/dump22924 | awk '{print $2$3$4$5}' | sort | uniq -c
44 RUNNABLE
22 TIMED_WAITING(onobjectmonitor)
9 TIMED_WAITING(parking)
36 TIMED_WAITING(sleeping)
130 WAITING(onobjectmonitor)
1 WAITING(parking)
```

<br>



### 死锁

锁是个非常有用的工具，运用场景非常多，因为其使用起来非常简单，而且易于理解。但同时它也会带来一些困扰，那就是可能会引起死锁，一旦产生死锁，会造成系统功能不可用。让我们先来看一段代码，这段代码会引起死锁，线程t1和t2互相等待对方释放锁。

```java
public class DeadLockDemo {
	/** A锁 */
	private static String A = "A";

	/** B锁 */
	private static String B = "B";

	public static void main(String[] args) {
		new DeadLockDemo().deadLock();
	}

	private void deadLock() {
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				synchronized (A) {
					try {
						Thread.sleep(2000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					synchronized (B) {
						System.out.println("1");
					}
				}
			}
		});

		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				synchronized (B) {
					synchronized (A) {
						System.out.println("2");
					}
				}
			}
		});
		t1.start();
		t2.start();
	}
}
```

这段代码只是演示死锁的场景，在现实中你可能很难会写出这样的代码。但是一些更为复杂的场景中你可能会遇到这样的问题，比如t1拿到锁之后，因为一些异常情况没有释放锁，比如死循环。又或者是t1拿到一个数据库锁，释放锁的时候抛了异常，没释放掉。

一旦出现死锁，业务是可感知的，因为不能继续提供服务了，那么只能通过dump线程看看到底是哪个线程出现了问题，以下线程信息告诉我们是DeadLockDemo类的42行和31号引起的死锁：

```java
> jps
88230 DeadLockDemo
71787 
88236 Jps

> jstack 88230
"Thread-1" #11 prio=5 os_prio=31 tid=0x00007fc230867800 nid=0x5903 waiting for monitor entry [0x000070000c152000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at DeadLockDemo$2.run(DeadLockDemo.java:36)
	- waiting to lock <0x00000007aab729f0> (a java.lang.String)
	- locked <0x00000007aab72a20> (a java.lang.String)
	at java.lang.Thread.run(Thread.java:748)

"Thread-0" #10 prio=5 os_prio=31 tid=0x00007fc230866800 nid=0x5703 waiting for monitor entry [0x000070000c04f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at DeadLockDemo$1.run(DeadLockDemo.java:25)
	- waiting to lock <0x00000007aab72a20> (a java.lang.String)
	- locked <0x00000007aab729f0> (a java.lang.String)
	at java.lang.Thread.run(Thread.java:748)
     

Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007fc2318268b8 (object 0x00000007aab729f0, a java.lang.String),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007fc231827d58 (object 0x00000007aab72a20, a java.lang.String),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
	at DeadLockDemo$2.run(DeadLockDemo.java:36)
	- waiting to lock <0x00000007aab729f0> (a java.lang.String)
	- locked <0x00000007aab72a20> (a java.lang.String)
	at java.lang.Thread.run(Thread.java:748)
"Thread-0":
	at DeadLockDemo$1.run(DeadLockDemo.java:25)
	- waiting to lock <0x00000007aab72a20> (a java.lang.String)
	- locked <0x00000007aab729f0> (a java.lang.String)
	at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

现在我们介绍下如何避免死锁的几个常见方法:

- 避免一个线程同时获取多个锁。
- 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源。
- 尝试使用定时锁，使用tryLock(timeout)来替代使用内部锁机制。
- 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败。

<br>



### 1.3 资源限制的挑战

##### （1）什么是资源限制？

资源限制是指在进行并发编程时，程序的执行速度受限于计算机硬件资源或软件资源的限制。比如服务器的带宽只有2M，某个资源的下载速度是1M每秒，系统启动十个线程下载资源，下载速度不会变成10M每秒，所以在进行并发编程时，要考虑到这些资源的限制。硬件资源限制有带宽的上传下载速度，硬盘读写速度和CPU的处理速度。软件资源限制有数据库的连接数和Sorket连接数等。<br>

##### （2）资源限制引发的问题

并发编程将代码执行速度加速的原则是将代码中串行执行的部分变成并发执行，但是如果某段串行的代码并发执行，但是因为受限于资源的限制，仍然在串行执行，这时候程序不仅不会执行加快，反而会更慢，因为增加了上下文切换和资源调度的时间。例如，之前看到一段程序使用多线程在办公网并发的下载和处理数据时，导致CPU利用率100％，任务几个小时都不能运行完成，后来修改成单线程，一个小时就执行完成了。<br>

##### （3）如何解决资源限制的问题？

对于硬件资源限制，可以考虑使用集群并行执行程序，既然单机的资源有限制，那么就让程序在多机上运行，比如使用ODPS，hadoop或者自己搭建服务器集群，不同的机器处理不同的数据，比如将数据ID％机器数，得到一个机器编号，然后由对应编号的机器处理这笔数据。对于软件资源限制，可以考虑使用资源池将资源复用，比如使用连接池将数据库和Sorket连接复用，或者调用对方webservice接口获取数据时，只建立一个连接。

##### （4）在资源限制情况下进行并发编程

那么如何在资源限制的情况下，让程序执行的更快呢？根据不同的资源限制调整程序的并发度，比如下载文件程序依赖于两个资源，带宽和硬盘读写速度。有数据库操作时，要数据库连接数，如果SQL语句执行非常快，而线程的数量比数据库连接数大很多，则某些线程会被阻塞住，等待数据库连接。

