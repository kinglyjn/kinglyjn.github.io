---
layout: post
title:  "js中的日期"
desc: "js中的日期"
keywords: "js中的日期,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [js]
icon: fa-html5
---

> 由于Date()是js原生函数，不同浏览器的解析器对其实现方式并不同，
> 所以返回值也会有所区别。本文测试未特别申明浏览器的情况下，均是
> 指win7 x64+chrome 44.0.2403.155 (正式版本) m（32 位）
> 版本。



### Date()与new Date()的区别

Date()直接返回当前时间字符串，不管参数是number还是任何string

```js
Date();
Date('sssss');
Date(1000);
//Fri Aug 21 2015 15:46:21 GMT+0800 (中国标准时间)
```

而new Date()则是会根据参数来返回对应的值，无参数的时候，返回当前时间的字符串形式；有参数的时候返回参数所对应时间的字符串。new Date()对参数不管是格式还是内容都要求,且只返回字符串。

```js
new Date();
//Fri Aug 21 2015 15:51:55 GMT+0800 (中国标准时间)

new Date(1293879600000);
new Date('2011-01-01T11:00:00')
new Date('2011/01/01 11:00:00')
new Date(2011,0,1,11,0,0)
new Date('jan 01 2011,11 11:00:00')
new Date('Sat Jan 01 2011 11:00:00')
//Sat Jan 01 2011 11:00:00 GMT+0800 (中国标准时间)

new Date('sss');
new Date('2011/01/01T11:00:00');
new Date('2011-01-01-11:00:00')
new Date('1293879600000');
//Invalid Date

new Date('2011-01-01T11:00:00')-new Date('1992/02/11 12:00:12')
//596069988000
```

从上面几个测试结果可以很容易发现

1. new Date()在参数正常的情况只会返回当前时间的字符串(且是当前时区的时间)
2. new Date()在解析一个具体的时间的时候，对参数有较严格的格式要求，格式不正确的时候会直接返回Invalid Date，比如将number类的时间戳转换成string类的时候也会导致解析出错
3. 虽然new Date()的返回值是字符串，然而两个new Date()的结果字符串是可以直接相减的，结果为相差的毫秒数。

那么，new Date()能接受的参数格式到底是什么标准呢?（相对于严格要求的多参数传值方法。非严格的单参数(数字日期表示格式)更常用且更容易出错，所以下文只考虑单参数数字时间字符串转换的情况)



### new Date()解析所支持的参数格式标准

**时间戳格式**

这个是最简单的也是最不容易出错的。当然唯一的缺点大概就是对开发者不直观，无法一眼看出具体日期。需要注意的以下两点:

```shell
1. js内的时间戳指的是当前时间到1970年1月1日00:00:00 UTC对应的毫秒数，和unix时间戳不是一个概念，后者表示秒数，差了1000倍
2. new Date(timestamp)中的时间戳必须是number格式，string会返回Invalid Date。所以比如new Date('11111111')这种写法是错的
```

**时间数字字符串格式**

不大清楚这种该怎么描述，就是类似YYYY/MM/DD HH:mm:SS这种。下文以dateString代指。
new Date(dateString)所支持的字符串格式需要满足RFC2822标准或者ISO 8601标准
这两种标准对应的格式分别如下:

RFC2822 标准日期字符串

```js
YYYY/MM/DD HH:MM:SS ± timezon(时区用4位数字表示)
// eg 1992/02/12 12:23:22+0800
```

> RFC2822还有别的格式，不过上面这个是比较常用的（另外这标准太难啃了，实在没耐心啃完，
> 所以也就没太深入)。RFC2822标准本身还有其他的非数字日期表达方式，不过不在这个话题
> 讨论范围内了,略过

ISO8601标准日期字符串

```js
YYYY-MM-DDThh:mm:ss ± timezone(时区用HH:MM表示)

1997-07-16T08:20:30Z
// “Z”表示UTC标准时区，即"00:00",所以这里表示零时区的`1997年7月16日08时20分30秒`
//转换成位于东八区的北京时间则为`1997年7月17日16时20分30秒`

1997-07-16T19:20:30+01:00
// 表示东一区的1997年7月16日19时20秒30分，转换成UTC标准时间的话是1997-07-16T18:20:30Z
```

> 1. 日期和时间中间的T不可以被省略，一省略就出错。
> 2. 虽然在chrome浏览器上时区也可以用+0100这种RFC2822的形式来表示，然而IE上不支持
>    这种混搭写法，所以用ISO8601标准形式表示的时候时区要用+HH:MM

单单从格式上来说，两者的区别主要在于分隔符的不同。不过需要注意的是，ISO 8601标准的兼容性比RFC2822差得多（比如IE8和iOS均不支持前者。我知道IE8很多人会无视，不过iOS也有这个坑的话，各位或多或少会谨慎点了吧？)，所以一般情况下建议用RFC 2822格式的。

不过需要注意的是，在未指定时区的前提下，对于只精确到day的日期字符串，RFC 2822返回结果是以当前时区的零点为准，而ISO8601返回结果则会以UTC时间的零点为标准进行解析。

例如：

```js
//RFC2822：
new Date('1992/02/13') //Thu Feb 13 1992 00:00:00 GMT+0800 (中国标准时间)
//ISO8601:
new Date('1992-02-13') //Thu Feb 13 1992 08:00:00 GMT+0800 (中国标准时间)
```

然而上面这个只是ES5的标准而已，在ES6里这两种形式都会变成当前时区的零点为基准1
以对于时间字符串对象，个人意见是要么用RFC2822形式，要么自己写个解析函数然后随便你传啥格式进来。


> 附录：mongo js example

```js
// 在脚本文件中可以直接访问db变量，以及其他全局变量，然而shell辅助函数（语法糖，
// 比如use db、以及show collections）不可以在脚本中使用，这些辅助函数都有对应的js函数
-------------------------------------------
辅助函数                等价函数
-------------------------------------------
use foo               db.getSisterDB("foo")
show dbs              db.getMongo().getDbs()
show collections      db.getCollectionNames()
-------------------------------------------


// 创建js文件 vim /home/ubuntu/mongo-shell/test01.js
var count = db.user.find().count();

for (var i=0; i<count; i++) {
    var userId = db.user.find()[i]._id
    var userName = db.user.find()[i].name;

    var address = db.address.find()[i];
    delete address._id;

    db.user.update({_id:userId}, {$set:{name:userName+i, address:address}});
}

//在mongo客户端交互窗口执行js文件：
> load("/home/ubuntu/mongo-shell/test01.js")

//或者在shell窗口直接执行命令
$ mongo --quiet /home/ubuntu/mongo-shell/test01.js
$ mongo --quiet localhost:27017/test /home/ubuntu/mongo-shell/test01.js




//连接到指定的数据库脚本，并且将db指向这个连接
//defineConnectTo.js
var connectTo = function(port, dbname){
    if (!port) {
        port = 27017;
    }

    if (!dbname) {
        dbname = "test";
    }

    db = connect("localhost:"+port+"/"+dbname);
    return db;
}

//如果在shell中加载这个脚本，connectTo函数就可以直接使用了
> typeof connectTo
undefined
> load('defineConnectTo.js')
> typeof connectTo
function
```

