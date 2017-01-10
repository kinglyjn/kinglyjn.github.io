---
layout: post
title:  "字符串操作及正则表达式（regex）"
desc: "字符串操作及正则表达式（regex）"
keywords: "java网络编程基础,java,kinglyjn"
date: 2012-11-17
categories: [Java]
tags: [java]
icon: fa-coffee
---

### java常见字符串操作

代码示例：<br>

```java
package regex;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class RegexDemo {
    public static void main(String[] args) {
        //testMatches();
        //testReplace();
        //testGetSubstring();
        testSplit();
    }
    
    //测试匹配
    public static void testMatches() {
        String str = "asdasdasdsadasdasdasssssss";
        System.out.println(str.matches(".*ssssss.*"));
    }
    
    //测试替换
    public static void testReplace() {
        String str = "窗前明月光，asdasdad asdad";
        str = str.replaceAll("[\u4e00-\u9fa5]", "c");
        System.out.println(str);
    }
    
    //测试萃取
    public static void testGetSubstring() {
        String str = "100,2000, asd9090 898窗前明月光，asdasdad asdad";
        Pattern pattern = Pattern.compile("\\d+");
        Matcher matcher = pattern.matcher(str);
        while (matcher.find()) {
            System.out.println(matcher.group());
        }
    }
    
    //测试分割
    public static void testSplit() {
        String str = "100,2000, asd9090 898窗前明月光，asdasdad asdad";
        String[] strs = str.split("\\d+");
        for (int i = 0; i < strs.length; i++) {
            System.out.println(strs[i]);
        }
    }
}
```
<br>


