---
layout: post
title:  "Spring Aspect Demo（spring切面示例）"
desc: "Spring Aspect Demo（spring切面示例）"
keywords: "Spring Aspect Demo（spring切面示例）,spring,aspect,java,kinglyjn"
date: 2012-11-18
categories: [Java]
tags: [spring,aspect]
icon: fa-leaf
---

applicationContext.xml

```xml
<!-- 配置扫描组件包 【com.sprdatajpa**】,通过注解定义bean, 默认同时也通过注解自动注入 --> 
<context:component-scan base-package="com.sprdatajpa"/>
<!-- 启用@AspectJ风格的切面声明 -->
<aop:aspectj-autoproxy proxy-target-class="true"/>
```
<br>


MyAspect.java

```java
package com.sprdatajpa.aspect;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

/**
 * @author kingly
 * @date   2015年6月7日
 * 使用AspectJ风格的切面声明测试
 * 
 */
@Component
@Aspect
public class MyAspect {
 
 /** 
     * 声明一个切入点（包括切入点表达式和切入点签名） 
     */  
    @Pointcut("execution(* com.sprdatajpa.service..*.*(..))")  
    public void pointcut1(){}  
      
    
    /** 
     * 声明一个前置通知 
     */  
    @Before(value="pointcut1()")
    public void beforeAdvide(JoinPoint point) {  
        System.out.println("触发了前置通知！");
        System.out.println(point.getTarget()); //获取被代理对象[service对象]
        Object[] args = point.getArgs();       //获取被代理对象方法的传入参数[service对象方法的传入参数]
        for (Object arg : args) {
   System.out.println(arg); 
  }
    } 
    
    
    /**
     * 声明一个后置通知
     */
    @After(value="pointcut1()")
    public void afterAdvice(JoinPoint point) {
     System.out.println("触发了后置通知，抛出异常也会被触发！");  
    }
    
    
    /** 
     * 声明一个返回后通知 
     */ 
    @AfterReturning(value="pointcut1()", returning="ret")
    public void afterReturningAdvice(JoinPoint point, Object ret) {
      System.out.println("触发了返回后通知，抛出异常时不被触发，返回值为：" + ret);
    }
    
    
    /** 
     * 声明一个异常通知
     */  
    @AfterThrowing(pointcut="pointcut1()", throwing="throwing")  
    public void afterThrowsAdvice(JoinPoint point, RuntimeException throwing){  
        System.out.println("触发了异常通知，抛出了RuntimeException异常！"); 
        System.out.println("异常堆栈信息如下：");
        throwing.printStackTrace();
    } 
    
    
    /** 
     * 声明一个环绕通知
     * 不管是其前置和后置通知，总是最先执行，异常抛出后后置通知不再执行
     */
    @Around(value="pointcut1()")
    public Object aroundAdvice(ProceedingJoinPoint point) throws Throwable {
     System.out.println("触发了环绕通知的开始~~~");
     Object o = point.proceed();
     System.out.println("触发了环绕通知的结束~~~");
     return o; //经过加工的对象
    }
}
```
<br>
