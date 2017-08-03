---
layout: post
title:  "JAVA中的线程池-ThreadPoolExecutor"
desc: "JAVA中的线程池-ThreadPoolExecutor"
keywords: "JAVA中的线程池-ThreadPoolExecutor,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



> java中的线程池是运用场景最多的并发组件，几乎所有的异步或者并发任务的程序都可以使用线程池。在开发过程中使用线程池将会带来3个好处：
>
> * 降低资源的消耗。通过反复利用已创建的线程降低线程创建和销毁造成的消耗；
> * 提高响应速度。当任务到达时，任务可以不需要等到线程创建就立即执行；
> * 提高线程的可管理性。



### 线程池的主要处理流程

当提交一个新任务到线程池时，线程池的主要处理流程如下：

1. 线程池判断核心线程池里的线程是否都在执行任务。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下一个流程。
2. 线程池判断工作队列是否已经满了，如果没有则将提交的任务存储在工作队列里，如果已经满了则进入下一个工作流程。
3. 线程池判断线程池的线程是否都处于工作状态，如果不是则创建一个新的工作线程来执行任务，如果已经满了则交给饱和策略来处理这个任务。

```default
使用者提交任务
    |
    |
    |
    |/            Y                    Y                   Y
核心线程池是否满了--------->队列是否满了--------->线程池是否满了------->按照策略执行无法执行的任务	
    |                        |                    |
    |N                       |N                   |N
    |                        |                    |
    |/                       |/                   |/
创建线程执行任务          将任务存储在队列里       创建线程执行任务     
```

<br>

<img src="http://img.blog.csdn.net/20170801142006205?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/>

ThreadPoolExecutor执行execute方法分为以下4中情况：

* 如果当前运行的线程小于corePoolSize，则创建线程来执行任务（注意：执行这一步需要获取去全局锁）。
* 如果运行的线程等于或者多于corePoolSize，则将任务加入到BlockingQueue。
* 如果无法将任务加入到BlockingQueue（队列已满），则创建新的线程来处理任务（注意执行这一步骤需要获取全局锁）。
* 如果创建新线程将使当前运行的线程超出maximumPoolSize，则任务将被拒绝，并调用RejectedExecutionHandler.rejectedExecution()方法。

ThreadPoolExecutor采取上述步骤的总体设计思路，是为了在执行execute()方法时，尽可能地避免获取全局锁（那将会是一个严重的可伸缩瓶颈）。在ThreadPoolExecutor完成预热之后（当前运行的线程数大于等于corePoolSize），几乎所有的execute方法调用都是执行步骤二，而步骤二不需要获取全局锁。

源码如下(java7)：

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

工作线程：线程池创建线程时，会将线程封装成工作线程Worker，Worker在执行任务后，还会循环获取工作队列里的任务来执行。

线程池中的线程执行任务分为两种情况：

* 在execute()方法中创建一个线程时，会让这个线程执行当前的任务。
* 这个线程执行完当前任务后，会反复从BlockingQueue获取任务来执行。

<img src="http://img.blog.csdn.net/20170801144601037?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/>

<br>



### 线程池的使用

我们可以通过ThreadPoolExecutor来创建一个线程池，常用的构造方法如下 ：

```java
new ThreadPoolExecutor(corePoolSize, 
                       maximumPoolSize, 
                       keepAliveTime, 
                       milliseconds, 
                       runnableTaskQueue, 
                       handler);
```

* corePoolSize：线程池的基本大小，当提交一个任务到线程池时，线程池会出创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池的基本大小时就不在创建。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有的基本线程；

* runnableTaskQueue：任务队列，用于保存等待执行的阻塞队列，可以选择使用以下几种阻塞队列：

  * ArrayBlockingQueue：基于数组结构的有界阻塞队列
  * LinkedBlockingQueue：基于链表结构的阻塞队列
  * SynchronousQueue：一个不存储元素的阻塞队列，每个插入操作必须等到另一个点成调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列
  * PriorityBlockingQueue：一个具有优先级的无限阻塞队列

* maximumPoolSize：线程池的最大数量，线程池允许创建的最大线程数。如果队列池满了，并且已创建的线程数小于最大线程数，则线程池会创建新的线程来执行任务。指的注意的是，如果使用了无界的任务队列这个参数就不会有什么效果；

* ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个床架出来的线程设置更有意义的名称。

* RejectedExcutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略来处理提交的新任务。者个策略默认情况下是AbortPolicy，表示无法处理任务时抛出异常。在JDK1.5中java线程池提供了以下4中策略:

  * AbortPolicy：直接抛出异常
  * CallerRunsPolicy：只用调用者所在线程来运行任务
  * DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前的任务
  * DiscardPolicy：不处理，丢弃掉

  当前也可以根据应用场景来实现RejectedExcutionHandler接口定义自定义策略。如记录日志或者持久化存储不能处理的任务；

* keepAliveTime：线程活动保持时间，线程池的工作线程空闲之后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间都很短，可以调大时间，以提高线程的利用率。

* TimeUnit：线程活动保持时间的单位，可选的单位有天（DAYS）、小时（HOURS）、分钟（MINUTES）、毫秒（MILLISECONDS）、微秒（MICROSECONDS，千分之一毫秒）和纳秒（NANOSECONDS，千分之一微秒）

<br>



### 向线程池提交任务

可以使用两个方法向线程池提交任务，分别为execute()和submit方法。

execute用于提交不需要返回值的任务（无法判断任务是否被执行成功），示例：

```java
threadPool.execute(new Runnable() {
	@Override
	public void run() {
		// TODO
	}
});
```

submit方法用于提交需要返回值的任务，线程池会返回一个future类型的对象，通过这个future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值，get()方法阻塞当前线程直到任务完成，而使用get(long timeout, TimeUnit unit)则会阻塞当前线程一段时间后立即返回，这个时候可能任务没有执行完。

```java
Future<?> future = threadPool.submit(taskWithReturnValue);
try {
	Object s = future.get();
} catch (InterruptedException e1) {
	// 处理中断异常
} catch (ExecutionException e1) {
	// 处理无法执行的任务异常
} finally {
	//关闭线程池
	threadPool.shutdown();
}
```

<br>



### 关闭线程池

可以使用线程池的shutdown或者shutdownNow方法来关闭线程池。他们的原理遍历线程池中的工作线程，然后逐个调用线程的inerrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。这两个方法存在一定的区别，shutdownNow首先将线程池的状态设置为STOP，然后尝试停止所有正在执行或暂停任务的线程，并返回等待执行的任务的列表，而shutdown只是将线程的状态设置为SHUTDOWN状态，然后中断所有没有正在执行任务的线程。

只要调用了这两个方法中的任意一个，isShutdown方法就会返回true，当所有的任务都已经关闭了之后，才表示线程池关闭成功，这时候调用isTerminated方法会返回true。至于应该调用哪种方法来关闭线程池，应该由提交到线程池中的任务特性来决定，通常调用shutdown方法来关闭线程池，如果任务不一定要执行完则可以调用shutdownNow方法。

<br>



### 合理配置线程池

要向合理分配线程池，就必须首先分析任务的特性，可以从以下几个角度来分析：

* 任务的性质：CPU密集型任务、IO密集型任务、混合型任务
* 任务的优先级：高、中、低
* 任务的执行时间：长、中、短
* 任务的依赖性：是否依赖其他系统资源，如数据库连接

性质不同的任务可以使不同规模的线程池分开处理。CPU密集型任务应该分配尽可能小的线程，如配置`Ncpu+1`个线程的线程池。由于IO密集型任务线程并不是一直在执行任务，则应分配尽可能多的线程，如` 2*Ncpu` 。混合型的任务，如果可以拆分，将其拆分成一个CPU密集型的任务和一个IO密集型的任务，这要两个任务执行的时间相差不是太大，那么分解后执行的吞吐量将高于串行执行的吞吐量，如果这两个任务执行的时间相差太大，则没必要进行分解。可以通过Runtime.getRuntime().avaliableProcessors()方法获得当前设备的CPU个数。

优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。他可以让优先级高的任务先执行。注意如果一直有优先级高的任务提交到队列里，那么优先级低的线程将可能永远必能执行。

执行时间不同的线程可以提交到不同规模的线程池来处理，或者可以使用优先级队列，让执行时间短的任务先执行。

以来数据库连接池的任务，因为提交sql之后需要等待数据库返回结果，等待的时间越长，则CPU的空闲时间就越长，那么线程数应该设置的更大，这样才能更好的利用CPU。

建议使用有节队列：永杰队列能能加系统的稳定性和预警能力，可以根据需要设置的大一点，比如几千。有一次，我们系统里后台任务线程池队列和线程池全满了，不断抛出抛弃任务的异常，通过排查发现是数据库出了问题，导致执行sql变得非常缓慢，因为后台任务线程池里的任务全是需要向数据库查询和插入数据的，所以导致线程池里的工作线程全部阻塞，任务积压在线程池里。如果当时我们设置成有界队列，那么线程池里的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而不只是后台任务出现问题。当然，我们系统的所有任务使用单独的服务器部署的，我们使用不同规模的线程池来完成不同类型的任务，但是出现这样的问题也会影响到其他任务。

<br>



### 线程池的监控

如果在系统中大量使用线程池，则有必要对线程池进行监控，方便在出现问题时，可以根据线程池的使用情况快速定位问题。可以通过线程池提供的参数来进行监控，在监控线程池的时候可以使用以下属性：

* takeCount：线程池需要执行的任务数量
* completedTaskCount：线程池在运行过程中已完成的任务数量，小于等于takeCount
* largestPoolSize：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否已经满过。如果该值等于线程池的最大大小，则表示线程池曾经满过。
* getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。
* getActiveCount：获取活动的线程数。

通过扩展线程池进行监控。可以通过继承线程池来自定义线程池，重写线程池的 beforeExecute、afterExecute和terminated方法，也可以在任务执行前、执行后和线程池关闭前执行一些代码来进行监控。例如监控任务的平均执行时间和最小执行时间等。这几个方法在线程池里是空方法：

```java
protected void beforeExecute(Thread t, Runnable r) { }

protected void afterExecute(Runnable r, Throwable t) { }

protected void terminated() { }
```







