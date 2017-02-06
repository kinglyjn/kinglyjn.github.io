---
layout: post
title:  "Spring MVC 配置文件的三个常用配置详解"
desc: "Spring MVC 配置文件的三个常用配置详解"
keywords: "Spring MVC 配置文件的三个常用配置详解,spring,java,kinglyjn"
date: 2012-12-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

Spring MVC项目中通常会有 sprng-servlet.xml 和 applicationContext.xml 二个配置文件，通常会出现以下几个配置<br>

1、`<context:annotation-config/>` <br>

它的作用是隐式地向 Spring 容器注册  AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、PersistenceAnnotationBeanPostProcessor、RequiredAnnotationBeanPostProcessor 这4个BeanPostProcessor。其作用是如果你想在程序中使用注解，就必须先注册该注解对应的类，如下图所示：<br>

```default

依赖的类                                        注解
-------------------------------------------------------------------------------------
CommonAnnotationBeanPostProcessor              @Resource 、@PostConstruct、@PreDestroy
PersistenceAnnotationBeanPostProcessor的Bean	   @PersistenceContext
AutowiredAnnotationBeanPostProcessor Bean      @Autowired
RequiredAnnotationBeanPostProcessor            @Required
-------------------------------------------------------------------------------------
```

当然也可以自己进行注册：

```xml
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor "/>
<bean class="org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor"/>
```
<br>


2、`<context:component-scan base-package="com.*" >` <br>

<context:component-scan/> 配置项不但启用了对类包进行扫描以实施注释驱动 Bean 定义的功能，同时还启用了注释驱动自动注入的功能（即还隐式地在内部注册了 AutowiredAnnotationBeanPostProcessor 和 CommonAnnotationBeanPostProcessor），因此当使用 <context:component-scan/> 后，就可以将 <context:annotation-config/> 移除了。在这里有一个比较有意思的问题，就是扫描是否需要在二个配置文件都配置一遍，我做了这么几种测试：<br>

1）只在applicationContext.xml中配置如下：<br>

```xml
<context:component-scan base-package="com.login" />
```
扫描到有@RestController、@Controller、@Compoment、@Repository等注解的类，则把这些类注册为Bean。<br>
启动正常，但是任何请求都不会被拦截，简而言之就是@Controller失效。<br>

2）只在spring-servlet.xml中配置上述配置：<br>
启动正常，请求也正常，但是事物失效，也就是不能进行回滚。<br>

3）在applicationContext.xml和spring-servlet.xml中都配置上述信息<br>
启动正常，请求正常，也是事物失效，不能进行回滚。<br>

4）在applicationContext.xml中配置如下：<br>

```xml
<context:component-scan base-package="com.login" />
```

在spring-servlet.xml中配置如下

```xml
<context:component-scan base-package="com.sohu.login.web" />
```

此时启动正常，请求正常，事物也正常了。<br>

`结论`：在spring-servlet.xml 中只需要扫描所有带@Controller注解的类，在applicationContext中可以扫描所有其他带有注解的类（也可以过滤掉带@Controller注解的类）。<br><br>


3、`<mvc:annotation-driven />` <br>

它会自动注册RequestMappingHandlerMapping 与RequestMappingHandlerAdapter （ 3.1 以后DefaultAnnotationHandlerMapping 和AnnotationMethodHandlerAdapter已经可以不用了）<br><br>

> 参考：[http://docs.spring.io/spring/docs/](http://docs.spring.io/spring/docs/)





