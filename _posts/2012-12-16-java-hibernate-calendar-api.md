---
layout: post
title:  "Hibernate Calendar API"
desc: "Hibernate Calendar API"
keywords: "Hibernate Calendar API,Calendar,hibernate,java,kinglyjn"
date: 2012-12-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

### java.util.Calendar

Calendar 的 month 从 0 开始，也就是全年 12 个月由 0 ~ 11 进行表示。
Calendar.DAY_OF_WEEK 定义和值如下：<br>

* Calendar.SUNDAY = 1
* Calendar.MONDAY = 2
* Calendar.TUESDAY = 3
* Calendar.WEDNESDAY = 4
* Calendar.THURSDAY = 5
* Calendar.FRIDAY = 6
* Calendar.SATURDAY = 7

<br>

SimpleDateFormat 的格式定义<br>

```default
Letter	 Date or Time Component	           Presentation	    Examples
-------------------------------------------------------------------------------
G        Era designator	                    Text            AD
y        Year                               Year            1996; 96
Y        Week year                          Year            2009; 09
M        Month in year (context sensitive)  Month           July; Jul; 07
L        Month in year (standalone form)    Month	        July; Jul; 07
w        Week in year                       Number          27
W        Week in month                      Number          2
D        Day in year                        Number          189
d        Day in month                       Number          10
F        Day of week in month               Number          2
E        Day name in week                   Text            Tuesday; Tue
u        Day number of week(1=Monday,,7=Sunday)	Number      1
a        Am/pm marker                       Text            PM
H        Hour in day (0-23)                 Number          0
k        Hour in day (1-24)                 Number          24
K        Hour in am/pm (0-11)               Number          0
h        Hour in am/pm (1-12)               Number          12
m        Minute in hour                     Number          30
s        Second in minute                   Number          55
S        Millisecond                        Number          978
z        Time zone                      General time zone   Pacific Standard Time; PST; GMT-08:00
Z        Time zone                      RFC 822 time zone   -0800
X        Time zone                      ISO 8601 time zone  -08; -0800; -08:00
-------------------------------------------------------------------------------
```
<br>


Java Calendar 演示代码如下<br>

```java
import java.util.Date;  
import java.text.SimpleDateFormat;  
import java.text.DateFormat;  
import java.text.ParseException;  
import java.util.Calendar;  
  
public class Test  
{  
  public Test()  
  {  
  }  
  
  public static void main(String[] args)  
  {  
    // 字符串转换日期格式  
    // DateFormat fmtDateTime = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
    // 接收传入参数  
    // String strDate = args[1];  
    // 得到日期格式对象  
    // Date date = fmtDateTime.parse(strDate);  
  
    // 完整显示今天日期时间  
    String str = (new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS")).format(new Date());  
    System.out.println(str);  
  
    // 创建 Calendar 对象  
    Calendar calendar = Calendar.getInstance();  
  
    try  
    {  
      // 对 calendar 设置时间的方法  
      // 设置传入的时间格式  
      SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-M-d H:m:s");  
      // 指定一个日期  
      Date date = dateFormat.parse("2013-6-1 13:24:16");  
      // 对 calendar 设置为 date 所定的日期  
      calendar.setTime(date);  
  
      // 按特定格式显示刚设置的时间  
      str = (new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS")).format(calendar.getTime());  
      System.out.println(str);  
    }  
    catch (ParseException e)  
    {  
      e.printStackTrace();  
    }  
  
    // 或者另一種設置 calendar 方式  
    // 分別爲 year, month, date, hourOfDay, minute, second  
    calendar = Calendar.getInstance();  
    calendar.set(2013, 1, 2, 17, 35, 44);  
    str = (new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS")).format(calendar.getTime());  
    System.out.println(str);  
  
    // Calendar 取得当前时间的方法  
    // 初始化 (重置) Calendar 对象  
    calendar = Calendar.getInstance();  
    // 或者用 Date 来初始化 Calendar 对象  
    calendar.setTime(new Date());  
  
    // setTime 类似上面一行  
    // Date date = new Date();  
    // calendar.setTime(date);  
  
    str = (new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS")).format(calendar.getTime());  
    System.out.println(str);  
  
    // 显示年份  
    int year = calendar.get(Calendar.YEAR);  
    System.out.println("year is = " + String.valueOf(year));  
  
    // 显示月份 (从0开始, 实际显示要加一)  
    int month = calendar.get(Calendar.MONTH);  
    System.out.println("nth is = " + (month + 1));  
  
    // 本周几  
    int week = calendar.get(Calendar.DAY_OF_WEEK);  
    System.out.println("week is = " + week);  
  
    // 今年的第 N 天  
    int DAY_OF_YEAR = calendar.get(Calendar.DAY_OF_YEAR);  
    System.out.println("DAY_OF_YEAR is = " + DAY_OF_YEAR);  
  
    // 本月第 N 天  
    int DAY_OF_MONTH = calendar.get(Calendar.DAY_OF_MONTH);  
    System.out.println("DAY_OF_MONTH = " + String.valueOf(DAY_OF_MONTH));  
  
    // 3小时以后  
    calendar.add(Calendar.HOUR_OF_DAY, 3);  
    int HOUR_OF_DAY = calendar.get(Calendar.HOUR_OF_DAY);  
    System.out.println("HOUR_OF_DAY + 3 = " + HOUR_OF_DAY);  
  
    // 当前分钟数  
    int MINUTE = calendar.get(Calendar.MINUTE);  
    System.out.println("MINUTE = " + MINUTE);  
  
    // 15 分钟以后  
    calendar.add(Calendar.MINUTE, 15);  
    MINUTE = calendar.get(Calendar.MINUTE);  
    System.out.println("MINUTE + 15 = " + MINUTE);  
  
    // 30分钟前  
    calendar.add(Calendar.MINUTE, -30);  
    MINUTE = calendar.get(Calendar.MINUTE);  
    System.out.println("MINUTE - 30 = " + MINUTE);  
  
    // 格式化显示  
    str = (new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SS")).format(calendar.getTime());  
    System.out.println(str);  
  
    // 重置 Calendar 显示当前时间  
    calendar.setTime(new Date());  
    str = (new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SS")).format(calendar.getTime());  
    System.out.println(str);  
  
    // 创建一个 Calendar 用于比较时间  
    Calendar calendarNew = Calendar.getInstance();  
  
    // 设定为 5 小时以前，后者大，显示 -1  
    calendarNew.add(Calendar.HOUR, -5);  
    System.out.println("时间比较：" + calendarNew.compareTo(calendar));  
  
    // 设定7小时以后，前者大，显示 1  
    calendarNew.add(Calendar.HOUR, +7);  
    System.out.println("时间比较：" + calendarNew.compareTo(calendar));  
  
    // 退回 2 小时，时间相同，显示 0  
    calendarNew.add(Calendar.HOUR, -2);  
    System.out.println("时间比较：" + calendarNew.compareTo(calendar));  
  }  
}  
```

要计算时间差，可用 Calendar.getTimeInMillis() 取得两个时间的微秒级的时间差，再加以换算即可，比如获得相差天数，代码如下：<br>

```java
long val = calendarEnd.getTimeInMillis() - calendarBegin.getTimeInMillis();  
// 换算后得到天数  
long day = val / (1000 * 60 * 60 * 24); 
```
<br><br>


### 使用hibernate映射Calendar

<img src="http://img.blog.csdn.net/20170119155618260?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br>



