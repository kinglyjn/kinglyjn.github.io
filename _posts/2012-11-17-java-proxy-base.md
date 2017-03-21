---
layout: post
title:  "java动态代理（类加载、asm、cglib、javassist）"
desc: "java动态代理（类加载、asm、cglib、javassist）"
keywords: "java动态代理（类加载、asm、cglib、javassist）,动态代理,java,kinglyjn"
date: 2012-11-17
categories: [Java]
tags: [java]
icon: fa-coffee
---

### class文件简介及加载

Java编译器编译好Java文件之后，产生.class 文件在磁盘中。这种class文件是二进制文件，内容是只有JVM虚拟机能够识别的机器码。JVM虚拟机读取字节码文件，取出二进制数据，加载到内存中，解析.class 文件内的信息，生成对应的 Class对象:

<img src="http://img.blog.csdn.net/20170321094108007?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br>

class字节码文件是根据JVM虚拟机规范中规定的字节码组织规则生成的、具体class文件是怎样组织类信息的，可以参考 此博文：[深入理解Java Class文件格式系列](http://blog.csdn.net/zhangjg_blog/article/details/21486985)。或者是[Java虚拟机规范](http://download.csdn.net/detail/u010349169/7439669)。<br>

下面通过一段代码演示手动加载 class文件字节码到系统内，转换成class对象，然后再实例化的过程：<br>

 a. 定义自己的类加载器：<br>
 
```java
package classloadertest;

public class MyClassLoader<T> extends ClassLoader {

	/**
	 * 自定义一个类加载器，用于将字节码转换为class对象 
	 * @param name   The expected binary name of the class, or null if not known
	 * @param bs
	 * @param off
	 * @param len
	 * @return
	 */
	@SuppressWarnings("unchecked")
	public Class<T> defineMyClass(String name, byte[] bs, int off, int len) {
		return (Class<T>) super.defineClass(name, bs, off, len);
	}
}
```
<br>

 b. 测试：<br>

```java
package classloadertest;

import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Method;
import javax.persistence.UniqueConstraint;
import sun.reflect.CallerSensitive;

/**
 * 程序猿类
 * @author zhangqingli
 *
 */
public class Programmer {
	
	public void code() {
		System.out.println("I am a Programmer, just coding ...");
	}
	
	public static void main(String[] args) throws Exception {
		// 获取字节码数组
		String classRoot = Programmer.class.getResource("/").getPath();
		InputStream is = new FileInputStream(classRoot+"/classloadertest/Programmer.class");
		byte[] bs = new byte[is.available()];
		int len = is.read(bs);
		
		// 将字节码数组转化为内存中的类对象
		Class<Programmer> clazz = new MyClassLoader<Programmer>().defineMyClass(null, bs, 0, len);
		System.out.println("生成的class对象为：" + clazz.getCanonicalName()); 
		
		// 利用反射生成对象并调用对象的方法
		Object o = clazz.newInstance();
		clazz.getMethod("code", null).invoke(o, null);
	}
}
```
<br><br>

### 在运行期的代码中生成二进制字节码

由于JVM通过字节码的二进制信息加载类的，那么，如果我们在运行期系统中，遵循Java编译系统组织.class文件的格式和结构，生成相应的二进制数据，然后再把这个二进制数据加载转换成对应的类，这样，就完成了在代码中，动态创建一个类的能力了。<br>

<img src="http://img.blog.csdn.net/20170321104314174?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br>

在运行时期可以按照Java虚拟机规范对class文件的组织规则生成对应的二进制字节码。当前有很多开源框架可以完成这些功能，如ASM，Javassist。<br><br>

### Java字节码生成开源框架介绍--ASM：

ASM 是一个 Java 字节码操控框架。它能够以二进制形式修改已有类或者动态生成类。ASM 可以直接产生二进制 class 文件，也可以在类被加载入 Java 虚拟机之前动态改变类行为。ASM 从类文件中读入信息后，能够改变类行为，分析类信息，甚至能够根据用户要求生成新类。<br>

不过ASM在创建class字节码的过程中，操纵的级别是底层JVM的汇编指令级别，这要求ASM使用者要对class组织结构和JVM汇编指令有一定的了解。<br>

下面通过ASM 生成下面类Programmer的class字节码：<br>

```java
package asmtest;

/**
 * 程序猿类
 * @author zhangqingli
 *
 */
public class Programmer {
	
	public void code() {
		System.out.println("I am a Programmer, just coding ...");
	}
}
```
<br>

使用ASM框架提供了ClassWriter 接口，通过访问者模式进行动态创建class字节码，看下面的例子：<br>

```java
package asmtest;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import net.sf.cglib.asm.ClassWriter;
import net.sf.cglib.asm.MethodVisitor;
import net.sf.cglib.asm.Opcodes;

/**
 * 使用asm动态创建class字节码
 * @author zhangqingli
 *
 */
public class MyGenerator {

	public static void main(String[] args) throws IOException {
		ClassWriter classWriter = new ClassWriter(0);  
        // 通过visit方法确定类的头部信息  
        classWriter.visit(Opcodes.V1_7,	// java版本  
                Opcodes.ACC_PUBLIC,		// 类修饰符  
                "Programmer", 			// 类的全限定名  
                null, "java/lang/Object", null);  
          
        //创建构造函数  
        MethodVisitor mv = classWriter.visitMethod(Opcodes.ACC_PUBLIC, "<init>", "()V", null, null);  
        mv.visitCode();  
        mv.visitVarInsn(Opcodes.ALOAD, 0);  
        mv.visitMethodInsn(Opcodes.INVOKESPECIAL, "java/lang/Object", "<init>","()V");  
        mv.visitInsn(Opcodes.RETURN);  
        mv.visitMaxs(1, 1);  
        mv.visitEnd();  
          
        // 定义code方法  
        MethodVisitor methodVisitor = classWriter.visitMethod(Opcodes.ACC_PUBLIC, "code", "()V",  
                null, null);  
        methodVisitor.visitCode();  
        methodVisitor.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/System", "out",  
                "Ljava/io/PrintStream;");  
        methodVisitor.visitLdcInsn("I'm a Programmer,Just Coding.....");  
        methodVisitor.visitMethodInsn(Opcodes.INVOKEVIRTUAL, "java/io/PrintStream", "println",  
                "(Ljava/lang/String;)V");  
        methodVisitor.visitInsn(Opcodes.RETURN);  
        methodVisitor.visitMaxs(2, 2);  
        methodVisitor.visitEnd();  
        classWriter.visitEnd();   
        
        // 使classWriter类已经完成  
        // 将classWriter转换成字节数组写到文件里面去  
        byte[] data = classWriter.toByteArray();  
        //
        File dir = new File(System.getProperty("user.dir")+"/tmp"); 
        if (!dir.exists()) {
        	dir.mkdirs();
        }
        File file = new File(dir, "Programmer.class");
        //
        FileOutputStream fout = new FileOutputStream(file);  
        fout.write(data);  
        fout.close();  
	}
}
```
<br>

上述的代码执行过后，用Java反编译工具（如JD_GUI）可以看到生成的Programmer.class字节码文件。<br><br>

### Java字节码生成开源框架介绍--Javassist：

Javassist是一个开源的分析、编辑和创建Java字节码的类库。是由东京工业大学的数学和计算机科学系的 Shigeru Chiba （千叶 滋）所创建的。它已加入了开放源代码JBoss 应用服务器项目,通过使用Javassist对字节码操作为JBoss实现动态AOP框架。javassist是jboss的一个子项目，其主要的优点，在于简单，而且快速。直接使用java编码的形式，而不需要了解虚拟机指令，就能动态改变类的结构，或者动态生成类。<br>

下面通过Javassist创建上述的Programmer类：<br>


```java
package javassisttest;

import java.io.File;
import java.io.IOException;
import javassist.CannotCompileException;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
import javassist.CtNewMethod;

/**
 * 通过Javassist创建Programmer.class字节码
 * @author zhangqingli
 *
 */
public class MyGenerater {
	
	public static void main(String[] args) throws CannotCompileException, IOException {
		ClassPool pool = ClassPool.getDefault();
		
		// 创建Programmer类
		CtClass cc = pool.makeClass("javassisttest.Programmer");
		// 定义code方法
		CtMethod codeMethod = CtNewMethod.make("public void code(){}", cc);
		// 插入方法代码
		codeMethod.insertBefore("System.out.print(\"哈哈哈before\");");
		cc.addMethod(codeMethod);
		
		//保存生成的字节码
		File dir = new File(System.getProperty("user.dir")+"/tmp"); 
        if (!dir.exists()) {
        	dir.mkdirs();
        }
        cc.writeFile(dir.getCanonicalPath());
	}
}
```
<br>



### java代理（需要实现特定接口）


```java
package cglibtest;

/**
 * 该类是所有被代理类的接口类，jdk实现的代理要求被代理类基于统一的接口
 * 
 */
public interface IService01 {
	void add();
	void update();
}




package cglibtest;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
/**
 * jdk动态代理需要目标类基于统一的接口
 *
 */
public class JdkAopTest01 {
	public static void main(String[] args) {
		//生成代理类
		IService01 iServiceProxy = (IService01) MyInvocationHandler01.generateProxyInstance(new Service01());
		
		//使用代理类
		iServiceProxy.add();
		iServiceProxy.update();
	}
}


//被代理类，即目标类target
class Service01 implements IService01 {

	@Override
	public void add() {
		System.out.println("Service01 add >>>>>>>>>>>>>>");
		Integer.parseInt("aaa");
	}

	@Override
	public void update() {
		System.out.println("Service01 update >>>>>>>>>>>>>>");
	}
}

//切面处理类InvocationHandler
class MyInvocationHandler01 implements InvocationHandler {
	private Object target;
	public MyInvocationHandler01(Object target) {
		this.target = target;
	}

	// 生成代理类
	public static Object generateProxyInstance(Object target) {
		MyInvocationHandler01 myInvocationHandler01 = new MyInvocationHandler01(target);
		return Proxy.newProxyInstance(target.getClass().getClassLoader(), 
				target.getClass().getInterfaces(),
				myInvocationHandler01);
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args)
			throws Throwable {
		//程序执行前加入逻辑，MethodBeforeAdviceInterceptor
		System.out.println("before------------------");
		
		//程序执行
		Object result = null;
		try {
			result = method.invoke(target, args);
		} catch (Exception e) {
			System.out.println("=========异常=========");
		} finally {
			System.out.println("=======执行finally========");
		}
		
		//程序执行后加入逻辑，MethodAfterAdviceInterceptor  
		System.out.println("after------------------------------");
		return result;
	}
}
```
<br><br>


### cglib代理（不需要实现特定的接口，它是通过生成目标子类实现代理的）

```java
package cglibtest;
import java.lang.reflect.Method;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
/**
 * cglib动态代理不需要目标类基于统一的接口
 * @author zhangqingli
 *
 */
public class CglibProxyTest02 {
	public static void main(String[] args) {
		//生成代理类
		Service02 serviceProxy = (Service02) MyInterceptor.generateProxyInstance(Service02.class);
		
		//使用代理类
		System.out.println(serviceProxy.add(100));
		serviceProxy.update();
	}
}



//被代理类，即目标对象target
class Service02 {
	
	public Integer add(Integer id) {
		System.out.println("Service02 add >>>>>>>>>>>>>>>>>");
		Integer.parseInt("aaa");
		return id;
	}
	
	public void update() {
		System.out.println("Service02 update >>>>>>>>>>>>>>>");
	}
}

//切面处理类，MethodInterceptor
class MyInterceptor implements MethodInterceptor {
	
	// 生成代理对象
	public static Object generateProxyInstance(Class<?> targetType) {
		MyInterceptor myInterceptor = new MyInterceptor();
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(targetType);
		enhancer.setCallback(myInterceptor);
		return enhancer.create();
	}
	
	@Override
	public Object intercept(Object obj, Method method, Object[] args,
			MethodProxy methodProxy) throws Throwable {
		//程序执行前加入逻辑，MethodBeforeAdviceInterceptor
		System.out.println("before--------------------------");
		
		//程序执行
		Object result = null;
		try {
			result = methodProxy.invokeSuper(obj, args); //使用method和methodProxy都能调用invoke方法，并且执行效果一样
		} catch (Exception e) {
			System.out.println("===========异常===========");
			throw e;
		} finally {
			System.out.println("=========执行finally==========");
		}
		
		
		//程序执行后加入逻辑，MethodAfterAdviceInterceptor
		System.out.println("after--------------------------");
		return result;
	}
}
```
<br>

