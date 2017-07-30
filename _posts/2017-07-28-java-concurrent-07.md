---
layout: post
title:  "JAVA中的锁(二)-重入锁"
desc: "JAVA中的锁(二)-重入锁"
keywords: "JAVA中的锁(二)-重入锁,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



### 重入锁

重入锁（ReentrantLock），顾名思义，就是支持重进入的锁，它表示该锁能够支持对资源的重复加锁。除此之外，该锁还支持获取锁时的公平和非公平性选择。

回忆上篇的示例Mutex，同时考虑如下场景，当一个线程调用Mutex的lock方法获取锁之后，如果再次调用lock方法，则线程将会被自己所阻塞，原因是Mutex在实现tryAcquire(int acquires)方法时没有考虑占有锁的线程再次获取锁的场景，而在调用tryAcquire(int acquires)时返回了false，导致该线程被一直阻塞。简单的说，Mutex是一个不支持重新进入的锁。而 synchronized关键字隐式地支持重进入，比如一个synchronized修饰的递归方法，在方法执行的时候，执行线程在获取了锁之后仍能够连续多次获取该锁。

ReentrantLock虽然没能像synchronized关键字一样支持隐式地重进入，但是在调用lock()方法时，已经获取到锁的线程，能够再次调用lock方法获取锁而不会被阻塞。

> 这里提到一个锁获取的公平性问题，如果在绝对时间上，先对锁进行获取的请求一定先被满足，那么这个锁就是公平的，反之是不公平的。公平地获取锁，也就是等待时间最长的线程最优先获取锁，也可以说锁的获取是顺序的。ReentrantLock提供了一个构造函数，能够控制锁是否是公平的。事实上，公平的锁机制往往没有非公平的效率高，但是并不是任何场景都是以TPS（transactions per secons，类似于QPS，queries per secons）作为唯一的指标，公平锁能够减少“饥饿”发生的概率，等待越久的线程越是能够优先得到满足。

下面将着重分析ReentrantLock是如何实现 重进入 和 公平性获取锁的特性，并通过测试来验证公平性获取锁对性能的影响。



#### 实现重进入

实现重进入需要解决一下两个问题：

* 线程再次获取到锁，锁需要去识别获取锁的线程是否为当前占据锁的线程，如果是则再次成功获取。
* 锁的最终释放，线程重复n次获取了锁，随后在第n次释放该锁之后，其他线程能够获取到该锁。锁的最终释放要求锁对于获取进行计数自增，计数表示当前锁被重复获取的次数，而锁被释放时，计数自减，当计数等于0时表示锁已经成功释放。

ReentrantLock是通过组合自定义同步器来实现锁的获取与释放，以非公平（默认的）实现为例，代码如下：

```java
//该方法增加了再次获取同步状态的处理逻辑：通过判断当前线程是否为获取锁的线程来决定获取操作是否成功
//如果是获取锁的线程再次请求，则将同步状态值进行增加并返回true，表示获取同步状态成功
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) { ////
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

成功获取锁的线程再次获取锁，只是增加了同步状态值，这也就要求ReentrantLock在释放同步状态时需要减少同步状态值，代码如下：

```java
//如果该锁被获取了n次，那么前(n-1)次tryRelease方法必须返回false，而只有同步状态完全释放了，才能返回true
//可以看到，该方法将同步状态是否为0作为最终的释放条件，返回true表示释放成功
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

<br>



#### 公平和非公平获取锁的区别

对于非公平锁，只要CAS设置同步状态成功，则表示当前线程获取了锁。而公平锁则不同，代码如下:

```java
/**
 * Sync object for fair locks
 */
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    //该方法与notifairTryAcquire(int acquires)比较，唯一不同的位置为判断条件多了
  	//hasQueuedPredecessors()方法，即加入了同步队列中当前节点是否有前驱节点的判断
  	//如果该方法返回true表示有线程比当前线程更早地请求获取锁，因此需要等待当前前驱线程
  	//获取并释放锁之后才能继续获取锁
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

下面我们来编写一个测试来观察公平和非公平锁在获取锁时候的区别：

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import org.junit.Test;

public class FairAndUnfairTest {
	private static class ReentrantLock2 extends ReentrantLock {
        private static final long serialVersionUID = -6736727496956351588L;

        public ReentrantLock2(boolean fair) {
            super(fair);
        }

        public Collection<Thread> getQueuedThreads() {
            List<Thread> arrayList = new ArrayList<Thread>();
            Collections.reverse(arrayList);
            return arrayList;
        }
    }
	private static class Job extends Thread {
        private Lock lock;

        public Job(Lock lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            try {
                start.await();
            } catch (InterruptedException e) {
            }
            for (int i = 0; i < 2; i++) {
                lock.lock();
                try {
                    System.out.println("Lock by [" 
                                       + getName() + "], Waiting by " 
                                       + ((ReentrantLock2) lock).getQueuedThreads());
                } finally {
                    lock.unlock();
                }
            }
        }
        public String toString() {
            return getName();
        }
    }
	
    private static Lock fairLock = new ReentrantLock2(true);
    private static Lock unfairLock = new ReentrantLock2(false);
    private static CountDownLatch start;

    @Test
    public void fair() {
        testLock(fairLock);
    }
    @Test
    public void unfair() {
        testLock(unfairLock);
    }
    private void testLock(Lock lock) {
        start = new CountDownLatch(1);
        for (int i = 0; i < 5; i++) {
            Thread thread = new Job(lock);
            thread.setName("" + i);
            thread.start();
        }
        start.countDown();
    }
}

//运行结果
//fair方法输出
Lock by [4], Waiting by []
Lock by [1], Waiting by [0, 2, 3, 4]
Lock by [0], Waiting by [2, 3, 4, 1]
Lock by [2], Waiting by [3, 4, 1, 0]
Lock by [3], Waiting by [4, 1, 0, 2]
Lock by [4], Waiting by [1, 0, 2, 3]
Lock by [1], Waiting by [0, 2, 3]
Lock by [0], Waiting by [2, 3]
Lock by [2], Waiting by [3]
Lock by [3], Waiting by []
  
//unfair方法输出
Lock by [4], Waiting by [0, 1, 2, 3]
Lock by [4], Waiting by [0, 1, 2, 3]
Lock by [0], Waiting by [1, 2, 3]
Lock by [0], Waiting by [1, 2, 3]
Lock by [1], Waiting by [2, 3]
Lock by [1], Waiting by [2, 3]
Lock by [2], Waiting by [3]
Lock by [2], Waiting by [3]
Lock by [3], Waiting by []
Lock by [3], Waiting by []
```

观察输出的结果我们看到，公平性锁每次都是从同步队列中的第一个节点获取到锁，而非公平性锁出现了一个线程连续获取锁的情况。为什么会出现线程连续获取锁的情况呢？回顾notifairTryAcquired(int acquires)方法，当一个线程请求非公平性重入锁时，只要获取了同步状态即能成功获取锁，在这个前提下，刚释放锁的线程再次获取同步状态的几率非常大，使得其他线程只能在同步队列中等待。

非公平性锁可能使线程变得“饥饿”，为什么它又被设定成默认的实现呢？再次观察上面的结果，如果把每次不同线程获取到锁定义为一次切换，公平性锁在测试中进行了10次切换，而非公平性锁只有5次切换，这说明非公平性锁的开销更小。下面运行测试用例（测试环境ubuntu-14.04 i5-34708GB，测试场景：10个线程，每个线程获取100000次锁），通过vmstat统计测试运行时系统上下文切换的次数，运行结果如下：

```default
对比项						Fair						Unfair
--------------------------------------------------------------------------------
切换次数(每秒间隔)			187							159
						 40163						 330
                         350577						 14390
                         348637						 159
                         349682
                         349994
                         354223
                         211737
                         183
--------------------------------------------------------------------------------
总耗时(毫秒)				 5754						  61				
```

在这个测试结果中，公平锁总耗时是非公平锁的94.3倍，总切换次数是其133倍。可以看出公平锁保证了锁的获取按照FIFO原则，而代价是进行大量的线程切换。非公平锁虽然可能造成线程饥饿，但极少的线程切换，保证了其强大的吞吐量。

