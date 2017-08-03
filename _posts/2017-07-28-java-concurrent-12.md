---
layout: post
title:  "JAVA并发容器和框架-Fork/Join框架"
desc: "JAVA并发容器和框架-Fork/Join框架"
keywords: "JAVA并发容器和框架Fork/Join框架,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



> Fork/Join框架是java7提供的一个用于并行执行任务的框架，是一个把大部分任务分割成若干小任务，最终汇总每个小任务结果后得到大任务结果的框架。我们再通过Fork、Join这两个词来理解Fork/Join框架。Fork就是把一个大任务切分成若干子任务并行地执行，join就是合并这些子任务的结果，最后得到这个任务的结果。比如计算1+2+3...+10000，可以分割成十个子任务，每个子任务分别对1000个数进行求和，最终汇总这10个子任务的结果。



### 工作窃取算法

工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。那么，为什么需要使用工作窃取算法呢？假如我们需要做一个比较大的任务，为了减少线程间的竞争，把这些子任务分别放到不同的队列里，并为每一个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应。比如A线程负责处理A队列里的任务。但是有的线程会先把自己队列里的任务做完，而其他线程对应的队列里还有任务等待处理。干完活的线程预期等待着，不如去帮助其他线程去干活，于是他就去其他线程对应的队列里窃取一个任务来执行。而在这时候他们会访问同一个队列，所以为了减少窃取任务的线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务去执行。

工作窃取算法的优点是：充分利用线程进行并行计算，减少线程间的竞争。

工作窃取算法的缺点是：在某些情况下还是存在竞争，比如双端队列里只有一刻任务时。并且该算法会消耗更多的系统资源。比如创建多个线程和双端队列。

<br>



### Fork/Join框架的设计

我们已经很清楚Fork/Join框架的需求了，那么可以思考一下，如果让我们设计一个Fork/join框架，该如何设计？这个思考有助于你理解Fork/join框架的设计。

* 步骤一：首先我们需要有一个Fork类吧大任务分割成若干子任务，有可能子任务还是很大，所以还需要不停地进行分割，直到分割的任务足够小。
* 步骤二：执行任务并合并结果。执行的任务分放在双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一爱一个队列中，启动一个线程从队列里拿数据，然后合并这些数据。

Fork/join框架有两个类来完成以上工作：

* ForkJoinTask：我们使用ForkJoinTask创建一个ForkJoin任务，他提供在任务中执行fork和join操作的机制，通常情况下我们只需要继承ForkJoinTask的子类即可，ForkJoinTask提供了两个子类：
  * RecusiveAction：用于没有返回结果的任务
  * RecusiveTask：用于又返回结果的任务
* ForkJoinPool：ForkJoinTask需要ForkJoinPool来执行

任务分割出来的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里没有任务时，他会随机从其他工线程的队列的尾部获取一个任务。



### 使用Fork/Join框架

让我们通过一个简单的需求来使用Fork/Join框架，需求是：计算1+2+3+4的结果。

使用Fork/Join框架首先要考虑到的是如何分割任务，如果我们希望每个子任务最多执行两个数的相加，那么我们设置分割的阈值是2，由于是4个数字相加，所以Fork／Join框架会把这个任务fork成两个子任务，子任务一负责计算1+2，子任务二负责计算3+4，然后再join两个子任务的结果。因为是有结果的任务，所以必须继承RecursiveTask，实现代码如下：

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

public class CountTask extends RecursiveTask<Integer> {
	private static final long serialVersionUID = 1L;
	private static final int THRESHOLD = 2; //阈值
	
	private int start;
	private int end;
	public CountTask(int start, int end) {
		super();
		this.start = start;
		this.end = end;
	}

	@Override
	protected Integer compute() {
		int sum = 0;
		boolean canCompute = (end-start) <= THRESHOLD;
		if (canCompute) {
			for (int i = start; i <=end; i++) {
				sum += i;
			}
		} else {
			//如果任务大于阈值，就分裂成哥子任务来计算
			int middle = (start+end)/2;
			CountTask leftTask = new CountTask(start, middle);
			CountTask rightTask = new CountTask(middle+1, end);
			//拆分成子任务
			leftTask.fork();
			rightTask.fork();
			//等到子任务执行完，并得到器结果
			Integer leftResult = leftTask.join();
			Integer rightResult = rightTask.join();
			//合并子任务
			sum = leftResult + rightResult;
		}
		return sum;
	}
	
	
	public static void main(String[] args) {
		ForkJoinPool forkJoinPool = new ForkJoinPool();
		
		//生成一个计算任务，负责计算1+2+3+4
		CountTask task = new CountTask(1, 4);
		//执行任务
		Future<Integer> resultFuture = forkJoinPool.submit(task);
		try {
			Integer result = resultFuture.get();
			System.out.println("result=" + result);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}


//运行结果
result=10
```

通过这个例子，我们进一步了解ForkJoinTask，ForkJoinTask与一般任务的主要区别在于它需要实现compute方法，在这个方法里，首先需要判断任务是否足够小，如果足够小就直接执行任务；如果不足够小，就必须分割成两个子任务，每个子任务在调用fork方法时，又会进入compute方法，看看当期啊任务是否需要再分割成子任务，如果不需要继续分割，则执行当前任务并返回结果。使用join方法会等待子任务执行完并返回结果。

<br>



### Fork/Join框架的异常处理 

ForkJoinTask在执行的时候有可能会抛出异常，但是我们没有办法在主线程中直接不获取异常，所以ForkJoinTask还提供了isCompletedAbnormally()方法来检查任务是否已经抛出异常或已经被取消了，并且可以通过ForkJoinTask的getException方法来获取异常。使用代码如下：

```java
if (task.isCompletedAbnormally()) {
	System.out.println(task.getException());
}
```

getException()返回Throwable对象，如果任务被取消了则返回CancellationException。如果任务没有完成或者没有抛出异常则返回null。