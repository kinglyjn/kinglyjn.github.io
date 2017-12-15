---
layout: post
title:  "Hbase调优（七）"
desc: "Hbase调优（七）"
keywords: "Hbase的表（五）,hbase,kinglyjn"
date: 2017-08-26
categories: [Java]
tags: [java]
icon: fa-coffee
---



### 服务端优化手段

**1、虚拟机内存优化** <br>
```default
java -Xms256m -Xmx512m -XX:NewSize=xxm -XX:MaxNewSize=xxm
-XX:+UseParNewGC //使用并行年轻代垃圾回收
```

**2、MemStore和 blockcache优化** <br>
调整MemStore限制：<br>
hbase.regionserver.global.memstore.upperLimit/lowerLimit<br>

upperlimit说明：<br>
hbase.hregion.memstore.flush.size 这个参数的作用是当单个Region内所有的memstore大小总和超过指定值时，flush该region的所有memstore。RegionServer的flush是通过将请求添加一个队列，模拟生产消费模式来异步处理的。那这里就有一个问题，当队列来不及消费，产生大量积压请求时，可能会导致内存陡增，最坏的情况是触发OOM。这个参数的作用是防止内存占用过大，当ReigonServer内所有region的memstores所占用内存总和达到heap的[40%]时，HBase会强制block所有的更新并flush这些region以释放所有memstore占用的内存。<br>

lowerLimit说明：<br>
同upperLimit，只不过lowerLimit在所有region的memstores所占用内存达到Heap的[35%]时，不flush所有的memstore。它会找一个memstore内存占用最大的region，做个别flush，此时写更新还是会被block。lowerLimit算是一个在所有region强制flush导致性能降低前的补救措施。在日志中，表现为 “** Flush thread woke up with memory above low water.”<br>

调整块缓存的大小：<br>
控制堆中块缓存大小的属性是一个用浮点数表示的百分比值，默认是20%（即0.2），改变 perf.hfile.block.cache.size属性可以改变这个百分比值。客户端如果读请求很多的话可以考虑增加块缓存的大小，以帮助用户缓存更多的查询数据。<br>

> 注意：块缓存和memstored的上限不能超过堆内存的100%，用户需要为其他操作保留空间，不然服务器端可能面临内存紧张的问题。默认他们占用的堆空间量是60%（memstore占0.4，blockcache占0.2），这是一个合理的值。<br>

​		
增加阻塞倍率：<br>
属性 hbase.hregion.memstore.block.multiplier 的默认值为2，他是一个用于阻塞来自客户端更新数据请求的安全阈值，当memstore达到属性 multiplier乘以hbase.hregion.memstore.flush.size 的大小限制时会阻塞进一步的更新。当有足够的存储空间时，用户可以增加这个值来更加平滑的处理【突发流量的写入】，这时候用户可以临时接收更多的数据。<br>

理想情况下，在不超过hbase.regionserver.global.memstore.upperLimit的情况下，Memstore应该尽可能多多的使用内存（配置给Memstore部分的，为而不是真的heap的 ）。下图在展示了一张“较好的情况”：<br>

<img src="http://img.blog.csdn.net/20171215184154339?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/>	

**3、启用压缩** <br>
lzo --> snappy 确保所有的rs安装了压缩类库 <br>

```shell
$hbase> create 'test01:t1', {NAME=>'d', COMPRESSION=>'snappy'}, {NAME='i', COMPRESSION=>'gz'}
```

​	
**4、避免切割风暴**<br>
与其依赖hbase自动管理拆分，用户还不如关闭这个行为然后手动调用split和major_compact命令，用户可以通过设置这个集群的hbase.hregion.max.filesize值或者在列族级别上把表模式中对应参数设置成非常大的值来完成。为防止手动拆分无法运行，最好不要将其设置为Long.MAX_VALUE，最好能将这个值设置为一个合理的上限，如100G。手动命令拆分和压缩region的好处是可以对他们进行时间控制，在不同regions上交错运行，这样可以尽可能分散I/O负载，并且避免拆分/合并风暴。<br>
​	
**5、预拆分区域和热点区域问题**<br>
小region对split和compaction友好，因为拆分region或compact小region里的storefile速度很快，内存占用低。缺点是split和compaction会很频繁。特别是数量较多的小region不停地split, compaction，会导致集群响应时间波动很大，region数量太多不仅给管理上带来麻烦，甚至会引发一些Hbase的bug。一般512以下的都算小region。大region，则不太适合经常split和compaction，因为做一次compact和split会产生较长时间的停顿，对应用的读写性能冲击非常大。此外，大region意味着较大的storefile，compaction时对内存也是一个挑战。当然，大region也有其用武之地。如果你的应用场景中，某个时间点的访问量较低，那么在此时做compact和split，既能顺利完成split和compaction，又能保证绝大多数时间平稳的读写性能。<br>
内存方面，小region在设置memstore的大小值上比较灵活，大region则过大过小都不行，过大会导致flush时app的IO等待增加，过小则因store file过多影响读性能。<br>
​		
既然split和compaction如此影响性能，有没有办法去掉？<br>
compaction是无法避免的，split倒是可以从自动调整为手动。<br>
只要通过将这个参数值调大到某个很难达到的值，比如100G，就可以间接禁用自动split（RegionServer不会对未到达100G的region做split）。再配合RegionSplitter这个工具，在需要split时，手动split。手动split在灵活性和稳定性上比起自动split要高很多，相反，管理成本增加不多，比较推荐online实时系统使用。

```default	
a.客户端创建表的时候进行预拆分
  通过split指令进行预分区；
  hbase.hregion.max.filesize设置在当前ReigonServer上单个Reigon的最大存储空间，单个Region超过该 
  值时，这个Region会被自动split成更小的region；
b.mv等相关指令进行跨区域regionserver移动
c.hash散列化rowkey
```

**6、合并区域**<br>
当用户删除里大量数据并且想减少每个服务器管理的region数目，这时候可以使用merge指令合并两个region<br>
​	
**7、负载均衡**<br>
balance_switch、balancer、balancer_enabled

<br>



### 客户端优化手段

1、禁止自动刷写<br>
当有大批量的操作时，使用htable#setAutoFlush(false)方法禁用自动刷写，改为手动提交，防止Put实例逐个被传递到服务器。通过htable#add(put)添加的实例都会添加到一个相同的写入缓存中，如果用户禁用了自动刷写，这些操作直到写缓存区被填满或手动提交htable#flushCommits() 时才会被送出。<br>
​	
2、使用扫描缓存（限制一次扫描的行数）<br>
如果hbase被用作一个mapreduce作业的输入源，最好将作为mapreduce作业输入扫描器实例的缓存用san#setCaching()方法设置为比1大的多的值，使用默认的-1意味着map任务在处理每一条记录时都会请求region服务器，例如将这个值设置为500，则一次可以传送500行数据到客户端处理。这里用户需要权衡传输数据的开销和内存的开销，因为缓存增大以后，无论是客户端还是服务器端都将消耗更多的内存缓存资源，所以缓存并不是越大越好。<br>
​	
3、限定扫描范围（限制一次扫描的列数）<br>
当scan被用来处理大量的行时（特别是用作mapreduce的输入源时），注意那些属性被选中了。如果Scan.addFamily()被调用了，纳特特定列族中的所有列都将返回到客户端。如果只是处理少数的列，则应当只添加这些列到scan的输入中。另外对于列数很多的情况，可以通过scan#setBatch()设置一次rpc调用返回的列数，这类似于scan#setCaching方法<br>
​	
4、关闭ResultScanner<br>
ResultScanner#close 虽不会带来性能的提升，但会避免可能的资源消耗<br>
​	
5、块缓存的优化<br>
Scan#setCacheBlocks()用来设置使用region服务器中的块缓存，如果mapreduce作业中使用扫描，这个方法应该设置成false。对于那些频繁访问的行，建议使用块缓存。<br>
​	
6、优化获取行键的方式<br>
当执行一个表的扫描以获取需要的行键时（没有列族、列名、列值和时间戳），在scan中用setFilter方法添加一个带 MUST_PASS_ALL 操作符的FilterList。FilterList中包含FirstKeyFilter和KeyOnlyFilter两个过滤器，这将会把发现的第一个KeyValue行键返回给客户端，最大限度地减少网络带宽。<br>
​	
7、关闭Put上的WAL<br>
Put#setDurability(Durability.SKIP_WAL);，这样服务端就不会把这个put写到WAL中，而只会把它写到memstore中，不过一旦region服务器出现故障后就会丢失数据，对于一些不太重要的数据可以考虑此优化措施。<br>

<br>



### 配置优化手段

1、zk的超时设定<br>
zookeeper.session.timeout=180-<br>
​	
2、增加处理线程<br>
hbase.regionserver.handlerc.count=10+<br>
​	
3、设置hbase堆大小<br>
hbase-env.sh#HBASE_HEAPSIZE=1G<br>