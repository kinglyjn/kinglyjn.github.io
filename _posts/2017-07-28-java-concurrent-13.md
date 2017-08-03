---
layout: post
title:  "JAVA中的并发工具类"
desc: "JAVA中的并发工具类"
keywords: "JAVA中的并发工具类,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



> 在JDK的并发包里面提供了几个非常有用的并发工具类。CountDownLatch、CyclicBarrier、Semaphore工具类提供了一种并发流程控制的手段，Exchager工具类则提供了在线程间交换数据的一种手段。下面我们通过一定的应用场景来介绍如何使用这些工具。



### 等待多线程完成的CountDownLatch

CountDownLatch允许一个或多个线程等待其他线程完成操作。假设有这样一个需求，我们需要解释的一个Excel里多个sheet的数据，此时可以考虑使用多线程，每个线程解析一个sheet里的数据，等到所有的sheet都解析完之后，程序需要提示解析完成。在这个需求中，要实现主线程等待所有线程完成sheet的解析操作。最简单的做法是使用join方法，代码如下：

````java

public class JoinCountDownLatchTest {
	public static void main(String[] args) throws InterruptedException {
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println("t1 finished!");
			}
		});
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println("t2 finished!");
			}
		});
		
		t1.start();
		t2.start();
		
		t1.join();
		t2.join();
		System.out.println("all finished!");
	}
}

//运行结果：
t1 finished!
t2 finished!
all finished!
````

join用于让当前执行线程等待join线程执行结束。其实现原理是不停检查join线程是否存活，如果join线程存活则rag当前线程无限等待下去。其中wait(0)表示永远等待下去，代码片段如下：

```java
public final synchronized void join(long millis)
    throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) { 
            wait(0); //////////
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

直到join线程终止之后，线程的this.notifyall()方法会被调用，调用notifyall方法是在JVM里面实现的，所以在jdk里看不到，大家可以查看JVM源码。在JDK1.5之后，并发包里的CountDownLatch也可以完成join的功能，并且比join的功能更强大。代码如下：

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchTest {
	static CountDownLatch c = new CountDownLatch(2);
	
	public static void main(String[] args) throws InterruptedException {
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(1);
				c.countDown();
				System.out.println(2);
				c.countDown();
			}
		}).start();
		
		c.await();
		System.out.println(3);
	}
}

//运行结果
1
2
3
```

CountDownLatch测试程序2：

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

public class CountDownLatchTest2 {
	static final int RUNNER_COUNT = 10;
	static AtomicInteger STATISTICAL_RESULT = new AtomicInteger();
	
	//保证所有的Runner线程能够同时开始
	static final CountDownLatch START_CDL = new CountDownLatch(1);
	//保证所有的Runner线程都运行结束后才继续执行
	static final CountDownLatch END_CDL = new CountDownLatch(RUNNER_COUNT);
	
	static class Runner implements Runnable {
		@Override
		public void run() {
			try {
				START_CDL.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			
			STATISTICAL_RESULT.incrementAndGet();
			System.out.println(Thread.currentThread().getName() + " done!");
			
			END_CDL.countDown();
		}
	}
	
	public static void main(String[] args) {
		for (int i = 0; i < RUNNER_COUNT; i++) {
			Thread t = new Thread(new Runner(), "Thread-"+i);
			t.start();
		}
		
		START_CDL.countDown();
		
		try {
			END_CDL.await();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("最终的统计结果是：" + STATISTICAL_RESULT.get());
	}
}

//运行结果(每次输出的顺序很可能不同)
Thread-3 done!
Thread-6 done!
Thread-7 done!
Thread-2 done!
Thread-5 done!
Thread-0 done!
Thread-4 done!
Thread-8 done!
Thread-9 done!
Thread-1 done!
最终的统计结果是：10
```

CountDownLatch的函数接收一个int类型的参数作为计数器，如归你想等待N个点完成，这里就传入N即可。

当我们调用CountDownLatch的countDown方法，N就会减1，CountDownLatch的await方法会阻塞当前线程，直到N变为0，await方法返回。由于countDown方法可以用在任何地方，所以这里说的N个点可以是N个线程、也可以是一个线程里的N个步骤，用在多线程时，只需要把这个CountDownLatch的引用传递到线程里面即可。

如果某个解析sheet的线程处理的比较慢，我们不可能让主线程一直等待下去，所以可以使用另外的一个指定时间的await方法，即await(long time, TimeUnit unit)，这个方法等待特定的时间之后，就不会阻塞当前的线程。join也有类似的方法。

注意：CountDownLatch计数器的值必须大于0，只是等于0的时候，计数器就是0，这时调用await方法不会阻塞当前线程。CountDownLatch不可能重新初始化或者修改CountDownLatch对象的内部计数器的值。

<br>



### 同步屏障-CyclicBarrier

CyclicBarrier的字面意思就是可循环使用的屏障。它主要做的事情是让一组线程一个屏障（也可以称作一个同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续执行。

CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞，示例代码如下：

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierTest {
	static final CyclicBarrier CYCLIC_BARRIER = new CyclicBarrier(2);
	
	public static void main(String[] args) {
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					CYCLIC_BARRIER.await();
				} catch (InterruptedException | BrokenBarrierException e) {
					e.printStackTrace();
				}
				System.out.println(1);
			}
		}).start();
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					CYCLIC_BARRIER.await();
				} catch (InterruptedException | BrokenBarrierException e) {
					e.printStackTrace();
				}
				System.out.println(2);
			}
		}).start();
	}
}

//运行结果：
1
2

//或者
2
1
```

因为上述两个线程的调度是由CPU决定的，两个线程都有可能先执行所以会产生两种输出。如果把new CyclicBarrier(2) 改成new CyclicBarrier(3)，则两个线程将会永远等待(主线程不会等待，因为主线程中没有调用CyclicBarrier的await方法)，因没有第三个线程到达屏障，所以之前到达屏障的两个线程都不会继续执行。

CyclicBarrier还提供了一个更高级的构造函数CyclicBarrier(int parties, Runnable barrierAction)，用于线程到达屏障的时候，优先执行barrierAction，方便处理更复杂的业务场景，代码如下：

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierTest2 {
	static final CyclicBarrier CYCLIC_BARRIER = new CyclicBarrier(2, new HigherPriorityRunner());
	
	static class HigherPriorityRunner implements Runnable {
		@Override
		public void run() {
			System.out.println("higherPriorityRunner done!");
		}
	}
	
	static class Runner implements Runnable {
		@Override
		public void run() {
			try {
				CYCLIC_BARRIER.await();
			} catch (InterruptedException | BrokenBarrierException e) {
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread().getName() + " done!");
		}
	}
	
	public static void main(String[] args) {
		new Thread(new Runner(), "Thread-Runner-01").start();
		new Thread(new Runner(), "Thread-Runner-02").start();
	}
}


//运行结果(线程1和线程2的输出结果可能顺序不一样，但是总是higherPriorityRunner的结果先输出)
higherPriorityRunner done!
Thread-Runner-02 done!
Thread-Runner-01 done!
```

CyclicBarrier可以用于多线程计算数据，最后合并结果的场景的。例如，用一个Excel保存了用户的所有银行流水，每个Sheet保存账户近一年的每笔银行流水，现在需要统计用户的日均银行流水，都执行完之后，得到每个sheet的日均银行流水，最后再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水，代码如下：

```java
import java.util.Map.Entry;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.Executor;
import java.util.concurrent.Executors;

public class BankWaterSevice implements Runnable {
	//创建一个屏障，拦截4个处理线程，当4个处理线程都达到屏障之后，执行当前类的run方法
	private CyclicBarrier CYCLIC_BARRIER = new CyclicBarrier(4, this);
	//假设只有4个sheet，所以只启动4个线程
	private Executor executor = Executors.newFixedThreadPool(4);
	//保存每个sheet计算出来的银流结果
	private ConcurrentHashMap<String, Integer> sheetBankWaterCount = new ConcurrentHashMap<>();
	
	@Override
	public void run() {
		int result = 0;
		for (Entry<String, Integer> sheet: sheetBankWaterCount.entrySet()) {
			result += sheet.getValue();
		}
		sheetBankWaterCount.put("result", result);
		System.out.println(sheetBankWaterCount);
	}
	
	public void count() {
		for (int i = 0; i < 4; i++) {
			executor.execute(new Runnable() {
				@Override
				public void run() {
					//计算当前sheet的银流数据，计算代码略
					sheetBankWaterCount.put(Thread.currentThread().getName(), 1);
					
					//引流数据计算完成，插入预设的屏障(告诉屏障我已到达)
					try {
						CYCLIC_BARRIER.await();
					} catch (InterruptedException | BrokenBarrierException e) {
						e.printStackTrace();
					}
				}
			});
		}
	}
	
	public static void main(String[] args) {
		BankWaterSevice bankWaterSevice = new BankWaterSevice();
		bankWaterSevice.count();
	}
}

//运行结果
{result=4, pool-1-thread-1=1, pool-1-thread-3=1, pool-1-thread-2=1, pool-1-thread-4=1}
```

使用线程池创建4个线程，分别计算每个sheet里的数据，每个sheet里的数据结算结果为1，再由BankWaterSevice的run方法汇总4个sheet线程计算出的结果。

<br>



**CyclicBarrier和CountDown的区别**

CountDown的计数器只能使用一次，而CyclicBarrier计数器可以使用reset()方法进行重置。所以CyclicBarrier能处理更加复杂的业务场景。例如，计算发生错误，可以重置计数器，并让线程重新执行一次。

CyclicBarrier还提供其他有用的方法，比如：

* getNumberWaiting()：可以获得CyclicBarrier阻塞的线程的数量
* isBroken()：用来判断阻塞的线程是否被中断

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierTest3 {
	static final CyclicBarrier CYCLIC_BARRIER = new CyclicBarrier(2);
	
	public static void main(String[] args) {
		Thread t = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					CYCLIC_BARRIER.await();
					System.out.println("哈哈");
				} catch (InterruptedException | BrokenBarrierException e) {
					e.printStackTrace();
				}
			}
		});
		
		t.start();
		t.interrupt();
		
		try {
			CYCLIC_BARRIER.await();
		} catch (InterruptedException | BrokenBarrierException e) {
			System.out.println(CYCLIC_BARRIER.isBroken());
		}
	}
}

//运行结果：
true
java.lang.InterruptedException
	at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:211)
	at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
	at CyclicBarrierTest3$1.run(CyclicBarrierTest3.java:12)
	at java.lang.Thread.run(Thread.java:748)
```

<br>



### 控制并发线程数的Semaphore

Semaphore（信号量）是控制同时访问特定资源的线程数量的，他通过协调各个线程，以保证合理地使用公共资源。Semaphore可以用于做流量控制，特别是公用资源有限的场景，比如数据库连接。假如有一个需求，要读取几万个文件的数据，因为都是IO密集型任务，我们可以启动几十个线程并发地读取，但是读到内存之后还需要存储到数据库中，而数据库的连接数只有10个，这时我们必须控制最多只能有10个线程同时获取数据库连接进行保存数据，否则会报错无法获取数据库连接。这个时候可以使用Semaphore来做限流！

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreTest {
	private static final int THREAD_COUNT = 30;
	private static ExecutorService THREAD_POOL =
     		 Executors.newFixedThreadPool(THREAD_COUNT);
	private static Semaphore SEMAPHORE = new Semaphore(10);
	
	public static void main(String[] args) {
		for (int i = 0; i < THREAD_COUNT; i++) {
			THREAD_POOL.execute(new Runnable() {
				@Override
				public void run() {
					try {
						SEMAPHORE.acquire();
						//数据库操作
						System.out.println(Thread.currentThread().getName()
                                           + " saving data...");
					} catch (InterruptedException e) {
						e.printStackTrace();
					} finally {
						SEMAPHORE.release();
					}
				}
			});
		}
		//关闭线程池
		THREAD_POOL.shutdown();
	}
}
```

在上述代码中，虽然有30个线程在运行，但是只允许10个线程并发执行。Semaphore的构造方法接收一个整形的数字表示可用的许可证数量。Semaphore(10)表示允许10个线程获取许可证，也就是最大并发数是10。Semaphore的使用也很简单，首先使用acquire方法获取一个许可证，使用完之后使用release()方法归还许可证，还可以使用tryAcquire方法尝试获取许证。

Semaphore还提供一些其他方法，具体如下：

* int availablePermits()：返回此信号量中当前可用的许可证数
* int getQueueLength：返回正在等待获取许可证的线程数
* boolean hasQueuedThread()：是否有线程正在等待获取许可证
* void reducePermits(int reduction)：减少reduction个许可证，是个protected方法
* Collection getQueuedThreads()：返回所有等待许可证的线程集合，是个protected方法

<br>



### 线程间交换数据的Exchanger

Exchanger（交换者）是一个用于线程间协作的工具类。Exchanger用于线程间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据，这两个线程通过exchange方法交换数据，如果第一个线程先执行exchange()方法，他会一直等待第二个线程也执行exchange()方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

下面看一下Exchanger的使用场景。

* Exchanger可以用于遗传算法，遗传算法里需要选出两个人作为交配对象，这时候会交换两人的数据，并使用交叉规则得出两个交配结果。
* Exchanger可以用于校对工作，比如我们需要纸质银行流水通过人工的方式录入成电子银行流水，为了避免错误，采用AB岗两人进行录入，录入到Excel之后，系统需要加载这两个Excel，并对两个Excel数据进行校对，看看是否录入一致。

测试代码如下：

```java
import java.util.concurrent.Exchanger;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExchangerTest {
	private static final Exchanger<String> EXCHANGER = new Exchanger<>();
	private static ExecutorService threadPool = Executors.newFixedThreadPool(2);
	
	public static void main(String[] args) {
		threadPool.execute(new Runnable() {
			@Override
			public void run() {
				try {
					String a = "银行流水A";	//A录入一行流水数据
					EXCHANGER.exchange(a);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		});
		
		threadPool.execute(new Runnable() {
			@Override
			public void run() {
				try {
					String b = "银行流水B";	//B录入一行流水数据
					String a = EXCHANGER.exchange(b);
					System.out.println("A和B数据是否一致：" + 
                                       a.equals(b) + ", A录入的是:" + a 
                                       + ", B录入的是:" + b);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		});
		
		threadPool.shutdown();
	}
}

//运行结果
A和B数据是否一致：false, A录入的是:银行流水A, B录入的是:银行流水B
```

如果两个线程有一个没有执行exchange()方法，则会一直等待，如果担心有特殊情况发生，避免一直等待，可以使用exchange(V v, long timeout, TimeUnit unit)设置最大等待时长。



















