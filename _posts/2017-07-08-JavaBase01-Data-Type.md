---
layout: post
title:  "Java数据类型、类加载器"
desc: "Java数据类型、类加载器"
keywords: "Java数据类型、类加载器,java,kinglyjn"
date: 2012-11-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

# Java Data Type

### 整型

下面是整数据的内存模型

```java
int    4个字节		-2^31~2^31-1(大约在+-20亿范围)
short  2个字节		-2^15~2^15-1(大约在+-3万范围)
long   8个字节		-2^63~2^63-1(大约在900亿亿范围) 
byte   1个字节	    -2^7~2^7-1(在-128~127范围内)
```

整形数据的进制前缀：

```java
2进制    0B    如 (0B1001)=9
8进制    0     如 (010)=8
16进制   0X    如 (0Xff)=255
```

从Java+7开始，加上前缀0b或0B就可以写二进制数。例如，0b1001就是9。另外，同样是从Java+7开始，还可以为数字字面量加下划线，如用1_000_000（或0b1111_0100_0010_0100_0000）表示一百万。这些下划线只是为了让人更易读。Java编译器会去除这些下划线。

<br>



### 符点型

```java
单精度 float    4个字节    取值范围大约 +-3.403E38  (有效位为6~7位)
双精度 double   8个字节    取值范围大约 +-1.798E308 (有效位为15位)
```

浮点数值不适用于无法接受舍入误差的金融计算中。例如，命令System.out.println（2.0–1.1）将打印出0.8999999999999999，而不是人们想象的0.9。这种舍入误差的主要原因是浮点数值采用二进制系统表示，而在二进制系统中无法精确地表示分数1/10。这就好像十进制无法精确地表示分数1/3一样。如果在数值计算中不允许有任何舍入误差，就应该使用BigDecimal & DecimalFormat类。<br>

自动类型提升和轻质类型转换：

```default
				   char
					|
byte --> short --> int  -->  long
					|		  |
					|         |
					|/		  |/
				   float --> double	
```

位运算符：

```java
&   and
|   or
~   not
  
<<  left   
>>  right (使用符号位填充高位)
>>> right (使用0填充高位)
  
System.out.println(i>>1);	//-1
System.out.println(0B11111111111111111111111111111111); //-1 (原码取反加一)

System.out.println(i>>>1);	//2147483647
System.out.println(0B01111111111111111111111111111111); //2147483647
```

<br>



### 字符型型

char在Java中是16位的，因为Java用的是Unicode。不过8位的ASCII码包含在Unicode中，是从0~127的。

Java中使用Unicode的原因是，Java的Applet允许全世界范围内运行，那它就需要一种可以表述人类所有语言的字符编码。Unicode。但是English，Spanish，German, French根本不需要这么表示，所以它们其实采用ASCII码会更高效。这中间就存在一个权衡问题。

因为char是16位的，采取的Unicode的编码方式，所以char就有以下的初始化方式：

```java
char c='a'; //字符，可以是汉字，因为是Unicode编码
char c=97; 	//十进制数，八进制数，十六进制数等等; //可以用整数赋值
char c='\u2345'; //如：char='\0',表示结束符，它的ascll码是0，这句话的意思和 char c=0 是一个意思。
```

<br>



### 布尔型

```java
boolean flag0 = false;
boolean flag1 = true;
```

<br>



### 引用数据类型

包括类（class）、接口（interface）、枚举（enum）、注解（annotation）

枚举示例：

```java
package com.ds.model.common;

public interface StateCode {
	/**
	 * 系统内部业务处理码
	 */
	public static enum SysCodeEnum {
		SYS_INIT("0100", "系统初始化"),
		SUCCESS("0200", "数据处理成功"),
		ERROR("0500", "系统内部异常");
		
		private String code;
		private String desc;
		private SysCodeEnum(String code, String desc) {
			this.code = code;
			this.desc = desc;
		}
		public String getCode() {
			return code;
		}
		public String getDesc() {
			return desc;
		}
	}
	
	
	/**
	 * 数据状态码
	 */
	public static enum DataCodeEnum {
		DATA_IS_NOMAL("1200", "数据正常"),
		DATA_IS_INVALID("1100", "数据无效"),
		DATA_IS_NULL("1000", "数据为空");
		
		private String code;
		private String desc;
		private DataCodeEnum(String code, String desc) {
			this.code = code;
			this.desc = desc;
		}
		public String getCode() {
			return code;
		}
		public String getDesc() {
			return desc;
		}
	}
}
```

<br>



注解示例：

```java
package com.ds.model.scanner;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import com.ds.model.common.CodeEnum;
import com.ds.model.common.SrcEnum;
import com.ds.model.router.RouteMethod;

/**
 * 业务链组件元数据
 * @author zhangqingli
 *
 */
@Target(ElementType.TYPE) //ANNOTATION_TYPE、CONSTRUCTOR、FIELD、LOCAL_VARIABLE、METHOD、PACKAGE、PARAMETER
@Retention(RetentionPolicy.RUNTIME) //CLASS、SOURCE
@Documented
//@Inherited
public @interface ScanableComponent {
	CodeEnum code();
	ComponentType type();
	SrcEnum src() default SrcEnum.UNKNOWN;
	String beanId();
	RouteMethod routeMethod() default RouteMethod.ORDER;
	int acquireRetryAttempts() default 1;
	int order() default Integer.MAX_VALUE;
	double weight() default 0;
}


//使用注解
@Component
@ScanableComponent(code=CodeEnum.BANK_CNCM, beanId="zcxCncmProcessor", type=ComponentType.PROCESSOR, src=SrcEnum.ZCX, 
	routeMethod=RouteMethod.ORDER, order=2, acquireRetryAttempts=1)
public class ZcxCncmProcessor extends AbstractProcessor {
	// ...
}
```

<br>



# 关于类的加载器

```java
类加载器是用来把类(.class)装载进内存的。jvm规范定义了两种类型的类加载器：启动类加载器(bootstrap)和用户自定义加载器(user-defined class loader)。jvm在运行时会产生3个类加载器组成的初始化加载器层次结构，如下所示：

  
  从前往后尝试加载类
  -------------------------------------------------------------------------->
  BootstrapClassloader  ---->  ExtensionClassLoader  ---->  SystemClassLoader
  <--------------------------------------------------------------------------
  														从后往前检查类是否已装载
  
  
BootstrapClassloader：引导类加载器，用C++编写的，是jvm自带的类加载器，负责加载java平台核心库，该加载器					 无法直接获取。
ExtensionClassLoader：负责jre/lib/ext目录下的jar包 或 -Djava.ext.dirs 指定目录下的jar包装载。
SystemClassLoader：系统类加载器，负责java --classpath 或者 -Djava.class.path 所指的目录下的类与					   jar包的装载，是最常用的类加载器。
  
  
ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
ClassLoader classLoader = clazz.getClassLoader();
```

<br>





