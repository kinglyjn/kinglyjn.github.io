---
layout: post
title:  "JAVA中的线程池-Executor框架"
desc: "JAVA中的线程池-Executor框架"
keywords: "JAVA中的线程池-Executor框架,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



> 在java中，使用线程来异步执行任务，java线程的创建与销毁需要增加一定的开销，如果我们为每一个任务创建一个新线程来执行，这些线程的创建和销毁需要耗费大量的计算资源，同时这种策略可能会使处于高负荷状态的应用最终崩溃。
>
> Java的线程既是工作单元，也是执行机制。从JDK1.5开始，把工作单元与执行机制分离开来，工作单元包括Runnable和Callable，而执行机制由Executor框架提供。



在HotSpot VM的线程模型中，java线程（Thread）被一对一映射为本地操作系统的线程。java线程启动时会创建一个本地操作系统线程；当该java线程终止时，这个操作系统线程也会被回收。操作系统会调度所有线程并将他们分配给可用的CPU。

在上层中，JAVA多线程程序把应用分解为若干个任务，然后使用用户级别的调度器（Executor框架）将这些任务映射为固定数量的线程；在底层，操作系统内核将这些线程映射到硬件处理器上。应用程序通过Executor框架控制上层的调度，而下层的调度则由操作系统来控制。这种两级调度模型的示意图如下：

<img src="http://img.blog.csdn.net/20170801163338481?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

Executor框架主要由三大部分组成：

* 任务：包括被执行任务需要实现的接口，Runnable接口或者Callable接口。
* 任务的执行：包括任务执行机制的核心接口Executor，以及继承自Executor的ExecutorService接口。Executor接口有两个关键类实现了ExecutorService接口，它们是ThreadPoolExecutor和ScheduledthreadPoolExecutor。
* 异步计算的结果：包括接口Future和实现Future接口的FutureTask。

Executor框架的主要接口：

<img src="http://img.blog.csdn.net/20170801201454118?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/>

Executor：是一个接口，他是Executor框架的基础，它将任务的提交和任务的执行分离开来。

ThreadPoolExecutor：是线程池的核心实现类，用来执行被提交的任务。

ScheduledThreadPoolExecutor：是一个实现类，可以在给定的延时后运行命令，或者定期执行命令。ScheduledThreadPoolExecutor要比Timer更加灵活，功能更加强大。

Future：Future接口以及FutureTask实现类代表异步计算的结果。

Runnable接口和Callable接口的实现类：都可以被ThreadPoolExecutor或ScheduledThreadExecutor执行。

<br>



**Executor框架的使用示意图如下：**

<img src="http://img.blog.csdn.net/20170801202213386?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

1. 主线程首先要创建实现Runnable或Callable接口的任务对象。工具类Executors可以把一个Runnable对象封装成为一个Callable对象（Executors.callable(Runnable task) 或者 Executors.callable(Runnable task, Object result)）。
2. 然后可以把Runnable对象直接交给ExecutorService执行（ExecutorService.execute(Runnable command)），或者也可以把Runnable对象或Callable对象交给ExecutorService执行（ExecutorService.submit(Runnable task) 或 ExecutorService.submit(Callable<T> t)）。如果执行submit方法，ExecutorService将返回一个实现Future接口的对象（到目前为止的JDK中返回的是FutureTask对象）。由于FutureTask实现了Runnable接口，程序员也可以创建FutureTask，然后直接交给ExecutorService执行。
3. 最后主线程可以执行FutureTask.get()方法来等待任务执行完成。主线程也可以执行FutureTask.cancel(boolean matInterruptIfRunning) 来取消此任务的执行。

<br>



### Executor框架的成员

主要成员有 ThreadPoolExecutor、ScheduledThreadPoolExecutor、Future接口|Runnable接口|Callable接口、Executors工具类。

<br>



#### ThreadPoolExecutor

ThreadPoolExecutor通常使用工厂类Executors来创建，Executors可以创建3种类型的ThreadPoolExecutor，即fixedThreadPool、singleThreadExecutor、cachedThreadPool。

* fixedThreadPool：fixedThreadPool适用于为了满足资源管理的需求，需要限制当前线程数量的应用场景，它适用于负载比较重的服务器，下面是创建使用固定线程数的fixedThreadPool的API：

  ```java
  //这里将keepAliveTime设置为0，意味着多余的空闲线程将会立即被终止
  //使用无界LinkedBlockingQueue作为工作队列(队列的容量为Integer.MAX_VALUE)，这将会带来如下影响：
  //1. 当线程池中的线程数到达corePoolSize后，新任务将在误解队列中等待，因此线程池中的线程数不会超过corePoolSize
  //2. 由于1，使用无界队列时maximumPoolSize将会是一个无效的参数
  //3. 由于1和2，使用无界队列的时keepAliveTime将会是一个无效的参数
  //4. 由于使用无界队列，运行中的fixedThreadPool(未执行shutdown或shutdownNow)不会拒绝任务(不会调用RejectedExecutionHandler.rejectedExecution方法)
  public static ExecutorService newFixedThreadPool(int nThreads) {
      return new ThreadPoolExecutor(nThreads, 
                                    nThreads,
                                    0L, 
                                    TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
  }


  public static ExecutorService newFixedThreadPool(int nThreads, 
                                                   ThreadFactory threadFactory) {
  	return new ThreadPoolExecutor(nThreads, 
                                    nThreads,
                                    0L, 
                                    TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory);
  }


  //附：创建ThreadPoolExecutor的API
  new ThreadPoolExecutor(corePoolSize, 
                         maximumPoolSize, 
                         keepAliveTime, //为空闲线程等待新任务的最长时间，超时后多余的线程被终止
                         milliseconds, 
                         runnableTaskQueue, 
                         handler);
  ```

* singleThreadExecutor：适用于需要保证顺序地执行各个任务，下面是singleThreadExecutor的API：

  ```java
  //singleThreadExecutor的corePoolSize和maximumPoolSize都被设置为1，
  //其他参数与fixedThreadPool相同
  //singleThreadExecutor使用也是使用无界队列LinkedBlockingQueue作为工作队列，这对线程池带来的影响与与fixedThreadPool相同
  public static ExecutorService newSingleThreadExecutor() {
      return new FinalizableDelegatedExecutorService(
        new ThreadPoolExecutor(1, 
                               1,
                               0L, 
                               TimeUnit.MILLISECONDS,
                               new LinkedBlockingQueue<Runnable>()));
  }

  public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
  	return new FinalizableDelegatedExecutorService(
        new ThreadPoolExecutor(1, 
                               1,
                               0L, 
                               TimeUnit.MILLISECONDS,
                               new LinkedBlockingQueue<Runnable>(),
                               threadFactory));
  }
  ```

* cachedThreadPool：是一个大小无界的线程池，适用于执行很多短期异步任务的小程序，或者负载比较轻的服务器。下面是cachedThreadPool的API：

  ```java
  //cachedThreadPool使用没有容量的SynchronousQueue作为线程的工作队列，
  //但cachedThreadPool是无界的。这就意味着，如果主线程提交任务的速度高于maximumPool中线程的处理速度
  //cachedThreadPool会不断创建新的线程，极端情况下可能因此而耗尽CPU资源
  public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, //即corePool为空
                                    Integer.MAX_VALUE, //即maximumPool是无界的
                                    60L, //空闲线程等待新任务的最长时间为60s
                                    TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
  }

  public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
      return new ThreadPoolExecutor(0, 
                                    Integer.MAX_VALUE,
                                    60L, 
                                    TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>(),
                                    threadFactory);
  } 
  ```

<br>



#### ScheduledThreadPoolExecutor

ScheduledThreadPoolExecutor通常使用工厂类Executors来创建。Executors可以创建两种类型的ScheduledThreadPoolExecutor，即ScheduledThreadPoolExecutor和SingleThreadScheduledExecutor。

* ScheduledThreadPoolExecutor：适用于需要多个后台线程执行周期任务，同时为了满足资源管理的需求而需要限制后台线程数量的应用场景。对应的API为：

  ```java
  public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
      return new ScheduledThreadPoolExecutor(corePoolSize);
  }

  public static ScheduledExecutorService newScheduledThreadPool(
              							int corePoolSize, 
    										ThreadFactory threadFactory) {
      return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
  }
  ```

* SingleThreadScheduledExecutor：适用于需要单个后台线程执行周期任务，同时需要保证顺序地执行各个任务的应用场景。对应的API为：

  ```java
  public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
      return new DelegatedScheduledExecutorService(new ScheduledThreadPoolExecutor(1));
  }

  public static ScheduledExecutorService newSingleThreadScheduledExecutor(
    										ThreadFactory threadFactory) {
      return new DelegatedScheduledExecutorService(
        		new ScheduledThreadPoolExecutor(1, threadFactory));
  }
  ```

ScheduledThreadPoolExecutor的`运行机制`：

DelayQueue是一个无界队列，所以ThreadPoolExecutor的maximumPoolSize在ScheduledThreadPoolExecutor中没有什么意义（设置maximumPoolSize没有效果）。

ScheduledThreadPoolExecutor的执行主要分为量大部分：

* 当调用ScheduledThreadPoolExecutor 的 scheduleAtFixedRate()方法或者scheduleWithFixedDelay()方法时候，会向ScheduledThreadPoolExecutor的DelayQueue中添加一个实现了RunnableScheduledFutureTask接口的ScheduledFutureTask。
* 线程池中的线程从DelayQueue中获取ScheduledFutureTask，然后执行。

示意图如下：

<img src="http://img.blog.csdn.net/20170802115335575?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

ScheduledThreadPoolExecutor为了实现周期性地执行任务，对ThreadPoolExecutor做了如下改动：

* 修改使用DelayQueue作为任务队列
* 获取任务的方式不同（后文提到）
* 执行周期任务后，增加了额外的处理（后文提到）

<br>

ScheduledThreadPoolExecutor的`实现`：

当调用ScheduledThreadPoolExecutor 的 scheduleAtFixedRate()方法或者scheduleWithFixedDelay()方法时候，会向ScheduledThreadPoolExecutor的DelayQueue中添加一个实现了RunnableScheduledFutureTask接口的ScheduledFutureTask。ScheduledFutureTask主要包含三个成员变量：

* long time：表示这个任务将要被执行的具体时间
* long sequenceNumber：表示这个任务被添加到ScheduledThreadPoolExecutor中的编号
* long period，表示任务执行的时间间隔

DelayQueue中封装了一个PriorityQueue，这个PriorityQueue会对队列中的ScheduledFutureTask进行排序。排序时，time小的排在前面（时间早的任务将被先执行），如果这两个ScheduledFutureTask的time相同，就比较sequenceNumber，sequenceNumber小的排在前面（也就是说如果任务的执行时间相同，那么先提交的任务将被先执行）。我们先看看ScheduledThreadPoolExecutor中的线程执行周期任务的四个步骤，如下图：

<img src="http://img.blog.csdn.net/20170802120506657?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

对这四个步骤的说明如下：

1. 线程1从DelayQueue中获取已经到期的ScheduledFutureTask（DealyQueue.take()），到期任务是指ScheduledFutureTask的time大于等于当前时间；

   1. 获取lock；
   2. 获取周期任务（会在一循环步骤中执行步骤2，直到线程从PriorityQueue获取到一个元素之后）；
      1. 如果PriorityQueue为空，当前线程到Condition中等待，否则执行下面的2.2；
      2. 如果PriorityQueue的头元素的time比当前时间大，到Condition中等待到time时间，否则执行下下面的2.3；
      3. 获取PriorityQueue的头元素，如果PriorityQueue不为空，则唤醒在Condition中等待的所有线程；
   3. 释放lock；

   源码如下（JDK1.7）

   ```java
   public E take() throws InterruptedException {
       final ReentrantLock lock = this.lock;
       lock.lockInterruptibly();
       try {
           for (;;) {
               E first = q.peek();
               if (first == null)
                   available.await();
               else {
                   long delay = first.getDelay(TimeUnit.NANOSECONDS);
                   if (delay <= 0)
                       return q.poll();
                   else if (leader != null)
                       available.await();
                   else {
                       Thread thisThread = Thread.currentThread();
                       leader = thisThread;
                       try {
                           available.awaitNanos(delay);
                       } finally {
                           if (leader == thisThread)
                               leader = null;
                       }
                   }
               }
           }
       } finally {
           if (leader == null && q.peek() != null)
               available.signal();
           lock.unlock();
       }
   }
   ```

2. 线程1执行这个ScheduledFutureTask；

3. 线程1修改ScheduledFutureTask的time为下一次将要被执行的时间；

4. 线程1报这个修改time后的ScheduledFutureTask放回到DelayQueue中（DealyQueue.add()）

   1. 获取lock；
   2. 添加任务；
      1. 向PriorityQueue添加任务；
      2. 如果在2.1中添加的任务是PriorityQueue的头元素，唤醒在Condition中等待的所有线程；
   3. 释放lock；

   源码如下：

   ```java
   public boolean offer(E e) {
       final ReentrantLock lock = this.lock;
       lock.lock();
       try {
           q.offer(e);
           if (q.peek() == e) {
               leader = null;
               available.signal();
           }
           return true;
       } finally {
           lock.unlock();
       }
   }
   ```

<br>



#### Future

Future接口和实现Future接口的FutureTask表示异步计算的结果。FutureTask除了实现Future接口外，还实现了Runnable接口，因此FutureTask可以交给Executor执行，也可以交由线程直接执行（FutureTask.run()）。当我们把Runnable接口或Callable接口的实现提交（submit）给TreadPoolExecutor或者ScheduledTreadPoolExecutor时，他们会向我们返回一个FutureTask对象，下面是对应的API：

```java
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}

public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}

public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
  	//newTaskFor是一个重载的方法，参数可以是Runnable，也可以是Callable
    RunnableFuture<T> ftask = newTaskFor(task); 
    execute(ftask);
    return ftask;
}
```

有一点需要注意，到目前最新的java1.8为止，java通过上述API返回的是一个实现了Future的FutureTask对象，在将来的JDK中，返回的可能不一定是FutureTask！

根据FutureTask.run()方法的执行时机，FutureTask可以处于以下三种状态：

* 未启动：创建FutureTask，FutureTask.run()还没有被执行以前，将导致调用线程被阻塞。此时执行FutureTask.cancel()将导致此任务永远不能被执行；
* 已启动：FutureTask.run()执行的过程中，将导致调用线程被阻塞；此时执行FutureTask.cancel(true)将以中断此任务线程的方式来视图停止任务，执行FutureTask.cancel(false)将不会对此任务的线程产生影响；
* 已完成：FutureTask.run()方法执行正常结束，或者被取消（FutureTask.cancel(…)），或者执行FutureTask.run()方法时抛出异常而异常结束，FutureTask处于已完成状态。当FutureTask处于已完成状态时，执行FutureTask.get()将导致调用线程立即返回结果，或抛出异常。此时执行FutureTask.cancel(...)将返回false。

<img src="http://img.blog.csdn.net/20170802124203779?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

`FutureTask的简单实使用`

可以把FutureTask交给Executor执行，也可以通过ExecutorService.submit(...)返回一个FutureTask，然后执行FutureTask.get()方法或者FutureTask.cancel(…)方法。除此之外，还可以单独使用FutureTask。

当一个线程需要等待另一个线程把某个任务执行完成后才能继续执行，此时可以使用FutureTask。假设有多个线程执行若干任务，每个线程最多只能被执行一次，当多个线程试图同时执行一个任务时，只允许一个线程执行任务，其他线程需要等待这个任务执行完成后才能继续执行（其实可以有多种实现方式，参见wait-notify、lock、Condition、CountDownLatch、CyclicBarrier等）。下面是对应的示例代码：

```java
private final ConcurrentMap<Object, Future<String>> teskCache = new ConcurrentHashMap<>();
public String executionTask(final String taskName) {
	Future<String> future = teskCache.get(taskName);
	if (future == null) {
		Callable<String> task = new Callable<String>() {
			@Override
			public String call() throws Exception {
				return taskName;
			}
		};
		FutureTask<String> futureTask = new FutureTask<>(task);
		teskCache.putIfAbsent(taskName, futureTask);
		if (future == null) {
			future = futureTask;
			futureTask.run();
		}
	}
	
	String result = null;
	try {
		result = future.get();
	} catch (InterruptedException e) {
		e.printStackTrace();
	} catch (ExecutionException e) {
		e.printStackTrace();
	}
	return result;
}
```

上述代码的执行示意图如下：

<img src="http://img.blog.csdn.net/20170802130754027?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>



`FutureTask的实现`

FutureTask的实现同样是基于AbstractQueuedSynchronizer（AQS）。java.util.concurrent中的很多可阻塞的类（如ReentrantLock）都是基于AQS来实现的。AQS是一个同步框架，他提供通用机制来原子性管理同步状态、阻塞和唤醒线程，以及维护被阻塞线程的队列。JAVA1.6中AQS被广泛使用，基于AQS的同步器包括ReentrantLock、ReentrantReadWriteLock、CountDownLatch、CyclicBarrier、Semaphore和FutureTask。

每一个基于AQS实现的同步器都会包含两种类型的操作：

* 直到有一个acquire操作，这个操作阻塞调用线程，除非/直到AQS允许这个线程继续执行。FutureTask的acquire操作为get()/get(long timeout, TimeUnit unit)
* 只少有一个release操作，这个操作改变AQS状态，改变后的状态可以允许一个或者多个线程被解除阻塞。FutureTask的release操作包括run()方法和cancel(…)方法。

基于`“聚合优于继承的原则”`，FutureTask声明了一个内部私有的继承于AQS的子类Sync，对FutureTask所有的共有方法的调用都会委托给这个内部子类；





####  Runnable和Callable接口

都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor。他们之间的区别是Runnable不会返回结果，而Callable可以返回结果。

除了可以自己创建实现Callable接口的对象外，还可以使用工厂类Executors来把一个Runnable包装成为一个Callable，如下：

```java
//如果提交的是此方法返回的Callable，则FutureTask.get()返回的将是result
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}

//如果提交的是此方法返回的Callable，则FutureTask.get()返回的将是null
public static Callable<Object> callable(Runnable task) { 
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<Object>(task, null);
}
```





