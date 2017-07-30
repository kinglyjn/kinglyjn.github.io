---
layout: post
title:  "JAVA中的锁（三）-读写锁"
desc: "JAVA中的锁（三）-读写锁"
keywords: "JAVA中的锁（三）-读写锁,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---

### 读写锁

读写锁在同一时刻可以允许多个读线程访问，但是在写线程访问时，所有的度线程和写线程均被阻塞。读写锁维护了一对锁，一个读锁和一个写锁，通过分离读锁和写锁，使得并发性相比其他锁有了很大提升。

除了保证写操作对读操作的可见性以及并发性的提升以外，读写锁能够简化读写交互场景的编程方式。在没有写锁支持的时候，如果需要完成预想的读写工作，就要使用到java的等待通知机制，就是当写操作开始时，所有晚于写操作的读操作均会进入等待线程，只有写操作完成并进行通知以后，所有等待的读操作才能继续执行（写操作之间依靠synchronized关键字进行同步），这样做的目的是使读操作能读取到正确的数据，不会出现脏读。改用读写锁实现上述功能，只需要在读操作时获取读锁，在写操作时获取写锁即可。当写锁被获取到时，后续的（非当前写线程的）读写操作都会被阻塞，写锁释放之后，所有操作继续执行，编程方式相对于使用等待通知的实现机制而言变得更加简单明了。

一般情况下，读写锁的性能会比排它锁好，因为大多数场景下读是多于写的，在读多于写的情况下，读写锁能够提供必排它所更好的并发性和吞吐量。java并发包提供的排它锁的实现是ReentrantReadWriteLock，它提供的特性如下：

* 公平性选择：支持非公平（默认）和公平的锁获取方式，吞吐量还是非公平由于公平；
* 重进入：该锁支持重进入，以读写线程为例，度线程在获取了读锁之后，能够再次获取读锁；而写线程在获取了写锁之后能够再次获取写锁，同时也可以获取读锁；
* 锁降级：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级称为读锁；

<br>

ReadWriteLock仅定义了 readLock() 和 writeLock()方法，而其实现 ReentrantReadWriteLock，除了接口方法外，还提供了一些便于外界监控其内部工作状态的方法：

```java
//返回当前读锁被获取的次数。该次数不等于获取读锁的线程数。
//例如仅一个线程，它连续获取(重进入)了n次读锁，那么占据读锁的线程数是1，但该方法返回n
int getReadLockCount()
  
//返回当前线程获取读锁的次数。该方法在java6中加入到ReentrantReadWriteLock中，
//使用ThreadLocal保存当前线程获取的次数，这也使得java6变得更加复杂
int getReadHoldCount()
 
//返回当前写锁被获取的次数
int getWriteHoldCount()  
  
//判断写锁是否被获取
int isWriteLocked()
```

接下来通过一个缓存示例来说明读写锁的使用方式：

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class Cache {
	static Map<String, Object> map = new HashMap<>();
	static ReentrantReadWriteLock rw1 = new ReentrantReadWriteLock();
	static Lock r = rw1.readLock();
	static Lock w = rw1.writeLock();

	//获取一个key对应的value
	public static final Object get(String key) {
		r.lock();
		try {
			return map.get(key);
		} finally {
			r.unlock();
		}
	}
	
	//设置key对应的value，并返回就的value
	public static final Object put(String key, Object value) {
		w.lock();
		try {
			return map.put(key, value);
		} finally {
			w.unlock();
		}
	}
	
	//清空所有的内容
	public static final void clear() {
		w.lock();
		try {
			map.clear();
		} finally {
			w.unlock();
		}
	}
}
```

在上述示例中，Cache组合一个非线程安全的HashMap作为缓存的实现，同时使用读写锁的读锁和写锁来保证Cache是线程安全的。在读操作get(key)方法中，需要获取读锁，这使得并发访问该方法时不会被阻塞，写操作put(key, value)方法和clear()方法，在更新map时必须提前获取写锁，当获取写锁后，其他线程对于读锁和写锁的获取均被阻塞，而只有写锁释放以后，其他的读写操作才能继续。Cache使用读写锁来提升读操作的并发性，也保证每次写操作对所有读操作的可见性，同时简化了编程方式。

<br>



### 读写锁的实现分析

主要内容包括：读写状态的设计、写锁的获取与释放、读锁的获取与释放、锁降级。

读写锁同样依赖自定义的同步器来实现同步功能，而读写状态就是其同步状态。回想ReentrantLock中自定义同步器的实现，同步状态就表示锁被一个线程重复获取的次数，而读写锁的自定义同步器需要在同步状态（一个整型变量）上维护多个读线程和一个写线程的状态，使得该状态的设计成为读写所实现的关键。

如果在一个整型变量上维护多种状态，就一定要“按位切割使用”这个变量，读写锁，将变量切分成了两部分：高16位表示读，低16位表示写，划分方式如下所示：

```shell
0000000000000010 0000000000000011
---------------- ----------------
高16位  读状态=2		低16位 写状态=3
```

当前同步状态表示一个线程已经获取了写锁，且重进入了两次同时也连续获取了2次读锁。读写锁通过位运算迅速确定其读状态和写状态。假设当前的同步状态值为S，那么写状态就等于 S&0x0000ffff（高16位全部抹去），读状态就等于 S>>>16（无符号补0右移16位）。当写状态增加1时，读写状态等于 S+1，当读状态增加1时，读写状态等于 S+(1<<16)，也就是 S+0x00010000。

根据读写状态的划分得出一个推论：

```default
当S不等于0时而写状态（S&0x0000ffff）等于0时，则读状态（S>>>16）大于0，即读锁已被获取。
```

<br>

**写锁的获取与释放**

写锁是一个支持可重入的排他锁，如果当前线程获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程不是已经获取写锁的线程，则当前线程进入等待状态，获取写锁的代码如下：

```java
protected final boolean tryAcquire(int acquires) {
    /*
     * Walkthrough:
     * 1. If read count nonzero or write count nonzero
     *    and owner is a different thread, fail.
     * 2. If count would saturate, fail. (This can only
     *    happen if count is already nonzero.)
     * 3. Otherwise, this thread is eligible for lock if
     *    it is either a reentrant acquire or
     *    queue policy allows it. If so, update state
     *    and set owner.
     */
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

该方法除了重入条件（当前线程未获取了写锁的线程），增加了一个读锁是否存在的判断。如果存在读锁，则写锁不能被获取，原因在于读写所要确保写锁的操作对读锁可见，如果允许读锁在已经被获取的情况下对写锁的获取，那么正在运行的其他线程就无法感知到当前写线程的操作，因此只有等待其他读线程都释放了读锁，写锁才能被当前线程获取，而写锁一旦被获取，则其他读写线程的后续访问均被阻塞。

写锁的释放与ReentrantLock的释放过程基本类似，每次释放均减少写状态，当写状态为0的时候表示写锁已经被释放，从而等待的读写线程能够继续访问读写锁，同时前一次写线程的修改对后续的读写线程可见。

<br>

**读锁的获取与释放**

读锁是一个支持可重入的共享锁，它能够被多个线同时获取，在没有其他线程访问（或者写状态为0）时，读锁总会被成功的获取，而所做的只是（线程安全地）增加读状态。如果当前线程已经获取了读锁，则增加读状态。如果当前线程在获取读锁时，写锁已经被其他线程获取，则进入等待状态。读状态时所有线程获取读锁次数的总和，而每个线程各自获取读状态的次数只能选择保存在ThreadLocal中，由线程自身来维护，这使得获取读锁的实现变得更加复杂。因此这里讲获取读锁的代码做了删减，仅保留必要部分：

```java
protected final int tryAcquireShared(int unused) {
  	for(;;) {
      	int c = getState();
      	int nextc = c + (1<<16);
      	if (nextc < c) {
          	throw new Error("Maximum lock count exceeded");
      	}
      	if (exclusiveCount(c)!=0 && owner!=Thread.currentThread()) {
          	return -1;
      	}
      	if (compareAndSetState(c, nextc)) {
          	return 1;
      	}
  	}
}
```

读锁的每次释放（线程安全的，可能有多个读线程同时释放锁）均减少读状态，减少的值是（1<<16）。

<br>



### 锁降级

锁降级就是写锁降级称为读锁，如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指 把持住（当前拥有的）写锁，在获取到读锁，随后释放（先前拥有的）写锁的过程。

接下来看一个锁降级的示例。因为数据不常变化，所以多个线程可以并发地进行数据的处理，当数据变更后，如果当前线程感知到数据的变化，则进行数据的准备工作，同时其他处理线程被阻塞，直到当前线程完成数据的准备工作。

```java
public void processData() {
	readLock.lock();
	if (!update) {
		//必须先释放读锁
		readLock.unlock();
		
		//锁降级从写锁获取到开始
		writeLock.lock();
		try {
			if (!update) {
				//准备数据的流程（略）
				// TODO
				update = true;
			}
			readLock.lock();
		} finally {
			writeLock.unlock();
		}
	}
	
	try {
		//使用数据的流程（略）
		// TODO
	} finally {
		readLock.unlock();
	}
}
```

上述示例中，数据发生变更后，update变量（volatile boolean类型）被设置为false，此时所有访问processData方法的线程都能够感知到变化，但是只有一个线程能获取到写锁，，其他线程会被阻塞在读锁和写锁的lock方法上。当前线程获取写锁完成数据准备工作后，再获取读锁，随后释放写锁，完成锁降级。

锁降级中读锁的获取是否是必要的呢？答案是必要的，主要是为了保证数据的可见性，如果当前线程不获取读锁而是直接释放写锁，假设此刻另一个线程（记为T）获取了写锁并修改了数据，那么当前线程是无法根治线程T的数据更新的。如果当前线程获取读锁，即遵循锁降级的步骤，那么线程T将会被阻塞，直到当前线程使用数据并释放读锁之后，线程T才能获取写锁，进行数据的更新操作。

ReentrantReadWriteLock不支持锁升级（把持读锁，获取写锁，最后释放读锁的过程）。目的也是为了保证数据的可见性，如果读锁已被多个线程获取，其中任意一个线程成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的。



