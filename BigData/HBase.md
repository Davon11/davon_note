# HBase

​	HBase 是受 [Google Bigtable 论文 ](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/68a74a85e1662fe02ff3967497f31fda7f32225c.pdf)启发实现的，其存储原理同样来自该论文中的 SSTable、内存表及 WAL，对于这方面的粗略介绍可以查看 [数据密集型系统设计](http://ddia.vonng.com/#/ch3?id=%e9%a9%b1%e5%8a%a8%e6%95%b0%e6%8d%ae%e5%ba%93%e7%9a%84%e6%95%b0%e6%8d%ae%e7%bb%93%e6%9e%84) ，HBase 中内存表(memstore)和 Redis zset 都选择以跳跃表实现



HBase 整体架构

![image-20210708213522153](https://i.loli.net/2021/07/08/zFcANn4gxhilwHC.png)

与其他集群框架不同，HBase 中 Master 宕机并不会导致集群不可用，仍然可以获取和修改数据，Master 负责的功能则包括建表、删表移动 Region 等需要跨 RegionServer 的操作。

RegionServer 才是核心组件，RegionServer 的架构如下。

![image-20210708214123558](https://i.loli.net/2021/07/08/7yh3ItpYGq8CH1b.png)

WAL(Write-Ahead Log)

​		 预写日志，也叫做 HLog，是为内存数据意外丢失时进行灾难恢复而设立的。一个 RegionServer 中有多个 Region 但只有一个 WAL，这多个 Region 共享 WAL。当开启预写日志时，所有的对数据的变更操作(增删改)操作都将首先写入 WAL 中，然后在写入 MemStore 中，当 MemStore 超过阈值时再批量刷写到 HFile 中。这其中 WAL 和 HFile 的数据都是存储在 HDFS 上的，而 MemStore 是基于内存的。看到这里读者可能有些疑问，既然数据已经通过 WAL 在 HDFS 上写过一遍了，为什么还需要后续的操作再往 HDFS 上写一遍，而不是直接往 HFile 中写。这里的设计考量是：WAL 是一个 Hadoop Sequence File，每次进行顺序写，其速度并不慢，而HFile 中的数据是属于 HBase 的，它们是按照 rowkey 排好序的，对于一个数据库来说，批量和顺序写入对性能的影响至关重要，由于从客户端传来的数据变更请求对应的 rowkey 基本是无序的，所以先在内存中进行集散和排序，然后再写入HFile中是更优的选择，这样保证每次写入的都是有序的数据，将来读取速度更快。

​		当节点故障时，Master 会安排这上面的 Region 转移至其他可用节点，而 WAL 由多个 Region 共享，会对 WAL 进行 Region 粒度的拆分，之后各个 Region 进行数据恢复





#### HBase 架构总结

* 一个RegionServer包含多个Region，划分规则是：一个表的一段键值在一个RegionServer上会产生一个Region。不过要当一行的数据量太大了（要非常大，否则默认都是不切分的），HBase也会把你的这个Region根据列族切分到不同的机器上去。
* 一个Region包含多个Store，划分规则是：一个列族分为一个Store，如果一个表只有一个列族，那么这个表在这个机器上的每一个Region里面都只有一个Store。
* 一个Store里面只有一个Memstore。
* 一个Store里面有多个HFile（StoreFile是HFile的抽象对象，所以如果说到StoreFile就等于HFile）。每次Memstore的刷写（flush）就产生一个新的HFile出来。