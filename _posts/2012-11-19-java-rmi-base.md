---
layout: post
title:  "java RMI"
desc: "java RMI"
keywords: "java RMI,java,kinglyjn"
date: 2012-11-19
categories: [Java]
tags: [java]
icon: fa-coffee
---


### 简介

RMI（即Remote Method Invoke 远程方法调用）。在Java中，只要一个类extends了java.rmi.Remote接口，即可成为存在于服务器端的远程对象，供客户端访问并提供一定的服务。JavaDoc描述：Remote 接口用于标识其方法可以从非本地虚拟机上调用的接口。任何远程对象都必须直接或间接实现此接口。只有在“远程接口”（扩展 java.rmi.Remote 的接口）中指定的这些方法才可远程使用。 <br>

注意：extends了Remote接口的类或者其他接口中的方法若是声明抛出了RemoteException异常，则表明该方法可被客户端远程访问调用。
<br>

同时，远程对象必须实现java.rmi.server.UniCastRemoteObject类，这样才能保证客户端访问获得远程对象时，该远程对象将会把自身的一个拷贝以Socket的形式传输给客户端，此时客户端所获得的这个拷贝称为“存根”，而服务器端本身已存在的远程对象则称之为“骨架”。其实此时的存根是客户端的一个代理，用于与服务器端的通信，而骨架也可认为是服务器端的一个代理，用于接收客户端的请求之后调用远程方法来响应客户端的请求。 <br>

RMI 框架的基本原理大概如下图，应用了代理模式来封装了本地存根与真实的远程对象进行通信的细节。<br>

<img src="http://img.blog.csdn.net/20170321160437694?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br>

下面给出一个简单的RMI 应用，其中类图如下：其中IService接口用于声明服务器端必须提供的服务（即service()方法），ServiceImpl类是具体的服务实现类，而Server类是最终负责注册服务器远程对象，以便在服务器端存在骨架代理对象来对客户端的请求提供处理和响应。<br>

<img src="http://img.blog.csdn.net/20170321160551493?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br>

其实整个简单的RMI 应用中各个类的交互时序如下图：<br>

<img src="http://img.blog.csdn.net/20170321160636345?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br><br>

### 编写步骤

```default
RMI（Remote Method Invocation）
=======================================================
使用步骤：
1. 编写远程接口（需要继承java.rmi.Remote接口）
2. 编写远程接口实现（继承UnicastRemoteObject，并实现1所写的远程接口）
3. 编写远程服务端启动和注册类，将远程对象注册到RMI注册服务器
4. 启动RMI注册器（这里有两种方法：在远程服务器代码中创建和启动（LocateRegistry.createRegistry(1099)），和在相应目录命令行启动> rmiregister）
5. 编译和运行服务端代码，并在服务端生成客户端stub（例如：rmic HelloImpl），将生成的客户端stub（例如：HelloImpl_Stub.class）拷贝到客户端与服务端代码相同的目录下
	|>javac HelloServer.java HelloImpl.java
	|>rmic HelloImpl
	|>java HelloServer
	| 或
	|>java -Djava.security.policy=$JAVA_HOME/jre/lib/security/policy.all HelloServer (事先编写了policy.all文件)
	| 服务端指定了安全管理器 System.setSecurityManager(new RMISecurityManager());（客户端类似）
	|
	|>sudo vi policy.all
	|	grant{
	|   	permission java.security.AllPermission "","";
	|	};
6. 编写、编译、运行客户端代码 



RMI局限性：
1.RMI对服务器的IP地址和端口依赖很紧密，但是在开发的时候不知道将来的服务器IP和端口如何，但是客户端程序依赖这个IP和端口。
  这个问题有两种解决途径：一是通过DNS来解决，二是通过封装将IP暴露到程序代码之外。
	 
2. RMI是Java语言的远程调用，两端的程序语言必须是Java实现，对于不同语言间的通讯可以考虑用Web Service或者公用对象请求代理体系（CORBA）来实现。
```
<br><br>

### 示例

```java
package test02.timesync;
import java.rmi.Remote;
import java.rmi.RemoteException;
/**
 * 定义 RMI接口方法
 * 继承Remote接口，并抛出RemoteException异常
 * @author zhangqingli
 *
 */
public interface ITimeServer extends Remote {
	long getServerTime() throws RemoteException;
}


package test02.timesync;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
/**
 * 实现 RMI接口方法
 * 继承UnicastRemoteObject类，并实现刚定义的RMI接口
 * @author zhangqingli
 *
 */
public class TimeServer extends UnicastRemoteObject implements ITimeServer {
	private static final long serialVersionUID = 1L;

	protected TimeServer() throws RemoteException {
		super();
	}

	@Override
	public long getServerTime() throws RemoteException {
		return System.currentTimeMillis();
	}
}


package test02.timesync;
import java.rmi.Naming;
import java.rmi.registry.LocateRegistry;
/**
 * RMI服务端启动类
 * @author zhangqingli
 *
 */
public class StartServer {
	
	/*
	 * 1.java1.4以后根和茎统一使用一个类Xxx_Stub.class，这个类帮助我们实现服务端和客户端RMI功能类的序列化和反序列化
	 * 2.这个stub类需要借助于java rmi编译器[rmic]来生成（类似于javac）
	 * 3.服务端需要开启rmi注册服务[rmiregistry]（也可以在程序中使用LocateRegistry.createRegistry(1099)启动）
	 */
	public static void main(String[] args) throws Exception {
		//启动rmi注册服务器
		LocateRegistry.createRegistry(1099);
		//注册功能类
		TimeServer ts = new TimeServer();
		//Naming.bind("ts1", ts); //如果发现rmiregistry服务器中已经绑定了ts1，则会抛出异常
		Naming.rebind("ts1", ts); //如果发现rmiregistry服务器中已经绑定了ts1，则会替换之前绑定的ts1对象
		System.out.println("bind timeServer complete!");
	}
}


package test02.timesync;
import java.rmi.Naming;
/**
 * 这是RMI服务客户端
 * 拷贝服务端的功能类接口（ITimeSevrer）和服务端生成的XXX_Stub.class到客户端对应的目录下，就可以直接调用服务端暴露的RMI接口
 * @author zhangqingli
 *
 */
public class TimeClient1 {
	
	public static void main(String[] args) throws Exception {
		ITimeServer ts1 = (ITimeServer) Naming.lookup("rmi://keyllopcm:1099/ts1");
		System.out.println("server time is: " + ts1.getServerTime()); 
	}
}
```
<br><br>

