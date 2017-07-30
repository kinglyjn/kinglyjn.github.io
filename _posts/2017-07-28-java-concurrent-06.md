---
layout: post
title:  "JAVA中的锁(一)"
desc: "JAVA中的锁(一)"
keywords: "JAVA中的锁(一),java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



### 信号机(Semaphore)和信号协调

以一个停车场是运作为例。为了简单起见，假设停车场只有三个车位，一开始三个车位都是空的。这时如果同时来了五辆车，看门人允许其中三辆不受阻碍的进入，然后放下车拦，剩下的车则必须在入口等待，此后来的车也都不得不在入口处等待。这时，有一辆车离开停车场，看门人得知后，打开车拦，放入一辆，如果又离开两辆，则又可以放入两辆，如此往复。

在这个停车场系统]中，车位是公共资源，每辆车好比一个线程，看门人起的就是信号量的作用。

更进一步，信号量的特性如下：信号量是一个非负整数（车位数），所有通过它的线程（车辆）都会将该整数减一（通过它当然是为了使用资源），当该整数值为零时，所有试图通过它的线程都将处于等待状态。在信号量上我们定义两种操作： Wait（等待） 和 Release（释放）。 当一个线程调用Wait（等待）操作时，它要么通过然后将信号量减一，要么一直等下去，直到信号量大于一或超时。Release（释放）实际上是在信号量上执行加操作，对应于车辆离开停车场，该操作之所以叫做“释放”是因为加操作实际上是释放了由信号量守护的资源。

在java中，还可以设置该信号量是否采用公平模式，如果以公平方式执行，则线程将会按到达的顺序（FIFO）执行，如果是非公平，则可以后请求的有可能排在队列的头部。

<br>



### Lock锁

锁是用来控制多个资源访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源（但是有些锁可以允许并发地访问共享资源，比如读写锁）。在 Lock接口出现以前，java是靠synchronized关键字来实现锁功能的，而在java SE 1.5以后，并发包中新增了Lock接口（以及相关实现类），它提供了与synchronized关键字类似的同步功能，只是在使用时需要显式地获取和释放锁，虽然他缺少了synchronized隐式获取释放锁的便捷性，但却拥有了锁获取与释放的可操作性、可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步通特性。Lock接口的实现基本都是通过聚合了一个同步器的子类来完成线程访问控制的。

Lock的使用也很简单，如下所示：

```java
public static void main(String[] args) {
	Lock lock = new ReentrantLock();
	lock.lock();
	try {
		// TODO
        // 不要将获取锁的操作写在try块中，因为如果在获取锁(自定义锁的实现)时发生了异常
        // 异常发生的同时，也会导致锁无辜释放
	} finally {
		lock.unlock(); //保证锁最终能够尽量释放
	}
}
```

Lock接口提供的synchronized关键字所不具备的主要特性：

* 尝试非阻塞地获取锁：当前线程A尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁
* 能被中断地获取锁：当获取到锁的线程被中断时，中断异常将会抛出，同时锁会被释放
* 超时获取锁：在指定的截止时间之前获取锁，如果截止时间到了仍没有获取到锁则返回

Lock接口的API：

```java
//获取锁，调用该方法当前线程会获取锁，获取到之后从该方法返回
void lock()

//可中断地获取锁，和lock方法不同的是该方法会响应中断，即在锁的获取中可以中断当前线程
void lockInterruptibly() throws InterruptedException

//尝试非阻塞的获取锁，调用该方法后立即返回，如果能够获取则返回true，否则false
boolean tryLock()

//超时地获取锁当前线程在以下3中情况下会返回：
//当前线程在超时时间内获取了锁
//当前线程在超时时间内被中断
//超时时间结束，返回false
boolean tryLock(long time, TimeUnit unit) throws InterruptedException

//释放锁
void unlock()

//获取等待通知组件，该组件和当前的锁绑定，当前线程只有获得了锁，才能调用该组件的wait方法，
//而调用之后当前线程则释放锁。
Condition newCondition()
```

<br>



### 队列同步器

队列同步器（AbstractQueuedSynchronizer），以下简同步器，是用来构建锁，或者是其他同步组件的基础框架，它使用了int成员滨量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。并发包的作者（Doug Lea）期望它能够称为实现大部分同步需求的基础。

同步器的主要使用方式是继承，子类通过继承同步器并实现它的抽象法法来管理同步状态，在抽象方法实现的过程中免不了要对同步的状态进行更改，这时候就要使用同步器提供的3个接口：

* getState()
* setState(int newState)
* compareAndSetState(int expect, int update)

因为他们能够保证状态的改变是安全的。子类推荐被定义为自定义同步组件的静态内部类，同步器自身并没有实现任何同步接口，他仅仅是定义了若干同步状态获取和释放的方法来供自定义同步组件来使用，同步器既可以支持独占式的获取同步状态，也可以支持共享式地获取同步状态，这样就可以方便的实现不同类型的同步组件（ReentrantLock、ReentrantReadWriteLock、CountDownLatch等）。

同步器是实现锁（也可以是任意同步组件）的关键，在锁的实现中聚合同步器，利用同步器实现锁的语义。这样可以理解二者之间的关系：锁是面向使用者的，它定义了使用者与锁交互的接口（比如他可以允许两个线程并行访问），隐藏了实现细节；同步器是面向锁的实现者的，他简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。锁和同步器很好的隔离了使用者和实现者锁关注的领域。

<br>

**队列同步器的接口与示例**

同步器的设计是基于模板方法模式的，也就是说，使用者需要继承同步器指定的方法，随后将同步器组合在自定义同步组件的实现中，并调用同步器提供的模板方法，而这些模板方法将会调用使用者重写的方法。

重写同步器指定的方法时，需要使用同步器提供的如下三个方法来访问或修改同步状态，即上面提到的getState、setState、compareAndSetState三个方法。

`同步器可重写的方法`：

* protected boolean tryAcquire(int arg)：独占式获取同步状态，实现该方法需要查询当前状态并判断同步状态状态是否符合预期，然后进行CAS设置同步状态；
* protected boolean tryRelease(int arg)：独占式释放同步状态，等待获取同步状态的线程将有机会获取同步状态；
* protected int tryAcquireShared(int arg)：共享式获取同步状态，发回值大于或等于0的值表示获取成功，反之获取失败；
* protected boolean tryReleaseShared(int arg)：共享式释放同步状态；
* protected boolean isHeldExclusively()：当前同步器是否在独占模式下被线程占用，一般该方法表示是否被当前线程所独占；



`同步器提供的模板方法`基本上分为三类：

* 独占式获取和释放同步状态  
  * void acquire(int arg)：独占式获取同步状态，如果当前线程获取同步状态成功则由该方法返回，否则将会进入同步队列中等待，该方法将会调用重写的tryAcquire(int arg)方法；
  * void acquireInterruptibly(int arg)：与acquire相同，但是该方法会响应中断，当前线程未获取到线程同步状态而进入同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException并返回；
  * boolean tryAcquireNanos(int arg, long nanos)：在acquireInterruptibly的基础上增加了超时限制，如果当前线程在超时时间内没有获取到同步状态，那么将会返回false，如果获取到了则返回true；
  * boolean release(int arg)：独占式的释放同步状态，该方法在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒；
* 共享式获取和释放同步状态  
  * void acquireShared(int arg)：共享式地获取同步状态，如果当前线程未获取到同步状态，将会进入到同步队列中等待，与独占式的主要区别在于同一个时刻可以有多个线程获取到同步状态；
  * void acquireSharedInterruptibly(int arg)：与acquireShared相同，该方法响应中断；
  * boolean tryAcquireSharedNanos(int arg, long nanos)：在acquireSharedInterruptibly 基础上增加了超时限制；
  * boolean releaseShared(int arg)：共享式地释放同步状态；
* 查询同步队列中的等待线程状况  
  * Collection<Thread> getQueuedThreads()：获取等待在同步队列上的线程集合；



只有掌握了同步器的工作原理才能更加深入地理解并发包中其他的并发组件，所以下面通过一个独占锁的示例来深入了解一下同步器的工作原理。顾名思义，独占锁就是在同一个时刻，只能有一个线程获取到锁，而其他获取锁的线程只能处于同步队列中等待，只有获取锁的线程释放的锁，后继的线程才能获取锁。代码示例如下：

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
public class Mutex implements Lock {
	/**
	 * 静态内部类：自定义同步器
	 */
	private static class Sync extends AbstractQueuedSynchronizer {
		private static final long serialVersionUID = 1L;

		@Override
		protected boolean tryAcquire(int arg) { //当状态为0的时候获取锁
			if (compareAndSetState(0, 1)) {
				setExclusiveOwnerThread(Thread.currentThread());
				return true;
			}
			return false;
		}

		@Override
		protected boolean tryRelease(int arg) { //释放锁并将同步状态设置为0
			if (getState()==0) {
				throw new IllegalMonitorStateException();
			}
			setExclusiveOwnerThread(null);
			setState(0);
			return true;
		}

		@Override
		protected boolean isHeldExclusively() { //是否处于独占状态
			return getState()==1;
		}
	}
	
	//仅需将操作代理到Sync上即可
	private final Sync sync = new Sync();
	
	
	@Override
	public void lock() {
		sync.acquire(1);
	}
	
	public boolean isLocked() {
		return sync.isHeldExclusively();
	}

	@Override
	public void lockInterruptibly() throws InterruptedException {
		sync.acquireInterruptibly(1);
	}

	@Override
	public boolean tryLock() {
		return sync.tryAcquire(1);
	}

	@Override
	public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
		return sync.tryAcquireNanos(1, unit.toNanos(timeout));
	}

	@Override
	public void unlock() {
		sync.release(1);
	}

	@Override
	public Condition newCondition() {
		return sync.new ConditionObject();
	}
	
	public boolean hasQueuedThreads() {
		return sync.hasQueuedThreads();
	}
}
```

上述示例中独占锁Mutex是一个自定义的同步组件，他是在同一个时刻只允许一个线程占有锁。Mutex中定义了一个静态内部类，该内部类继承了同步器并实现了同步获取和释放同步状态。在tryAcquire(int acquires)方法中，如果经过CAS设置成功（同步状态设置为1），则表示获取了同步状态，而在tryRelease(int releases)中只是将同步状态重置为0，用户使用Mutex时并不会直接和内部的同步器实现打交道，而是调用Mutex提供的方法。在Mutex的实现中，以获取lock()为例，只需要在方法实现中调用同步器的模板方法aquire(int args)即可，当前线程调用该方法获取同步状态失败后会被加入到同步队列中等待，这样就大大降低了实现一个可靠同步组件的门槛。

<br>



### 队列同步器的实现分析

接下来将从实现角度分析同步器是如何完成线程同步的，主要包括：同步队列、独占式同步状态获取与释放、共享式同步状态获取与释放、超时获同步状态等同步器的核心数据结构与模板方法。

**同步队列**

同步器依赖内部的同步队列（一个双向的FIFO队列）来完成同步状态的管理，当前线程获取同步状态失败时，同步器会将当前线程以及等待状态信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程。当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。

同步队列中的节点（Node）用来保存 获取同步状态失败的线程 的引用、等待状态、以及前驱和后继节点，节点的属性类型和属性名称以及描述如下所示：

```default
init waitStatus: 等待状态，包含如下状态：
	1.CANCELLED，值为1，由于在同步队列中等待的线程超时或者被中断，需要从同步队列中取消等待，节点进入该状
	  态讲不会发生变化
	2.SIGNAL，值为-1，后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知
	  后继节点，使后继节点的线程得以运行
	3.CONDITION，值为-2，节点在等待队列中，结点线程等待在 Condition 上，当其他线程对Condition调用了
	  signal()方法后，该节点将会从等待队列中转移到同步队列中，加入到对同步状态的获取中
	4.PROPAGATE，值为3，表示下一次共享式同步状态获取将会无条件的传播下去
	4.INITIAL，值为0，初始状态
```

下面是Node静态内部类的源码：

```java
static final class Node {  
        /** 标识一个这个节点是否为shared类型 */  
        static final Node SHARED = new Node();  
        /** Marker to indicate a node is waiting in exclusive mode */  
        static final Node EXCLUSIVE = null;  
  
        /** waitStatus value to indicate thread has cancelled */  
        static final int CANCELLED =  1;  
        /** waitStatus value to indicate successor's thread needs unparking */  
        static final int SIGNAL    = -1;  
        /** waitStatus value to indicate thread is waiting on condition */  
        static final int CONDITION = -2;  
        /** 
         * waitStatus value to indicate the next acquireShared should 
         * unconditionally propagate 
         */  
        static final int PROPAGATE = -3;  
  
        /** 
         *   等待状态值，又以下状态值: 
         * 
         *   SIGNAL:     值为-1 ，后续节点处于等待状态，而当前节点的线程如果 
         *               释放了同步状态或者取消等待，节点进入该状态不会变化           
         *   CANCELLED:  值为 1，由于在同步队列中等待的线程等待超时或者被中断 
         *               需要从同步队列中取消等待，节点进入该状态将不会变化                     
         *   CONDITION:  值为-2，节点在等待队列中，节点线程等待在Condition上， 
         *               当其他线程对Condition调用了signal()方法后，该节点将会 
         *               从等待队里中转移到同步队列中，加入对同步状态的获取中 
         
         *   PROPAGATE:  值为-3，表示下一次共享式同步状态获取将会无条件地被传播下去 
         *             
         *   0:          初始化状态 
         */  
        volatile int waitStatus;  
  
        /** 
         * 前驱节点，当节点加入同步队列时被设置 
         */  
        volatile Node prev;  
  
         /** 
         * 后继节点 
         */  
        volatile Node next;  
  
        /** 
         * 获取状态状态的线程 
         */  
        volatile Thread thread;  
  
         /** 
         * 等待队列中的后继节点。如果当前节点是共享的，那么这个字段是一个shared常量， 
         * 也就是说节点类型（独占或共享）和等待队列中个后继节点共用同一个字段 
         */  
        Node nextWaiter;  
  
        /** 
         * Returns true if node is waiting in shared mode. 
         */  
        final boolean isShared() {  
            return nextWaiter == SHARED;  
        }  
  
        /** 
         * Returns previous node, or throws NullPointerException if null. 
         * Use when predecessor cannot be null.  The null check could 
         * be elided, but is present to help the VM. 
         * 
         * @return the predecessor of this node 
         */  
        final Node predecessor() throws NullPointerException {  
            Node p = prev;  
            if (p == null)  
                throw new NullPointerException();  
            else  
                return p;  
        }  
  
        Node() {    // Used to establish initial head or SHARED marker  
        }  
  
        Node(Thread thread, Node mode) {     // Used by addWaiter  
            this.nextWaiter = mode;  
            this.thread = thread;  
        }  
  
        Node(Thread thread, int waitStatus) { // Used by Condition  
            this.waitStatus = waitStatus;  
            this.thread = thread;  
        }  
    } 
```

<br>



### 独占式同步状态的获取与释放

```java
//主要完成同步状态的获取、节点的构造、加入同步队列、以及在同步队列中进行自旋等待的相关工作
//其主要逻辑是：首先调用自定义同步器实现 tryAcquire(int arg) 的方法，该方法保证线程安全
//地获取同步状态，如果同步状态获取失败，则构造同步节点(独占式Node.EXCLUSIVE,同一时刻只能
//有一个线程成功获取同步状态)并通过 addWaiter(Node node)方法将该节点加入到同步队列的尾部，
//最后调用 acquireQueued(Node node, int arg) 方法，使得该节点以“死循环“的方式获取同步
//状态。如果获取不到则阻塞节点中的线程，而被阻塞线程的唤醒需要依靠前驱节点的出队 或者阻塞线程
//的中断来实现
public final void acquire(int arg) {
  	if (!tryAcquire(arg) &&
     	 acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    	selfInterrupt();
}


//compareAndSetTail(Node expect, Node update)保证了节点能够被线程安全的添加。试想一下，
//如果使用一个普通的LinkedList来维护节点之间的关系，那么当一个线程获取了同步状态，而其他线程
//由于调用 tryAcquire(int arg) 方法获取同步状态失败而被并发地添加到Linkedlist中时，
//LinkedList将难以保证Node的正确添加，最终的结果可能是节点的数量有偏差，而且顺序也是乱的
private Node addWaiter(Node mode) {
  	Node node = new Node(Thread.currentThread(), mode);
  	// Try the fast path of enq; backup to full enq on failure
  	Node pred = tail;
  	if (pred != null) {
    	node.prev = pred;
    	if (compareAndSetTail(pred, node)) {
      		pred.next = node;
      		return node;
    	}
  	}
  	enq(node);
  	return node;
}

//enq(final Node node)方法中，同步器通过”死循环”来保证节点的正确添加，在“死循环”中只有
//通过CAS将节点设置为尾节点之后，当前线程才能从该方法中返回，否则当前线程不断地尝试设置。可以
//看出，该方法将并发添加节点的请求通过CAS变得"串行化"了
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}

//节点进入到同步队列之后，就进入了一个自旋的过程，每个节点（或者说每个线程）都在自省地观察，
//当条件满足获取到同步状态时，就可以从这个自旋的过程中退出，否则依旧留在这个自旋的过程中（
//并且阻塞节点的线程），代码如下。在该方法中当前线程在“死循环”中尝试获取同步状态，而只有
//前驱节点是头结点才能尝试获取同步状态（这样保证了早获取早通知的逻辑思路）。
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }	
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

独占式同步状态获取流程如下图所示：

```default
获取同步状态-------失败-------->生成节点
	|							|
	|							|
	|							|/
	|						加入到同步队列尾部
    |                       	|
    |							|CAS设置
    |                        	|							 
	|						前驱是否为头结点------------><------|		
    |							|					  |	 	线程被中断或者前驱节点被释放
    |							|Y					线程进入等待状态
    |							|/					  |
    |						获取同步状态------失败------->
    |							|
    |							成功
    |							|
    |							|/
    |<-----------------------当前节点设置成头节点	
	|
	|/	
 退出返回
```

当同步状态获取成功以后，当前线程从acquire(int arg)方法返回，对于锁这种并发组件而言，代表着当前线程获取了锁。

当前线程获取同步状态并执行了相应的逻辑之后，就需要释放同步状态，使得后续节点能够继续获取同步状态。代码如下：

```java
//该方法释放了同步状态之后，会唤醒其后继节点(进而使得后继节点能够重新尝试获取同步状态)
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

分析了独占式同步状态的获取和释放之后，适当做个小结：

1. 在获取同步状态时，同步器维护一个同步队列，获取同步状态失败的线程都会被加入到队列中，并在队列中进行自旋；
2. 移出队列（或停止自旋）的条件是前驱节点成为头节点且成功获取了同步状态；
3. 在释放同步状态时，同步器调用了tryRelease(int arg)方法，唤醒头结点的后续节点，使得后续节点能够继续获取同步状态。

<br>



### 共享式同步状态的获取和释放

共享式获取与独占式获取最主要的差别在于同一时刻能否有多个线程同时获取到同步状态。以文件的读写为例，如果一个程序在对文件进行读操作，那么这一个时刻对于该文件的写操作均被阻塞，而读操作能够同时进行。写操作要求对资源的独占式访问，而读操作可以是共享式访问。

共享式访问资源时，其他共享式的访问均被允许，而独占式的访问被阻塞；独占式访问资源时，同一时刻其他访问均被阻塞。

通过调用同步器的acquireShared(int arg)方法可以共享式地获取同步状态，该方法代码如下：

```java

public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg); //获取同步状态失败后
}

private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg); //如果当前节点的前驱节点是头结点，尝试获取同步状态
                if (r >= 0) {				   //获取同步状态成功后
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

与独占式一样，共享式获取也需要释放同步状态，代码如下：

```java
//该方法在释放同步状态之后，将会唤醒后续处于等待状态的节点
//对于能够支持多个线程同时访问的并发组件(e.g. Semaphore)，在释放资源上它和独占式的主要区别在于
//tryReleaseShared(int arg)方法必须确保同步状态(或者资源数)线程被安全释放，一般是通过循环和CAS
//来保证的，因为释放同步状态的操作会来自于多个线程。
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

<br>



### 自定义一个同步组件-TwinsLock

设计一个同步工具，该工具在同一个时刻只允许至多两个线程同时访问，超过两个线程的访问将被阻塞，我们江浙个同步工具命名为TwinsLock。

* 首先确定其访问模式。TwinsLock能够在同一时刻支持多个线程的访问，这显然是共享式访问，因此需要使用同步器提供的acquireShared(int arg)等和Shared相关的方法，这就要求TwinsLock必须重写 tryAcquireShared(int arg)方法 和 tryReleaseShared(int arg)方法，这样才能保证同步器的共享式同步状态的获取与释放方法得以执行。
* 其次定义资源数。TwinsLock在同一个时刻只允许至多两个线程同时访问，表明同步资源数为2，这样可以设置初始状态status为2，当一个线程进行获取则status减1，该线程释放则status加1，状态的合法范围为2、1、0，其中0表示当前已经有两个线程获取了同步资源，此时再有其他线程对同步状态进行获取的话，该线程只能被移入到阻塞队列中去。在同步状态进行变更的时候，需要使用compareAndSet(int expect, int update)方法做原子性保障。
* 最后组合自定义同步器，一般情况下自定义同步器会被定义为自定义同步组件的内部类。

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

public class TwinsLock implements Lock {
	private final Sync sync = new Sync(2);
	
	private static final class Sync extends AbstractQueuedSynchronizer {
		private static final long serialVersionUID = 1L;

		Sync(int count) {
			if (count < 0) {
				throw new IllegalArgumentException("count must large than 0");
			}
			setState(count);
		}

		//返回值大于0才表示当前线程才获取到了同步状态
		@Override 
		protected int tryAcquireShared(int reduceCount) {
			for (;;) {
				int current = getState();
				int newCount = current-reduceCount;
				if (newCount<0 || compareAndSetState(current, newCount)) { //CAS确保正确设置
					return newCount;
				}
			}
		}

		@Override
		protected boolean tryReleaseShared(int returnCount) {
			for(;;) {
				int current = getState();
				int newCount = current + returnCount;
				if (compareAndSetState(current, newCount)) { //CAS确保正确设置
					return true;
				}
			}
		}
	}
	
	/**
	 * 获取锁
	 */
	@Override
	public void lock() {
		sync.acquireShared(1);
	}

	/**
	 * 释放锁
	 */
	@Override
	public void unlock() {
		sync.releaseShared(1);
	}
	
	@Override
	public void lockInterruptibly() throws InterruptedException {
		// TODO
	}

	@Override
	public boolean tryLock() {
		// TODO
		return false;
	}

	@Override
	public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
		// TODO
		return false;
	}

	@Override
	public Condition newCondition() {
		// TODO 
		return null;
	}
}
```

下面编写一个测试类来验证TwinsLock是否能按照预期来工作。在测试类中，定义了工作者线程Worker，该线程在执行过程中获取锁，当获取锁之后使当前线程睡1s（并不释放锁），随后打印当前线程名称，最后再次睡眠1s并释放锁，测试代码如下：

```java
public class TwinsLockTest {
	public static void main(String[] args) throws InterruptedException {
		final Lock lock = new TwinsLock();
		class Worker implements Runnable {
			@Override
			public void run() {
				lock.lock();
				try {
					sleep(1);
					System.out.println(Thread.currentThread().getName());
					sleep(1);
				} finally {
					lock.unlock();
				}
			}
		}
		
		//启动10个线程
		for (int i = 0; i < 10; i++) {
			Worker worker = new Worker();
			Thread thread = new Thread(worker, "Thread-"+i);
			thread.setDaemon(true);
			thread.start();
		}
		sleep(10);
	}
	
	public static void sleep(long seconds) {
		try {
			Thread.sleep(seconds*1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

//运行结果
//运行测试可以看到线程名称成对输出，也就是在同一个时刻只有两个线程能够获取到锁，
//这表明TwinsLock可以按照预期正常工作。
```





