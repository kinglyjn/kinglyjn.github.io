---
layout: post
title:  "Java run Js"
desc: "Java run Js"
keywords: "Java run Js,java,js,kinglyjn"
date: 2012-11-16
categories: [Java]
tags: [java]
icon: fa-coffee
---


### 示例一

```java
import java.io.File;
import java.io.FileReader;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
public class CommonTest {
 public static void main(String[] args) throws Exception {
  ScriptEngineManager sem=new ScriptEngineManager();
  ScriptEngine se=sem.getEngineByExtension("js");
  se.eval(new FileReader(new File("src\\test\\java\\test\\md5.js")));
  String s=(String) se.eval("toMD5Str('1')");
  System.out.println(s);
 }
}


//运行结果
c4ca4238a0b923820dcc509a6f75849b
```
<br><br>


### 示例二

```java
public static void main(String args[]) {
    ScriptEngineManager manager = new ScriptEngineManager();
    // 得到javascript脚本引擎
    ScriptEngine engine = manager.getEngineByName("javascript");
    try {
        // 开始运行脚本，并返回当前的小时
        Double hour = (Double)engine.eval("var date = new Date();" +"date.getHours();");
        String msg;
        // 将小时转换为问候信息
        if (hour < 10) {
            msg = "上午好";
        }  else if (hour < 16) {
            msg = "下午好";
        } else if (hour < 20) {
            msg = "晚上好";
        } else {
            msg = "晚安";
        }
        out.printf("小时%s: %s%n", hour, msg);
    } catch (ScriptException e) {
        err.println(e);
    }
}
```
<br>
<br>


### 让脚本运行得更快

众所周知，解释运行方式是最慢的运行方式。上述的几个例子无一例外地都是以解释方式运行的。由于Java EE 6的脚本引擎可以支持任何实现脚本引擎接口的语言。有很多这样的语言提供了编译功能，也就是说，在运行脚本之前要先将这些脚本进行编译(这里的编译一般将不是生成可执行文件，而只是在内存中编译成更容易运行的方式)，然后再执行。如果某段脚本要运行之交多次的话，使用这种方式是非常快的。我们可以使用 ScriptEngine的compile方法进行编译。并不是所有脚本引擎都支持编译，只有实现了Compilable接口的脚本引擎才可以使用 compile进行编译，否则将抛出一个错误。下面的例子将演示如何使用compile方法编译并运行javascript脚本。<br>

```java
import javax.script.*;
import java.io.*;
import static java.lang.System.*;
public class CompileScript {
    public static void main(String args[]) {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("javascript");
        engine.put("counter", 0); // 向javascript传递一个参数
        // 判断这个脚本引擎是否支持编译功能
        
        if (engine instanceof Compilable) {
            Compilable compEngine = (Compilable)engine;
            
            try {
                // 进行编译
                CompiledScript script = compEngine.compile("function count() { " +
                " counter = counter +1; " +
                " return counter; " +
                "}; count();");
                out.printf("Counter: %s%n", script.eval());
                out.printf("Counter: %s%n", script.eval());
                out.printf("Counter: %s%n", script.eval());
            } catch (ScriptException e) {
                err.println(e);
            }
        } else {
            err.println("这个脚本引擎不支持编译!");
        }
    }
}


//运行结果
Counter: 1.0
Counter: 2.0
Counter: 3.0
```
<br>
<br>


### 动态调用脚本语言的方法

上面的例子只有一个函数，可以通过eval进行调用并将它的值返回。但如果脚本中有多个函数或想通过用户的输入来决定调用哪个函数，这就需要使用invoke方法进行动态调用。和编译一样，脚本引擎必须实现Invocable接口才可以动态调用脚本语言中的方法。下面的例子将演示如何通过动态调用的方式来运行上面的翻转字符串的javascript脚本。<br>

```java
import javax.script.*;
import java.io.*;
import static java.lang.System.*;
public class InvocableTest {
    public static void main(String args[]) {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("javascript");
        String name="abcdefg";
        if (engine instanceof Invocable) {
            try {
                engine.eval("function reverse(name) {" +
                " var output =' ';" +
                " for (i = 0; i <= name.length; i++) {" +
                " output = name.charAt(i) + output" +
                " } return output;}");
                Invocable invokeEngine = (Invocable)engine;
                Object o = invokeEngine.invokeFunction("reverse", name);
                out.printf("翻转后的字符串：%s", o);
            } catch (NoSuchMethodException e) {
                err.println(e);
            } catch (ScriptException e) {
                err.println(e);
            }
        } else {
            err.println("这个脚本引擎不支持动态调用");
        }
    }
}
```
<br><br>


### 动态实现接口

脚本引擎还有一个更吸引的功能，那就是动态实现接口。如我们要想让脚本异步地执行，即通过多线程来执行，那InvokeEngine类必须实现 Runnable接口才可以通过Thread启动多线程。因此，可以通过getInterface方法来使InvokeEngine动态地实现 Runnable接口。这样一般可分为3步进行。<br>

1. 使用javascript编写一个run函数<br>
   engine.eval("function run() {print(异步执行);}");<br>

2. 通过getInterface方法实现Runnable接口<br>
   Runnable runner = invokeEngine.getInterface(Runnable.class);<br>

3. 使用Thread类启动多线程<br>
   Thread t = new Thread(runner);<br>t.start();<br>


下面是实现这个功能的详细代码:<br>

```java
import javax.script.*;
import static java.lang.System.*;
public class InterfaceTest {
    public static void main(String args[]) {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("javascript");
        try {
            engine.eval("function run() {print(异步调用);}");
            Invocable invokeEngine = (Invocable)engine;
            Runnable runner = invokeEngine.getInterface(Runnable.class);
            Thread t = new Thread(runner);
            t.start();
            t.join();
        } catch (InterruptedException e) {
            err.println(e);
        } catch (ScriptException e) {
            System.err.println(e);
        }
    }
}
```

