---
layout: post
title:  "JAVA并发编程基础"
desc: "JAVA并发编程基础"
keywords: "JAVA并发编程基础,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



### 使用JMX程序代码检查程序中包含的线程

```java
import java.lang.management.ManagementFactory;
import java.lang.management.ThreadInfo;
import java.lang.management.ThreadMXBean;

/**
* 也可以使用 jstack 或 jconsole 工具
*/
public class MultiThreadTest {
	public static void main(String[] args) {
		//获取java线程管理器
		ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
		
		//不需要获取同步的monitor和synchronizer信息，仅获取线程和新城堆栈信息
		ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
		for (ThreadInfo threadInfo : threadInfos) {
			/*
			System.out.println(threadInfo.getLockOwnerName());
			System.out.println(threadInfo.getLockOwnerId());
			System.out.println(threadInfo.getThreadName());
			System.out.println(threadInfo.getThreadState());
			System.out.println(threadInfo.getBlockedCount());
			System.out.println(threadInfo.getBlockedTime());
			System.out.println(threadInfo.getWaitedCount());
			System.out.println(threadInfo.getWaitedTime());
			System.out.println(threadInfo.getLockedMonitors());
			System.out.println(threadInfo.getLockedSynchronizers());
			System.out.println(threadInfo.getLockInfo());*/
			System.out.println(threadInfo.getThreadName());
		}
	}
}
```

可见，一个java程序的运行不仅仅是main方法的运行，而是main方法和其他线程的同时执行。



### 为什么要使用多线程

* 更多的处理器核心
  * 线程是大多数系统调度的基本单元，一个程序作为一个进程来运行，程序运行过程中能够创建多个线程，而一个线程在一个时刻只能运行在一个处理核心上。试想一下，一个单线程的程序在运行的时候只能使用一个处理器核心，那么再多的处理核心加入也无法显著提升改程序的执行效率。相反，如果改程序使用多线程技术，将计算逻辑分配到多个处理器核心上，就会显著提高程序的运行速度。
* 更快的响应时间
  * 如订单的创建，它包括插入订单数据、生成订单快照、发送邮件通知卖家、记录货品销售数量等。在这种场景中，我们会使用多线程技术，将数据一致性不强的操作派发给其他线程来处理（也可以使用消息队列），如生成订单快照、发送邮件等。
* 更好的编程模型
  * 一旦开发人员建立好了模型，稍作修改，总能方便地映射到java提供的多线程模型上。


<br>




### 线程的优先级

现代的计算机基本采用时分的形式调度运行的线程，操作系统会分出一个个时间片，线程会分配到若干时间片，当线程的时间片用完了就会发生线程调度，并等待下次分配。线程分配到多少时间片也就决定了线程能够使用多少系统资源，而线程的优先级就是决定线程分配多或者分配少一些系统资源的属性。

在java中，通过一个int型的成员变量 Thread.priority 来控制优先级，优先级的范围是 1~10，默认是5。优先级高的线程分配的时间片数量要多于优先级高的线程。设置线程优先级时，针对频繁阻塞（休眠或者io操作）的线程需要设置较高的优先级，而偏重于计算（需要较多cpu时间或者偏运算）的线程则设置较低的优先级，确保cpu不被独占。在不同的jvm及操作系统上，线程的规划会存在差异，有些操作系统甚至会会略对线程优先级的设定（如mac、ubuntu14.04等）。

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

public class PriorityTest {
	private static volatile boolean start = false;
	private static volatile boolean end = false;
	
	public static void main(String[] args) throws InterruptedException {
		List<Job> jobs = new ArrayList<>();
		for (int i = 0; i < 10; i++) {
			int priority = i < 5 ? Thread.MIN_PRIORITY : Thread.MAX_PRIORITY;
			Job job = new Job(priority);
			jobs.add(job);
			Thread thread = new Thread(job, "Thread" + i);
			thread.setPriority(priority);
			thread.start();
		}
		
		start = true;
		TimeUnit.SECONDS.sleep(10);
		end = true;
		for (Job job : jobs) {
			System.out.println("Job Priority: " + job.priority + ", Count: " + job.jonCount);
		}
	}
	
	static class Job implements Runnable {
		private int priority;
		private long jonCount;
		
		public Job(int priority) {
			this.priority = priority;
		}

		@Override
		public void run() {
			while (!start) {
				Thread.yield();
			}
			while (!end) {
				Thread.yield();
				jonCount++;
			}
		}
	}
}

//运行结果：
Job Priority: 1, Count: 1264681
Job Priority: 1, Count: 1268516
Job Priority: 1, Count: 1259088
Job Priority: 1, Count: 1263380
Job Priority: 1, Count: 1261560
Job Priority: 10, Count: 1253439
Job Priority: 10, Count: 1245494
Job Priority: 10, Count: 1261947
Job Priority: 10, Count: 1260787
Job Priority: 10, Count: 1257460
```

从输出结果可以看到，线程优先级并没有生效，线程优先级为1的线程和线程优先级为10的线程技术的结果非常接近。这表示程序的正确性不能依赖于线程优先级的高低。

<br>



### 某一个时刻下线程的状态

* NEW：初始化状态，线程被构建，但是还没有调用start方法
* RUNNABLE：运行状态，java线程将操作系统中的就绪和运行两种状态统称为“运行中”
* BLOCK：阻塞状态，表示线程阻塞于锁
* WAITING：等待状态，表示线程进入等待状态，进入等待状态的线程需要等待其他线程做出一些特定的动作
* TIME_WAITING：超时等待状态，该状态不同于WAITING，他是可以在指定时间内自行返回的
* TERMINATED：终止状态，表示当前线程已执行完毕

```java
import java.util.concurrent.TimeUnit;
public class ThreadStateTest {
	public static void main(String[] args) {
		//睡眠线程（该线程不断进行睡眠）
		new Thread(new Runnable() {
			@Override
			public void run() {
				while (true) {
					sleep(100);
				}
			}
		}, "TimeWaitingThread").start();
		
		//等待线程（该线程在A.class实例上等待）
		new Thread(new Runnable() {
			@Override
			public void run() {
				while (true) {
					synchronized (A.class) {
						try {
							A.class.wait();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
			}
		}, "WaitingThread").start();
		
		//使用两个锁线程，一个获取锁成功，另一个被阻塞
		new Thread(new Runnable() {
			@Override
			public void run() {
				synchronized (B.class) {
					while (true) {
						sleep(100);
					}
				}
			}
		}, "BlockedThread-1").start();
		new Thread(new Runnable() {
			@Override
			public void run() {
				synchronized (B.class) {
					while (true) {
						sleep(100);
					}
				}
			}
		}, "BlockedThread-2").start();
	}
	
	
	static class A {}
	static class B {}
	public static final void sleep(long seconds) {
		try {
			TimeUnit.SECONDS.sleep(seconds);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

对应的线程状态:

```java
...
"BlockedThread-2" #13 prio=5 os_prio=31 tid=0x00007fa794005800 nid=0x5d03 waiting for monitor entry [0x000070000b52d000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at ThreadStateTest$4.run(ThreadStateTest.java:47)
	- waiting to lock <0x00000007ab5494a8> (a java.lang.Class for ThreadStateTest$B)
	at java.lang.Thread.run(Thread.java:748)

"BlockedThread-1" #12 prio=5 os_prio=31 tid=0x00007fa795851000 nid=0x5b03 waiting on condition [0x000070000b42a000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at ThreadStateTest.sleep(ThreadStateTest.java:59)
	at ThreadStateTest$3.run(ThreadStateTest.java:37)
	- locked <0x00000007ab5494a8> (a java.lang.Class for ThreadStateTest$B)
	at java.lang.Thread.run(Thread.java:748)

"WaitingThread" #11 prio=5 os_prio=31 tid=0x00007fa795850000 nid=0x5903 in Object.wait() [0x000070000b327000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007ab2b78a0> (a java.lang.Class for ThreadStateTest$A)
	at java.lang.Object.wait(Object.java:502)
	at ThreadStateTest$2.run(ThreadStateTest.java:22)
	- locked <0x00000007ab2b78a0> (a java.lang.Class for ThreadStateTest$A)
	at java.lang.Thread.run(Thread.java:748)

"TimeWaitingThread" #10 prio=5 os_prio=31 tid=0x00007fa79584f800 nid=0x5703 waiting on condition [0x000070000b224000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at ThreadStateTest.sleep(ThreadStateTest.java:59)
	at ThreadStateTest$1.run(ThreadStateTest.java:10)
	at java.lang.Thread.run(Thread.java:748)
...
```

<br>



### Daemon线程

Deamon线程是一种支持型线程，因为它主要是被用作后台调度以及支持性工作。可以通过调用Thread.setDaemon(true)设置（注意需要在启动线程之前设置！）。

```java
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
public class DaemonThreadTest {
	public static void main(String[] args) {
		Thread thread = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					TimeUnit.SECONDS.sleep(10);
				} catch (InterruptedException e) {
					e.printStackTrace();
				} finally {
					System.out.println("finally run!");
				}
			}
		});
		thread.setDaemon(true);
		thread.start();
	}
}
```

运行上述程序，可以看到在终端没有任何输出。main程序（非daemon线程）在启动了目标线程之后随着main方法执行完毕而结束，此时java虚拟机中已经没有非daemon线程，虚拟机需要退出。java虚拟机中的所有deamon线程需要立即终止，因此被main启动的后台线程也终止了，但是后台线程的finally方法并没有被执行。（注意：后台线程不能依靠finally块中的内容来确保执行关闭或清理资源的逻辑）。

<br>



### 启动和终止线程

一个新的线程对象是由其父线程来进行空间分配的，而子线程继承了父线程是否为daemon、优先级、和加载资源的contextClassLoader、以及可继承的ThreadLocal，同时还会分配一个唯一的id来标识这个child线程。至此，一个能够运行的线程对象就初始化好了，在堆内存中等待着运行。

```java
/**
     * Initializes a Thread.
     *
     * @param g the Thread group
     * @param target the object whose run() method gets called
     * @param name the name of the new Thread
     * @param stackSize the desired stack size for the new thread, or
     *        zero to indicate that this parameter is to be ignored.
     * @param acc the AccessControlContext to inherit, or
     *            AccessController.getContext() if null
     * @param inheritThreadLocals if {@code true}, inherit initial values for
     *            inheritable thread-locals from the constructing thread
     */
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }
        this.name = name;

        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            if (security != null) {
                g = security.getThreadGroup();
            }
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }
        g.checkAccess();
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }
        g.addUnstarted();
        this.group = g;
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;
        setPriority(priority);
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;
        tid = nextThreadID();
    }
```

在线程初始化完成之后，调用start方法就可以启动这个线程，调用start方法的含义是：当前线程（及parent线程）同步告知java虚拟机，只要线程规划器空闲，应立即启动调用start方法的线程。

注意，启动一个线程之前最好为这个线程设置一个线程名称，因为这样在使用jstack分析程序或进行排查的过程中，会给开发人员一些提示。<br>



### 安全的终止线程

中断可以理解为线程的一个标识位属性，它表示一个运行中的线程是否被其他线程进行了中断操作。中断好比其他线程对该线程打了一个招呼，其他线程通过调用该线程的interrupt()方法对其进行中断操作。

线程通过检查自身是否被中断进行响应，通过调用自身的isInterrupted()来进行判断是否被中断，也可以调用静态方法 Thread.interrupted() 对当前线程的中断表标识位进行`复位`。如果该线程已处于终结状态，即使该线程被中断过，在调用该线程对象的 isInterrupted() 时依旧会返回false。

从java的api中可以看到，许多声明抛出 InterruptedException的方法（例如sleep），这些方法在抛出InterruptedException之前，java虚拟机都会讲该线程的中断标识位清除，然后抛出InterruptedException，此时调用isInterrupted方法将会返回false。isInterrupted只是表示当前线程被终止过并且中断标识位没有被复位（调用Thread.isInterrupted() 或 抛出InterruptedException异常）。

```java
import java.util.concurrent.TimeUnit;
public class InterruptedTest2 {
	public static void main(String[] args) throws InterruptedException {
		Thread thread1 = new Thread(new Runner1(), "Runner1");
		thread1.setDaemon(true);
		
		Thread thread2 = new Thread(new Runner2(), "Runner2");
		thread2.setDaemon(true);
		
		thread1.start();
		thread2.start();
		
		TimeUnit.SECONDS.sleep(5);
		thread1.interrupt();
		thread2.interrupt();
		
		System.out.println("thread1 interrupted is: " + thread1.isInterrupted());
		System.out.println("thread2 interrupted is: " + thread2.isInterrupted());
		TimeUnit.SECONDS.sleep(2);
	}
	
	static class Runner1 implements Runnable {
		@Override
		public void run() {
			while (true) {
				try {
					TimeUnit.SECONDS.sleep(1);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
	
	static class Runner2 implements Runnable {
		@Override
		public void run() {
			while (true) {
				
			}
		}
	}
}

//运行结果
//可以看到抛出InterruptedException的线程Runner1其中断标识位被清除了，
//而Runner2线程其中断标识位没有被清除。
thread1 interrupted is: false
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at InterruptedTest2$Runner1.run(InterruptedTest2.java:28)
	at java.lang.Thread.run(Thread.java:748)
thread2 interrupted is: true
```

最安全的做法是利用 interrpted属性和一个boolean变量的复合的形式  来控制是否需要停止任务并终止该线程。

```java
import java.util.concurrent.TimeUnit;

public class InterruptedTest {
	public static void main(String[] args) {
		Thread thread01 = new Thread(new Runner(), "ThreadOne");
		thread01.start();
		sleep(1); //睡眠1s，main线程对ThreadOne线程进行中断，使ThreadOne线程能够感知中断而结束
		thread01.interrupt();
		
		Runner target2 = new Runner();
		Thread thread02 = new Thread(target2, "ThreadTwo");
		thread02.start();
		sleep(1); //睡眠1s，main线程对ThreadTwo线程进行取消，使ThreadOne线程能够感知on为false而终端
		target2.cancel();
	}
	
	static class Runner implements Runnable {
		private int count = 0;
		private volatile boolean on = true;
		@Override
		public void run() {
			while (on && !Thread.currentThread().isInterrupted()) {
				count++;
			}
			System.out.println(Thread.currentThread().getName() + ": count is " + count);
		}
		
		public void cancel() {
			this.on = false;
		}
	}
	
	public static void sleep(long secounds) {
		try {
			TimeUnit.SECONDS.sleep(secounds);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

这种通过标志位 或 中断操作的方式能够使线程在终止时候有机会清理资源，而不是武断地将线程终止，因此这种终止线程的做法显得更加安全和优雅。

<br>



### 线程间的通信之volatile和synchronized关键字

java支持多个线程同时访问一个对象或者对象的成员变量，由于每个线程都可以拥有这个变量的拷贝（虽然对象及成员变量分配的内存是在共享内存中的，但是每个执行线程还是拥有一份拷贝，这样做的目的是加速程序的执行，这是现代多核处理器的一个显著特性），所以在程序执行的过程中，一个线程看到的变量并不一定是最新的。

关键字volatile可以用来可以用来修饰字段（成员变量），就是告知程序，任何对该变量的访问均需要从共享内存中获取，而对他的改变必须同步刷新到共享内存中，他能保证所有线程对变量访问的可见性。

举个例子，定义一个表示程序是否运行的成员变量boolean on=true，那么另一个线程可能对它执行关闭动作（on=false），这里涉及到多个线程对变量的访问，因此需要将其定义为volatile boolean on=true，这样当其他新城对他进行访问的时候，可以让所有线程感知到变化，因为所有对on的访问和修改都需要已共享内存为标准。但是过多的使用volatile是没有必要的，因为他会降低程序的执行效率。

关键字synchronized可以修饰方法或者同步代码块，它主要确保多个线程在同一个时刻，只能有一个线程处于方法或者同步代码块中，他保证了线程对变量访问的可见性和排他性。

<br>



### 线程的等待和通知机制

一个线程修改了对象的值，而另一个线程感知到了变化，然后进行相应的操作，整个过程始于一个线程，而终止于另一个线程，而最终执行又是另一个线程。前着是生产者，后者是消费者，这种模式隔离了 “做什么” 和 “怎么做”，在功能上实现了解耦，在体系结构上局别了良好的伸缩性。

这种模式简单的实现办法是，让消费者线程不断的循环检查变量是否符合预期，如下面的代码所示：

```java
while (value != desire) {
  Thread.sleep(1000);
}
doSomething();
```

上面的代码在条件value不是期望值时就睡眠一段时间，这样做的目的是防止过快地无偿消耗，这种方式看似能够实现所需的功能，但是query存在以下问题：

* 难以保证及时性。在睡眠时，基本不消耗处理器资源，但是如果睡的太久就不能及时发现条件的变化，实时性难以保证。
* 难以降低开销。如果降低睡眠的时间，比如睡1ms，这样虽然能够及时发现变化，却造成了处理器资源更大的浪费。

以上问题看似矛盾难以调和，庆幸的是java通过内置的等待-通知机制完美的解决了所需的功能。等待-通知相关的方法是任何java对象都具备的，因为他被定义在所有对象的超类Object里面。

* wait()：调用该方法进入WAITING状态，只有等待另外线程的通知或被中断才会返回，需要注意的是，调用该方法会释放对象的锁（而sleep则不会释放锁）。
* wait(long)：超时等待一段时间（ms），如果没有通知就返回。
* wait(long, int)：对于超时时间更加精准的控制，可以达到ns。
* notify()：通知一个在对象上等待的线程，使其从wait()方法返回，而返回的前提是该线程获取到了对象的锁。
* notifyAll()：通知所有等待在该对象上的线程。

等待-通知机制，是指一个线程A调用了对象O的wait方法，进入等待状态，而另一个线程B调用了O对象的notify或notifyAll方法，线程A接收到通知以后从对象O的wait方法返回，进而执行后续的操作。上述两个线程通过对象O来完成交互，而对象上的wait和notify或notfyAll的关系就好像开关信号一样，用来完成等待方和通知方的交互工作。<br>

```java
import java.util.concurrent.TimeUnit;
public class WaitNotifyTest {
	public static void main(String[] args) {
		Thread waitThread = new Thread(new Wait(), "WaitThread");
		waitThread.start();
		sleep(1);
		
		Thread notifyThread = new Thread(new Notify(), "NotifyThread");
		notifyThread.start();
	}
	
	static class A {
		static volatile boolean ok = false;
		static Object lock = new Object();
	}
	
	static class Wait implements Runnable {
		@Override
		public void run() {
			synchronized (A.lock) {
				//当条件不满足时，继续wait，同时释放lock的锁
				while (!A.ok) { //也可以是if，但最好是while，以达到被通知后再次校验的目的
					try {
						System.out.println(Thread.currentThread()
                                           .getName() + " ok is "+ A.ok +".");
						A.lock.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
				//当条件满足时，完成工作
				System.out.println(Thread.currentThread().getName() 
                                   + " ok is "+ A.ok +", done!");
			}
		}
	}
	
	static class Notify implements Runnable {
		@Override
		public void run() {
			//加锁
			synchronized (A.lock) {
				//获取lock的锁，然后进行通知，通知时不会释放lock的锁
				//直到释放了lock的锁之后，waitThread才能从wait方法中返回
				System.out.println(Thread.currentThread() 
                                   + " hold the lock. notifyAll...");
				A.ok = true;
              	A.lock.notifyAll();
				sleep(5);
			}
			
			//再次加锁
			synchronized (A.lock) {
				System.out.println(Thread.currentThread() 
                                   + " hold the lock again. sleeping...");
				sleep(5);
			}
		}
	}
	
	
	public static void sleep(long seconds) {
		try {
			TimeUnit.SECONDS.sleep(seconds);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}


//运行结果(第三行和第四行的输出顺序可能会互换)
WaitThread ok is false.
Thread[NotifyThread,5,main] hold the lock. notifyAll...
Thread[NotifyThread,5,main] hold the lock again. sleeping...
WaitThread ok is true, done!
```

需要注意的细节如下：

* 使用wait、notify、notifyAll时需要先调用对象加锁，并且等待-通知两个线程需要使用同一个锁对象
* 调用wait方法以后，线程状态由RUNNING变为WAITING，并将当前线程放置到对象的等待队列
* notify或notifyAll方法被调用后，等待线程依旧不会从wait返回，只有调用notify或notifyAll的线程释放锁之后，等待线程才有机会抢到锁从wait返回。
* notify方法是将等待队列中的一个等待线程从等待队列中移到同步队列中，而notifyAll方法则是将等待队列中所有的线程都移到同步队列中，被移动的线程由WAITING状态变为BLOCKED状态。

<br>



**等待-通知模式的经典范式**

```java
//等待方遵循以下原则
//1. 获取对象的锁
//2. 如果条件不满足，那么就调用对象的wait方法，被通知后仍要检查条件
//3. 满足条件则执行对应的逻辑
synchronized (A对象) {
	while (条件不满足) {
 		A对象.wait()
    }
  	条件满足时的逻辑
}

//通知方遵循如下原则
//1. 获得对象的锁
//2. 改变条件
//3. 通知所有等待在对象上的线程
synchronized (A对象) {
 	改变条件
    A对象.notifyAll();
}


//注意上述语法中：
//等待方调用A对象等待方法的时候可以不强制加锁
//通知方调用A对象的通知方法的时候必须加A对象的锁！
```

<br>



### thread.join()方法的使用

如果在一个线程A中执行了thread.join()语句，表示当前线程A将等待thread线程终止之后才从thread.join()处返回。除了join()外，Thread还提供了join(long)和join(long,int)两个超时方法，如果线程thread没有在给定的时间内终止，那么将会从超时方法中返回。（join方法的底层仍是使用的java object的等待-通知机制实现的）

```java
public class JoinTest {
	public static void main(String[] args) {
		Thread prev = Thread.currentThread();
		
		for (int i = 0; i < 10; i++) {
			//每一个线程都拥有前一个线程的引用，需要等待前一个线程的终止，才能从等待中返回
			Thread thread = new Thread(new DomainoRunner(prev), "Thread" + i);
			thread.start();
			prev = thread;
		}
	}
	
	static class DomainoRunner implements Runnable {
		private Thread prev;
		
		public DomainoRunner(Thread prev) {
			this.prev = prev;
		}

		@Override
		public void run() {
			try {
				prev.join();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread().getName() + " terminated!");
		}
	}
}

//运行结果：
Thread0 terminated!
Thread1 terminated!
Thread2 terminated!
Thread3 terminated!
Thread4 terminated!
Thread5 terminated!
Thread6 terminated!
Thread7 terminated!
Thread8 terminated!
Thread9 terminated!
```

<br>



### 管道流

管道流和普通的文件或网络输入输出流不同之处在于，它主要用于在线程之间进行数据的传输，而传播的媒介为内存。管道流的四个实现类：

* PipedInputStream / PipedOutputStream
* PipedReader / PipedWriter

管道流输出流和输入流在使用之前必须要先进行绑定，也就是调用connect()方法，如果没有将输入/输出流绑定起来，对于该流的访问就会出现异常。

```java
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PipedReader;
import java.io.PipedWriter;
public class PipedTest {
	public static void main(String[] args) throws IOException {
		PipedWriter  pw = new PipedWriter();
		PipedReader pr = new PipedReader();
		pw.connect(pr); //
		
		Thread printThread = new Thread(new PrintThread(pr), "PrintThread");
		printThread.start();
		String line = null;
		BufferedReader br = new BufferedReader(
          new InputStreamReader(new FileInputStream("xxx")));
		while ((line=br.readLine()) != null) {
			pw.write(line);
		}
		
		br.close();
		pw.close();
	}
	
	
	static class PrintThread implements Runnable {
		private PipedReader r;
		
		public PrintThread(PipedReader r) {
			this.r = r;
		}

		@Override
		public void run() {
			BufferedReader br = new BufferedReader(r);
			try {
				String line = null;
				while ((line=br.readLine()) != null) {
					System.out.println(line);
				}
			} catch (IOException e) {
				e.printStackTrace();
			} finally {
				try {
					br.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	}
}
```

<br>



### 线程变量（ThreadLocal）

ThreadLocal，即线程变量，是一个以ThreadLocal对象为键、任意对象为值的存储结构。这个结构被附带在线程上，也就是说一个线程可以根据一个ThreadLocal对象，查询到绑定在这个线程上的一个值。

* set(T)：设置一个绑定当前线程的值
* get(T)：获取一个绑定当前线程的值

```java
import java.util.concurrent.TimeUnit;
public class ThreadLocalTest {
	private static ThreadLocal<Long> TL_TIME = new ThreadLocal<>();
	
	public static void main(String[] args) throws InterruptedException {
		TL_TIME.set(System.currentTimeMillis());
		TimeUnit.SECONDS.sleep(2);
		System.out.println("耗时(ms)：" + (System.currentTimeMillis()-TL_TIME.get()));
	}
}
```

<br>



### 等待超时模式

开发人员经常会面临这样的调用场景：调用一个方法时候等待一段时间（一般来说会给一个时间段），如果该方法在给定的时间内得到结果，那么结果立刻返回，反之超时返回默认结果。

前面介绍了等待通知的经典范式，即 加锁-条件循环-处理逻辑 3个步骤，而这种范式无法做到超时等待。而超时等待的加入，需要对经典范式做一下小的改动，改动的内容如下：

```java
public synchronized Object get(long mills) throws InterruptedException {
  	long future = System.currentTimeMillis() + mills;
  	long remaining = mills;
  	//当剩余时间大于0并且result返回值不满足要求
  	while (result==null && remaining>0) {
     	wait(remaining);
      	remaining = future-System.currentTimeMillis();
  	}
  	return result;
}
```

可以看出，等待超时模式就是在等待-通知范式的基础上增加了超时控制，这使得该模式相比原有模式更具有灵活性，因为即使方法执行的时间过长，也不会永久阻塞调用者，而是会按照调用者的要求“按时”返回。

<br>



### CountDownLatch

一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。CountDownLatch是在java1.5被引入的，跟它一起被引入的并发工具类还有CyclicBarrier、Semaphore、ConcurrentHashMap和BlockingQueue，它们都存在于java.util.concurrent包下。CountDownLatch这个类能够使一个线程等待其他线程完成各自的工作后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

主要方法:

* public CountDownLatch(int count);
* public void **countDown**();
* public void **await**() throws InterruptedException

示例：

```java
public class CountDownLatchDemo {  
    final static SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
    public static void main(String[] args) throws InterruptedException {  
        CountDownLatch latch=new CountDownLatch(2);//两个工人的协作  
        Worker worker1=new Worker("zhang san", 5000, latch);  
        Worker worker2=new Worker("li si", 8000, latch);  
        worker1.start();//  
        worker2.start();//  
        latch.await();//等待所有工人完成工作  
        System.out.println("all work done at "+sdf.format(new Date()));  
    }  
      
    static class Worker extends Thread{  
        String workerName;   
        int workTime;  
        CountDownLatch latch;  
        
        public Worker(String workerName ,int workTime ,CountDownLatch latch){  
             this.workerName=workerName;  
             this.workTime=workTime;  
             this.latch=latch;  
        }  
        
        public void run(){  
            System.out.println("Worker "+workerName+
                               " do work begin at "+sdf.format(new Date()));  
            doWork();//工作了  
            System.out.println("Worker "+workerName
                               +" do work complete at "+sdf.format(new Date()));  
            latch.countDown();	//工人完成工作，计数器减一  
  
        }  
          
        private void doWork(){  
            try {  
                Thread.sleep(workTime);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    } 
}  

//运行结果：
Worker zhang san do work begin at 2011-04-14 11:05:11
Worker li si do work begin at 2011-04-14 11:05:11
Worker zhang san do work complete at 2011-04-14 11:05:16
Worker li si do work complete at 2011-04-14 11:05:19
all work done at 2011-04-14 11:05:19
```

<br>
