---
layout: post
title:  "常见框架单例、多例与线程安全性总结"
desc: "常见框架单例、多例与线程安全性总结"
keywords: "常见框架单例、多例与线程安全性总结,kinglyjn,张庆力"
date: 2016-12-15
categories: [Java]
tags: [frame]
icon: fa-bookmark-o
---

单例与多例问题是指，当多个用户访问某个类时，系统是为每个用户创建一个该类实例，还是整个系统无论多少用户访问，只创建一个该类实例。<br>

线程安全问题是指，多个用户同时在访问同一个程序时，其对于某一数据的修改，会不会影响到其他用户中的该数据。若没有影响，则是线程安全的;若有可能影响，则是线程不安全的。<br>

现在对 HttpServlet、HttpSession、SpingMVC、Struts2 中的 Action、Hibernate 中的 SessionFactory与 Session，进行总结。
<br><br>

(1) HttpServlet<br>

其是单例的。即无论多少用户访问同一个业务，如 LoginServlet，Web 容器只会创建一个该 Servlet 实例。而该实例是允许多用户访问的。<br>

若 Servlet 中包含成员变量，则每个用户对于成员变量的修改，均会影响到其他用户所看到的该变量的值，所以这时是线程不安全的。若不包含成员变量，则是线程安全的。<br><br>


(2) HttpSession<br>

其是多例的。Web 容器会为每个用户开辟一个 Session，多个用户会有多个 Session。而每个用户只能访问自己的 Session。所以，对于 Session 来说，就不存在并发访问的情况，也就不存在线程安全的问题了。所以可以说是线程安全的。<br><br>


(3) SpingMVC Controller<br>
 Spring MVC Controller默认是单例的：<br>
 单例的原因有二：<br>
 1、为了性能。<br>
 2、不需要多例。<br>
 如果需要多例，则需要在Controller类上加注解 @Scope("prototype")<br><br>

  

(4) Struts2 的 Action<br>

其是多例的。对于同一个业务，例如 LoginAction，系统会为每一个用户创建一个LoginAction 的实例，并使其成员变量 username 与 password 接收用户 交的数据。同一用户只能访问自己的 Action。所以，对于 Action 来说，就不存在并发访问的情况，也就不存在线程安全的问题了。所以可以说是线程安全的。 <br><br>



(5) Hibernate 的 SessionFactory<br>

其是单例的。无论多少用户访问该项目，系统只会创建一个 SessionFactory 对象，即这个对象是可以被所有用户访问的。<br>

SessionFactory实现类中所包含的成员变量基本都是 final常量，即任何用户均不能修改。所以，也就不存在用户的修改对其他用户的影响问题了，所以是线程安全的。 <br><br>


(6) Hibernate 的 Session<br>

其是多例的。系统会为每个用户创建一个 Session。<br>

Session 的实现类中定义了很多的非 final 成员变量，一个事务对成员变量所做的修改，会影响到另一个事务对同一数据的访问结果，所以是线程不安全的。 <br><br>

