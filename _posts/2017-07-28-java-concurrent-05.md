---
layout: post
title:  "并发编程之连接池的设计"
desc: "并发编程之连接池的设计"
keywords: "并发编程之连接池的设计,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



### 一个简单的数据库连接池示例

我们使用等待超时模式构造一个简单的数据库连接池，在示例中模拟从连接池中获取、使用、和释放连接的过程，而客户端获取连接的过程被设定为等待超时模式，也就是在1000ms每如果无法获取到可用连接，将会给客户端返回一个null。设定连接池的大小为10，然后调用客户端的线程数来模拟无法获取连接的场景。

首先看一下连接池的定义。它通过构造函数初始化连接数的最大上限，通过一个双向队列来维护连接，调用方先调用fetchConnection(long)方法来指定在多少毫秒内超时获取连接，当连接使用完成后，需要调用releaseConnection(Connection)来将连接放回到连接池中。

示例代码如下：

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.sql.Connection;
import java.util.LinkedList;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class ConnectionPoolTest {
	//线程池
	private static final ConnectionPool pool = new ConnectionPool(3);
	//保证所有的ConnectionRunner线程能够同时开始
	private static CountDownLatch startCountDownLatch = new CountDownLatch(1); 
	//保证所有的ConnectionRunner线程都运行结束后才继续执行
	private static CountDownLatch endCountDownLatch;
	
	/**
	 * 模拟客户端获取、使用、最后释放数据库连接的过程
	 */
	public static void main(String[] args) throws InterruptedException {
		//定义线程的数量，可以修改进行观察
		int threadCount = 10;
		//定义每个线程获取连接的次数
		int count = 20;
		AtomicInteger got = new AtomicInteger();
		AtomicInteger notGot = new AtomicInteger();
		endCountDownLatch = new CountDownLatch(threadCount);
		for (int i = 0; i < threadCount; i++) {
			new Thread(new ConnectionRunner(count, got, notGot),
                       "ConnectionRunner"+i).start();
		}
		startCountDownLatch.countDown();
		
		endCountDownLatch.await();
		System.out.println("total invoke: " + (threadCount*count));
		System.out.println("got connection: " + got);
		System.out.println("not got connection: " + notGot);
	}
	
	static class ConnectionRunner implements Runnable {
		int count;
		AtomicInteger got;
		AtomicInteger notGot;
		
		public ConnectionRunner(int count, AtomicInteger got, AtomicInteger notGot) {
			this.count = count;
			this.got = got;
			this.notGot = notGot;
		}

		@Override
		public void run() {
			try {
				startCountDownLatch.await();
			} catch (Exception e) {
			}
			
			while (count > 0) {
				try {
					//从线程池获取连接，如果1000ms内无法获取到，将会返回null
					//分别统计获取到的次数和未获取到的次数got、notGot
					Connection connection = pool.fetchConnection(1000);
					if (connection != null) {
						try {
							connection.createStatement();
							connection.commit();
						} finally {
							pool.releaseConnection(connection);
							got.incrementAndGet();
						}
					} else {
						notGot.incrementAndGet();
					}
				} catch (Exception e) {
					e.printStackTrace();
				} finally {
					count--;
				}
			}
			endCountDownLatch.countDown();
		}
	}
}


class ConnectionPool {
	private LinkedList<Connection> pool = new LinkedList<>();
	public ConnectionPool(int initialSize) {
		if (initialSize > 0) {
			for (int i = 0; i < initialSize; i++) {
				pool.addLast(ConnectionDriver.createConnection());
			}
		}
	}
	
	public void releaseConnection(Connection connection) {
		if (connection != null) {
			synchronized (pool) {
				//连接释放后需要进行通知，这样其他消费者能够感知到连接池中已经归还了一个连接
				pool.addLast(connection);
				pool.notifyAll();
			}
		}
	}
	
	/**
	 * 在mills时间内无法获取连接，将会返回null
	 */
	public Connection fetchConnection(long mills) throws InterruptedException {
		synchronized (pool) {
			if (mills < 0) { //超时
				while (pool.isEmpty()) {
					pool.wait();
				}
				return pool.removeFirst();
			} else {
				long future = System.currentTimeMillis();
				long remaining = mills;
				while (remaining>0 && pool.isEmpty()) {
					pool.wait(remaining);
					remaining = future - System.currentTimeMillis();
				}
				Connection result = null;
				if (!pool.isEmpty()) {
					result = pool.removeFirst();
				}
				return result;
			}
		}
	}
}

class ConnectionDriver {
	//由于java.sql.Connection是一个接口，最终的实现需要数据库驱动提供方来提供
	//考虑到这只是一个示例，我们通过动态代理构造了一个Connection对象，
	//该connection代理实现仅仅是在commit方法调用休眠100ms
	public static Connection createConnection() {
		return (Connection) Proxy.newProxyInstance(
          ConnectionDriver.class.getClassLoader(), new Class[]{Connection.class}, 
                                                   new InvocationHandler() {
			@Override
			public Object invoke(Object proxy, Method method, Object[] args) 
              throws Throwable {
				if (method.getName().equals("commit")) {
					TimeUnit.MILLISECONDS.sleep(100);
				}
				return null;
			}
		});
	}
}
```

上述代码使用了CountDownLatch来确保ConnectionRunner线程能够同时开始执行，并且在全部结束之后，才能使main线程从等待状态中返回。实验结果表明，在资源一定的情况下（连接池中的10个连接），随着客户端线程的逐步增加，客户端出现超时无法获取的比率不断升高。虽然客户端线程在这种超时获取的模式下无法正常获取连接，但它能够保证客户端线程不会一致挂在连接获取的操作上，而是“按时”返回，并告知客户端连接获取出现问题，是系统的一种自我保护机制。数据库连接池的设计也可以复用到其他资源的获取场景，针对昂贵的资源的获取均应该加以超时限制。<br>



### 连接池技术及其示例

对于服务端的程序，经常面对的是客户端传入的短小（执行时间短、工作内容较为单一）任务，需要服务器快速处理并返回结果。如果服务端每接收到一次任务，就创建一个线程，然后去执行，这在源性阶段是个不错的选择，但是面对成千上万的任务递交进服务器时，如果还是采用一个任务一个线程的方式，那么将会创建数以万计的线程，这不是一个好的选择。因为这会使得操作系统频繁的进行线程的上下文切换，无故增加系统的负载，而线程的创建和消亡都是需要耗费系统资源的，这无疑浪费了系统资源。

线程池技术很好的解决了这个问题（以空间换时间的典型事例）。它预先创建了若干数量的线程，并且不能由用户直接对线程的创建进行控制，在这个前提下重复使用固定或较为固定的线程来完成任务的执行。这样做的好处是：

* 消除频繁创建和消亡线程的系统资源开销
* 面对过量任务的提交能够平缓的劣化

下面先看一个简单的线程池接口定义：

```java
/**
 * 客户端可以通过execute方法将job提交到线程池执行，而客户端自身不用等待job的执行完成
 * 这里的工作者线程代表着一个重复执行job的线程，而每个由客户端提交的job都将进入到一个工
 * 作者队列中，等待工作者线程的处理
 */
public interface ThreadPool<Job extends Runnable> {
	// 执行一个job，这个job需要实现Runnbale
	void execute(Job job);
	
	//关闭线程池
	void shutdown();
	
	//增加工作者线程
	void addWorkers(int num);
	
	//减少工作线程
	void removeWorkers(int num);
	
	//得到正在等待执行的任务总量
	int getJobSize();
}
```

接下来是线程池接口的默认实现：

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.LinkedList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

public class DefaultThreadPool<Job extends Runnable> implements ThreadPool<Job> {
	//线程池最大限制数
	private static final int MAX_WORKER_NUMBERS = 10;
	//线程池默认的数量
	private static final int DEFAULT_WORKER_NUMBERS = 5;
	//线程池最小的数量
	private static final int MIN_WORKER_NUMBERS = 5;
	//这是一个工作列表，将会向里面插入工作
	private final LinkedList<Job> jobs = new LinkedList<>();
	//工作者列表
	private final List<Worker> workers = Collections.synchronizedList(new ArrayList<>());
	//工作者线程的数量
	private int workerNum = DEFAULT_WORKER_NUMBERS;
	//线程编号生成器
	private AtomicLong threadNum = new AtomicLong();
	
	public DefaultThreadPool() {
		initializeWorkers(DEFAULT_WORKER_NUMBERS);
	}
	public DefaultThreadPool(int num) {
		workerNum = num > MAX_WORKER_NUMBERS ? 
          		MAX_WORKER_NUMBERS : num < MIN_WORKER_NUMBERS ? 
                  MIN_WORKER_NUMBERS : num;
		initializeWorkers(workerNum);
	}
	private void initializeWorkers(int num) {
		for (int i = 0; i < num; i++) {
			Worker worker = new Worker();
			workers.add(worker);
			new Thread(worker, "Thread-Worker-" + threadNum.incrementAndGet()).start();
		}
	}

	@Override
	public void execute(Job job) {
		if (job != null) {
			//添加一个工作，然后进行通知
			synchronized (jobs) {
				jobs.addLast(job);
				jobs.notify();
			}
		}
	}

	@Override
	public void shutdown() {
		for (Worker worker : workers) {
			worker.shutdown();
		}
	}

	@Override
	public void addWorkers(int num) {
		synchronized (jobs) {
			//限制新增的worker数量不能超过最大值
			if (num+this.workerNum > MAX_WORKER_NUMBERS) {
				num = MAX_WORKER_NUMBERS-this.workerNum;
			}
			initializeWorkers(num);
			this.workerNum += num;
		}
	}

	@Override
	public void removeWorkers(int num) {
		synchronized (jobs) {
			if (num > this.workerNum) {
				throw new IllegalArgumentException("beyond workerNum.");
			}
			//按照给定的数量停止worker
			int count = 0;
			while (count<num) {
				Worker worker = workers.get(count);
				if (workers.remove(worker)) {
					count++;
				}
			}
			this.workerNum -= count;
		}
	}

	@Override
	public int getJobSize() {
		return jobs.size();
	}
	
	
	/**
	 * 工作者：负责消费任务
	 */
	public class Worker implements Runnable {
		//是否在工作
		private volatile boolean running = true;

		@Override
		public void run() {
			while (running) {
				Job job = null;
				synchronized (jobs) {
					while (jobs.isEmpty()) {
						try {
							jobs.wait();
						} catch (InterruptedException e) {
							//感知到外部对workerThread的中断操作，返回
							Thread.currentThread().interrupt();
							return;
						}
					}
					//取出一个job
					job = jobs.removeFirst();
					if (job != null) {
						try {
							//注意是run而不是start
							job.run(); 
						} catch (Exception e) {
							// TODO
						}
					}
				}
			}
		}
		
		public void shutdown() {
			this.running = false;
		}
	}
}
```

从线程池的默认实现我们可以看到，当客户端调用execute(Job)方法时，会不断向任务列表jobs中添加job，而每个工作线程会不断从jobs上取出一个job进行执行，当jobs为空时，工作者线程进入等待状态。

添加一个job后，对工作队列jobs调用了其notify方法，而不是notifyAll，因为能够确保有工作者线程被唤醒，这时使用notify竟会比使用notifyAll获得更小的开销（避免将等待队列中的线程全部移动到阻塞队列中）。

`线程池的本质` 就是使用了一个线程安全的工作队列连接工作者线程和客户端线程。客户端线程将任务放入工作队列后边返回，而工作者线程则不断从工作队列上取出任务并执行。当工作队列为空的时候，所有的工作线程均【等待在工作队列上】，当客户端提交了一个任务之后，会通知任意一个工作线程，随着大量的任务被提交，更多的工作线程将被唤醒。

<br>



### 一个基于线程池技术的简单web服务器

目前的浏览器都支持多线程访问，比如在请求一个html页面的时候，页面中包含的图片资源、样式资源会被浏览器并发地发起访问获取，这样用户就不会遇到一直等到一个图片完全下载完才能查看文字内容的尴尬情况。

如果服务器是单线程的，多线程的浏览器也没有用武之地，因为服务器还是一个请求一个请求的顺序处理。因此，大部分web服务器都是支持并发访问的。常用的web服务器是tomcat、jetty，其在处理请求的过程中都使用了线程池技术。

下面通过上述线程池实现来构造一个简单的web服务器，这个web服务器用来处理http请求，目前只能处理简单的文本和jpg图片内容。这个服务器使用main线程不断加接收客户端socket连接，将连接以及请求提交给线程池处理，这使得web服务器能够同时处理多个客户端请求。

```java
import java.io.BufferedReader;
import java.io.Closeable;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

public class SimpleHttpServer {
    // 处理HttpRequest的线程池
    static ThreadPool<HttpRequestHandler> threadPool = new DefaultThreadPool<HttpRequestHandler>(11);
    // SimpleHttpServer的根路径
    static String basePath = "/Users/zhangqingli/Documents/workspace-ec/tongniu/zdemo-thread/src";
    static ServerSocket serverSocket;
    // 服务器监听端口
    static int port = 8080;

    public static void setPort(int port) {
        if (port > 0) {
            SimpleHttpServer.port = port;
        }
    }

    public static void setBasePath(String basePath) {
        if (basePath != null && new File(basePath).exists() && new File(basePath).isDirectory()) {
            SimpleHttpServer.basePath = basePath;
        }
    }

    // 启动SimpleHttpServer
    public static void start() throws Exception {
        serverSocket = new ServerSocket(port);
        Socket socket = null;
        while ((socket = serverSocket.accept()) != null) {
            // 接收一个客户端Socket，生成一个HttpRequestHandler，放入线程池中执行
            threadPool.execute(new HttpRequestHandler(socket));
        }
        serverSocket.close();
    }

    static class HttpRequestHandler implements Runnable {

        private Socket socket;

        public HttpRequestHandler(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            String line = null;
            BufferedReader br = null;
            BufferedReader reader = null;
            PrintWriter out = null;
            InputStream in = null;
            try {
                reader = new BufferedReader(
                  new InputStreamReader(socket.getInputStream()));
                String header = reader.readLine();
                //由相对路径计算得出绝对路径
                String filePath = basePath + header.split(" ")[1];
                out = new PrintWriter(socket.getOutputStream());
                
                //打印http请求
                System.out.println(header);
                while (true) {
                		line = reader.readLine();
                		System.out.println(line);
                		if (line==null || "".equals(line.trim())) {
                			break;
                		}
                }
                
                //如果请求jpg或ico资源
                if (filePath.endsWith("jpg") || filePath.endsWith("ico")) {
                    out.println("HTTP/1.1 200 OK");
                    out.println("Content-Type: image/jpeg");
                    out.println("");
                    out.flush();
                    
                    in = new FileInputStream(filePath);
                    OutputStream os = socket.getOutputStream();
                    byte[] bs = new byte[1024];
                    int len = 0;
                    while ((len=in.read(bs)) != -1) {
                    		os.write(bs, 0, len);
                    }
                    os.flush();
                } else {
                    br = new BufferedReader(
                      new InputStreamReader(new FileInputStream(filePath)));
                    out = new PrintWriter(socket.getOutputStream());
                    out.println("HTTP/1.1 200 OK");
                    out.println("Content-Type: text/html; charset=UTF-8");
                    out.println("");
                    while ((line = br.readLine()) != null) {
                        out.println(line);
                    }
                }
                out.flush();
            } catch (Exception ex) {
                out.println("HTTP/1.1 500");
                out.println("");
                out.flush();
            } finally {
                close(br, in, reader, out, socket);
            }
        }
    }

    //关闭流或socket
    private static void close(Closeable... closeables) {
        if (closeables != null) {
            for (Closeable closeable : closeables) {
                try {
                    closeable.close();
                } catch (Exception ex) {
                    // ignore
                }
            }
        }
    }
    
    /**
     * 启动服务器
     */
    public static void main(String[] args) throws Exception {
		start();
	}
}
```

测试页面index.html（存放在basePath目录下）：

```html
<html>
	<head>
		<title>测试</title>
	</head>
	<body>
		<h1>第一张图片</h1>
		<img src="1.jpg" style="width:60%">
		<h1>第二张图片</h1>
		<img src="2.jpg" style="width:60%">
		<h1>第三张图片</h1>
		<img src="3.jpg" style="width:60%">
	</body>
</html>
```

测试步骤：

```java
//第一步：启动SimpleHttpServer服务端
//第二步：在浏览器地址栏输入 localhost:8080/index.html 即可看到服务端返回的响应

//服务端输出的http请求：
GET /index.html HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: Hm_lvt_d88c5dc8895505e7e0140c84e7e941f1=1493494970; _uab_collina=149365565849393142850533

GET /1.jpg HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36
Accept: image/webp,image/apng,image/*,*/*;q=0.8
Referer: http://localhost:8080/index.html
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: Hm_lvt_d88c5dc8895505e7e0140c84e7e941f1=1493494970; _uab_collina=149365565849393142850533

GET /2.jpg HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36
Accept: image/webp,image/apng,image/*,*/*;q=0.8
Referer: http://localhost:8080/index.html
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: Hm_lvt_d88c5dc8895505e7e0140c84e7e941f1=1493494970; _uab_collina=149365565849393142850533

GET /3.jpg HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36
Accept: image/webp,image/apng,image/*,*/*;q=0.8
Referer: http://localhost:8080/index.html
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: Hm_lvt_d88c5dc8895505e7e0140c84e7e941f1=1493494970; _uab_collina=149365565849393142850533


//浏览器响应头
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Server: Keyllo
  
HTTP/1.1 200 OK
Content-Type: image/jpeg
Server: Keyllo
  
HTTP/1.1 200 OK
Content-Type: image/jpeg
Server: Keyllo
  
HTTP/1.1 200 OK
Content-Type: image/jpeg
Server: Keyllo
```

SimpleHttpServer与客户端建立连接之后，并不会处理客户端的请求，而是将器包装成HttpRequestHandler并交由线程池处理，在线程池的Worker处理客户端请求的同时，SimpleHttpServer能继续完成后续客户端连接的建立。不会阻塞后续客户端的请求。

下面通过Apache HTTP server benchmarking tool （版本2.3）来测试不同线程数下，SimpleHttpServer的吞吐量的表现。

测试场景是5000次请求，分10个线程并发执行，测试内容主要考察响应时间（越小越好）和每秒查询的数量（越高越好）测试结果如下如所示：

```default
线程池数量			1			5			10
响应时间			0.352		0.246		 0.163
每秒查询的数量		  3076		  4065		   6123
测试完成时间		   1.625	   1.230		0.816
```

可以看到，随着线程数的增加，SimpleHttpServer的吞吐量不断增大，响应时间不断变小，线程池的作用非常明显。

但是线程池并不是越多越好，具体的数量需要评估每个任务的处理时间，以及当前计算机处理器能力和数量。使用的线程过少，无法发挥处理器的性能；使用的线程过多，将会增加系统的无故开销，反而会起到反作用。