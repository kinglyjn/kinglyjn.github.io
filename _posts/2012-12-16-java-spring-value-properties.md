---
layout: post
title:  "通过@Value注解读取properties配置内容"
desc: "通过@Value注解读取properties配置内容"
keywords: "通过@Value注解读取properties配置内容,spring,java,kinglyjn"
date: 2012-12-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

java <br>

```java
@Controller  
@RequestMapping("/value")  
public class ValuePropertyController extends ApplicationController{  
      
    @Value("#{configProperties['jdbc.jdbcUrl']}")  
    private String jdbcUrl;   
      
    @RequestMapping  
    public String value(){  
        System.out.println(jdbcUrl);  
        return "";  
    }  
}  
```
<br>

applicationContext.xml <br>

```xml
<bean id="configProperties" class="org.springframework.beans.factory.config.PropertiesFactoryBean">  
   <property name="locations">  
       <list>  
           <value>classpath:database.properties</value>  
       </list>  
   </property>  
</bean>  
<bean id="propertyConfigurer" class="org.springframework.beans.factory.conPreferencesPlaceholderConfigurer">  
    <property name="properties" ref="configProperties"/>  
</bean>  
```
<br>

database.properties <br>

```default
jdbc.jdbcUrl=jdbc:mysql://localhost:3306/commentDemo?useUnicode=true&characterEncoding=UTF-8  
```
<br>

