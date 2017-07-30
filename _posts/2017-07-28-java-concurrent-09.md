---
layout: post
title:  "JAVA中的锁（四）-LockSuport"
desc: "JAVA中的锁（四）-LockSuport"
keywords: "JAVA中的锁（四）-LockSuport,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---

### LockSuport工具

LockSupport是JDK中比较底层的类，用来创建锁和其他同步工具类的基本线程阻塞原语。java锁和同步器框架的核心 AQS: AbstractQueuedSynchronizer，就是通过调用 LockSupport .park()和 LockSupport .unpark()实现线程的阻塞和唤醒 的。 LockSupport 很类似于二元信号量(只有1个许可证可供使用)，如果这个许可还没有被占用，当前线程获取许可并继 续 执行；如果许可已经被占用，当前线 程阻塞，等待获取许可。

LockSuport提供的阻塞和唤醒方法：

```java
//阻塞当前线程，当调用unpark或者当前线程被中断，才能从park方法返回
void park()
//阻塞当前线程，blocker用来标识当前线程在等待的对象(简称阻塞对象，该对象主要用于问题排查和系统监控)
park(Object blocker)
  
//阻塞当前线程，最长不超过nanos纳秒
void parkNanos(long nanos)
void parkNanos(Object blocker, long nanos)
  
//阻塞当前线程，直到deadline时间(从1970年到deadline时间的毫秒数)
void parkUntil(long deadline)
void parkUntil(Object blocker, long deadline)
  
//唤醒处于阻塞状态的线程t
void unpark(Thread t)
```

LockSuport是不可重入的的，如果一个线程连续2次调用 LockSupport.park()，那么该线程一定会一直阻塞下去。

```java
public static void main(String[] args) {
	Thread thread = Thread.currentThread();
	
	LockSupport.unpark(thread);
	System.out.println("aa");
	
	LockSupport.park();
	System.out.println("bb");
	
	LockSupport.park();
	System.out.println("cc");
}
//运行结果
//这段代码打印出aa和bb，不会打印cc，因为第二次调用park的时候，线程无法获取许可出现死锁。
```

下面我们来看下LockSupport对应中断的响应性：

```java
import java.util.concurrent.locks.LockSupport;
public class LockSupportTest {
	public static void main(String[] args) throws Exception {
		t2();
	}

	public static void t2() throws Exception {
		Thread t = new Thread(new Runnable() {
			private int count = 0;

			@Override
			public void run() {
				long start = System.currentTimeMillis();
				long end = 0;

				while ((end - start) <= 1000) {
					count++;
					end = System.currentTimeMillis();
				}
				System.out.println("about 1 second after, count=" + count);

				// 等待获取许可
				LockSupport.park();
				System.out.println("thread over. " +
                                   Thread.currentThread().isInterrupted());
			}
		});

		t.start();
		Thread.sleep(2000);
		System.out.println("main thread sleep 2 second done!");

		// 中断线程
		t.interrupt();
		System.out.println("main over");
	}
}

//运行结果：
about 1 second after, count=22987103
main thread sleep 2 second done!
main over
thread over. true
```

最终线程会打印出thread over. true。这说明 线程如果因为调用park而阻塞的话，能够响应中断请求(中断状态被设置成true)，但是不会抛出InterruptedException 。

<br>



### Condition接口

任意一个java对象，都拥有一组监视器方法（定义在Object上），主要包括wait()、wait(long timeout)、notify()、notifyAll()方法，者写方法与synchronized关键字配合，可以实现等待通知模式。Condition接口也提供了类似Object的监控器方法，与Lock配合使用可以实现等待通知模式，但是这两者在使用方式和功能特性上还是有差别的。

```default
对比项						Object Monitor Methods				Condition
------------------------------------------------------------------------------------------
前置条件					获取对象的锁					调用Lock.lock获取锁
													   调用Lock.newCondition()获取Conditio

调用方式					直接调用(obejct.wait())		  直接调用(condition.wait())
等待队列个数				   一个						  多个
当前线程释放锁并进入等待状态	  支持						 支持

当前线程释放锁并进入等待状态	  不支持						支持
在等待中不响应中断

当前线程释放锁并进入少时等待	  支持						 支持

当前线程释放锁进入等待状态到	  不支持						支持
将来的某个时间

唤醒等待队列中的一个线程		支持						  支持
唤醒等待队列中的全部线程		支持						  支持
------------------------------------------------------------------------------------------
```

Condition类定义了等待/通知两种类型的方法，当前线程调用这些方法时，需要提前获取到Condition对象关联的锁。Condition对象是由Lock对象（调用Lock对象的newCondition()方法）创建出来的，换句话说，Condition是依赖于Lock对象的。

简单使用示例：

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionTest {
	Lock lock = new ReentrantLock();
	Condition condition = lock.newCondition(); //一般会将Condition作为成员变量
	
	public static void main(String[] args) {
		
	}
	
	public void conditionWait() throws InterruptedException {
		lock.lock();
		try {
			condition.await(); //当前线程释放锁并在此等待
		} finally {
			lock.unlock();
		}
	}
	
	public void conditionSignal() {
		lock.lock();
		try {
			condition.signal(); //通知等待的某个线程，被通知的线程将从await方法返回
		} finally {
			lock.unlock();
		}
	}
}
```

Condition定义的方法及描述：

```java
//当前线程进入等待状态直到被通知（signal）或中断，当前线程进入运行状态且从await方法返回的情况包括：
//1.其他线程调用该Condition的signal()或者signalAll()方法，而当前线程被选中唤醒
//2.其他线程(调用interrupt()方法)中断当前线程
//如果当前线程从await方法返回，表明该线程已经获取了Condition对象所对应的锁
void await() throws InterruptedException
  
//当前线程进入等待状态直到被通知，但是该方法对中断不敏感
void awaitUninterruptibly()
  
//当前线程进入等待状态直到被通知、中断或者超时。返回值表示剩余的时间，如果在nanosTimeout纳秒之前被唤醒
//那么返回值就是(nanosTimeout-实际耗时)，如果返回值是0或负数，那么可以认定已经超时了
long awaitNanos(long nanosTimeout) throws InterruptedException
  
//当前线程进入等待状态直到被通知、中断或到某个时间。
//如果没有到指定时间就被通知则方法返回true，否则表示到了指定时间方法返回false
boolean awaitUntil(Date deadline) throws InterruptedException
  
//唤醒一个等待在Condition上的线程，该线程从等待方法返回前必须获得与Condition相关联的锁
void signal()
  
//唤醒所有等待在Condition上的线程，该线程从等待方法返回前必须获得与Condition相关联的锁
void signalAll()
```

<br>







### 阻塞队列

> 阻塞队列 (BlockingQueue)是 Java util.concurrent包下重要的数据结构，BlockingQueue提供了线程安全的队列访问方式：当阻塞队列进行插入数据时，如果队列已满，线程将会阻塞等待直到队列非满；从阻塞队列取数据时，如果队列已空，线程将会阻塞等待直到队列非空。并发包下很多高级同步类的实现都是基于BlockingQueue实现的。



#### 自己构建一个阻塞队列

有界阻塞队列是一种特殊的队列，当队列是空的时候，队列的获取操作将会阻塞获取线程，直到队列中有新增的元素；当队列已满时，队列的插入操作将会阻塞插入线程，直到出现新的“空位”，代码如下：

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class BoundedQueue<E> {
	private Object[] items;
	private int addIndex, removeIndex, count;
	private Lock lock = new ReentrantLock();
	private Condition notEmpty = lock.newCondition();
	private Condition notFull = lock.newCondition();
	
	public BoundedQueue(int size) {
		items = new Object[size];
	}
	
	public void add(E e) throws InterruptedException {
		lock.lock();
		try {
			while (count==items.length) {
				notFull.await();
			}
			items[addIndex] = e;
			if (++addIndex==items.length) {
				addIndex = 0;
			}
			++count;
			notEmpty.signal();
		} finally {
			lock.unlock();
		}
	}
	
	@SuppressWarnings("unchecked")
	public E remove() throws InterruptedException {
		lock.lock();
		try {
			while (count==0) {
				notEmpty.await();
			}
			Object x = items[removeIndex];
			if (++removeIndex==items.length) {
				removeIndex = 0;
			}
			--count;
			notFull.signal();
			return (E) x;
		} finally {
			lock.unlock();
		}
	}
}
```

<br>



### Condition的实现分析

> Condition的实现ConditionObject是同步器AbstractQueuedSynchronizer的内部类，因为Condition的操作需要获取相关联的锁，所以作为同步器的内部类也较为合理。每个Condition对象都包含着一个队列（以下简称为等待队列），该队列是Condition实现等待通知机制的关键。

#### 等待队列

等待队列也是一个FIFO的队列，在队列的每个节点都包含了一个线程的引用，该线程就是在Condition对象和是哪个等待的线程，如果一个线程调用了Condition.await()方法，那么这个线程就会释放锁、构成节点加入等待队列并进入等待状态。事实上，节点的定义复用了同步器中节点的定义，也就是说，同步队列和Condition的等待队列中节点的类型都是AbstractQueuedSynchronizer.Node。

在Object的监视器模型上，一个对象只有一个同步队列和等待队列，而并发包中的Lock（更确切地说是同步器）拥有一个同步队列和多个Condition等待队列，其对应关系如下：

```default
如图所示：
Condition的实现是同步器的内部类，因此每一个Condition实例都能访问同步器提供的方法，相当于每一个Condition都拥有所属同步器的引用：

|同步器|
|head |--------->|节点|			 |节点|			 |节点|			|节点|
|	  |		     |prev|<----------|prev|<----------|prev|<----------|prev|
|     |		     |next|---------->|next|---------->|next|---------->|next|
|	  |																   |
|tail |<---------------------------------------------------------------|
  |	|
  |	|
  |	|
  |	|_____
  |		 |Condition	 |
  |		 |firstWaiter|-------->|节点		|--------->|节点		|
  |		 |			 |         |nextWaiter|		    |nextWaiter|
  |		 |  		 |									|
  |		 |lastWaiter |<---------------------------------|
  |		 
  |		 
  |		 
  |	_____
  		 |Condition	 |
  		 |firstWaiter|-------->|节点		|--------->|节点		|
  		 |			 |         |nextWaiter|		    |nextWaiter|
  		 |  		 |									|
  		 |lastWaiter |<---------------------------------|	 
```

<br>

#### 等待

调用Condition的await()方法（或者以await开头的方法），会使当前线程进入等待队列并释放锁，同时线程状态变为等待状态。当从await方法返回时，当前线程一定是获取了Condition相关联的锁。从队列（同步队列和Condition等待队列）的角度来看await()方法，当调用await()方法，相当于同步队列的首节点（获取了锁的节点）移动到Condition的等待队列中。

CondiitionObject的await方法：

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
  	//当前线程加入等待队列
    Node node = addConditionWaiter();
  	//释放同步状态，也就是释放锁
    int savedState = fullyRelease(node);
    int interruptMode = 0;
  	//被唤醒后的线程将从isOnSyncQueue退出(isOnSyncQueue(node)返回true)
  	//进而调用同步器的acquireQueued()方法加入到获取同步状态的竞争中
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

调用该方法的线程是成功获取了锁的线程，也就是同步队列中的首节点，该方法会将当前线程构造成Condition等待节点，并将其加入到等待队列中（addConditionWaiter），然后释放同步状态（fullyRelease），唤醒同步队列中的后继结点，然后当前线程进入等待状态。

当等待队列中的节点被唤醒，则唤醒节点的线程开始尝试获取同步状态。如果不是通过其他线程调用Condition.signal()方法唤醒，而是等待线程进程进行中断，则会抛出InterruptedException。

<br>

#### 通知

调用Condition的signal()方法，将会唤醒在等待队列中等待时间最长的节点（首节点），在唤醒节点之前，会将节点移动到同步队列的尾部。Condition的signalAll方法，相当于等待队列中的每一个节点均执行了一次signal方法，效果就是将等待队列中所有的节点全部移动到同步队列中，并唤醒每个节点的线程。

```java
public final void signal() {
  	 //调用该方法的前置条件是当前线程必须获取了锁，可以看到signal进行了isHeldExclusively检查
  	 //也就是当前线程必须是获取了锁的线程
     if (!isHeldExclusively())
         throw new IllegalMonitorStateException();
     //获取Condition等待队列中的首节点
  	 Node first = firstWaiter;
     if (first != null)
       	 //将Condition等待队列中的首节点移动到同步队列的尾部，并使用LockSupport唤醒节点中的线程
         doSignal(first);
}
```

<br>



#### 阻塞队列接口

BlockingQueue 具有 4 组不同的方法用于插入、移除以及对队列中的元素进行检查。如果请求的操作不能得到立即执行的话，每个方法的表现也不同。这些方法如下：

```default
			抛异常				阻塞			特定值			超时
------------------------------------------------------------------------------------------
插入		   add(o)			put(o)		  offer(o)		offer(o, timeout, timeunit)	
移除		   remove(o)		take(o)		  poll(o)		poll(timeout, timeunit)
检查		   element(o)					  peek(o)
------------------------------------------------------------------------------------------

解释：
抛异常：如果试图的操作无法立即执行，抛一个异常。
阻塞：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。
特定值：如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。
超时：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是true / false)。
```

无法向一个 BlockingQueue 中插入 null。如果你试图插入 null，BlockingQueue 将会抛出一个 NullPointerException。可以访问到 BlockingQueue 中的所有元素，而不仅仅是开始和结束的元素。比如说，你将一个对象放入队列之中以等待处理，但你的应用想要将其取消掉。那么你可以调用诸如 remove(o) 方法来将队列之中的特定对象进行移除。但是这么干效率并不高(译者注：基于队列的数据结构，获取除开始或结束位置的其他对象的效率不会太高)，因此你尽量不要用这一类的方法，除非你确实不得不那么做。

**BlockingQueue 的实现类**

- **ArrayBlockingQueue**：ArrayBlockingQueue 是一个有界的阻塞队列，其内部实现是将对象放到一个数组里。有界也就意味着，它不能够存储无限多数量的元素。它有一个同一时间能够存储元素数量的上限。你可以在对其初始化的时候设定这个上限，但之后就无法对这个上限进行修改了(译者注：因为它是基于数组实现的，也就具有数组的特性：一旦初始化，大小就无法修改)。
- **DelayQueue**：DelayQueue 对元素进行持有直到一个特定的延迟到期。注入其中的元素必须实现 java.util.concurrent.Delayed 接口。
- **LinkedBlockingQueue**：LinkedBlockingQueue 内部以一个链式结构(链接节点)对其元素进行存储。如果需要的话，这一链式结构可以选择一个上限。如果没有定义上限，将使用 Integer.MAX_VALUE 作为上限。
- **PriorityBlockingQueue**：PriorityBlockingQueue 是一个无界的并发队列。它使用了和类 java.util.PriorityQueue 一样的排序规则。你无法向这个队列中插入 null 值。所有插入到 PriorityBlockingQueue 的元素必须实现 java.lang.Comparable 接口。因此该队列中元素的排序就取决于你自己的 Comparable 实现。
- **SynchronousQueue**：SynchronousQueue 是一个特殊的队列，它的内部同时只能够容纳单个元素。如果该队列已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。同样，如果该队列为空，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。据此，把这个类称作一个队列显然是夸大其词了。它更多像是一个汇合点。



### 简单的生产消费模型

```java
import java.util.Random;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

public class BlockingQueueTest {
    //生产者
    public static class Producer implements Runnable{
        private final BlockingQueue<Integer> blockingQueue;
        private volatile boolean flag;
        private Random random;

        public Producer(BlockingQueue<Integer> blockingQueue) {
            this.blockingQueue = blockingQueue;
            flag=false;
            random=new Random();

        }
        public void run() {
            while(!flag){
                int info=random.nextInt(100);
                try {
                    blockingQueue.put(info);
                    System.out.println(Thread.currentThread().getName()+" produce "+info);
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }               
            }
        }
        public void shutDown(){
            flag=true;
        }
    }
    //消费者
    public static class Consumer implements Runnable{
        private final BlockingQueue<Integer> blockingQueue;
        private volatile boolean flag;
        public Consumer(BlockingQueue<Integer> blockingQueue) {
            this.blockingQueue = blockingQueue;
        }
        public void run() {
            while(!flag){
                int info;
                try {
                    info = blockingQueue.take();
                    System.out.println(Thread.currentThread().getName()
                                       +" consumer "+info);
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }               
            }
        }
        public void shutDown(){
            flag=true;
        }
    }
    public static void main(String[] args){
        BlockingQueue<Integer> blockingQueue = new LinkedBlockingQueue<Integer>(10);
        Producer producer=new Producer(blockingQueue);
        Consumer consumer=new Consumer(blockingQueue);
        //创建5个生产者，5个消费者
        for(int i=0;i<10;i++){
            if(i<5){
                new Thread(producer,"producer"+i).start();
            }else{
                new Thread(consumer,"consumer"+(i-5)).start();
            }
        }

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        producer.shutDown();
        consumer.shutDown();

    }
}
```