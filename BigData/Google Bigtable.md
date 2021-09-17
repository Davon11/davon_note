# Google Bigtable 简介

Bigtable 本质是一个分布式 K-V 数据库，其主要目标就是数据的高性能读写。

## 分布式

​	Bigtable 将整个大的表水平划分为多个 Tablet ，每个 Tablet 负责一定范围内的 Key 对应的数据，Bigtable 的服务节点称为 Tablet Server，每个 Tablet Server 中有若干个 Tablet，Bigtable 依赖于 Chubby（与 Zookeeper 类似），通过 Chubby 提供的能力 Bigtable 可以：1. 确保任意时刻只有一个 Master 节点；2.  存放 Tablet Server 信息、 Tablet 和 Tablet Server 的映射、每张表的列列族信息在 Chubby 中；3. 当 Tablet Server 和 Master 节点出现故障下线时，可以及时通知集群进行后续处理(备份的Master 节点转正、Tablet 转移等)。

## 读写设计

**写**：设立写缓存 Memtable(内存表)，当客户端请求写入数据时直接写入内存中的 Memtable 后就返回成功，写入的数据在 Memtable 中将按照 Key 排序，当内存表中写入的数据达到一定大小后，将数据刷写到磁盘磁盘文件中，称之为 SSTable ， SSTable 一旦写入完成后续将不会变更其中的数据（只会在合并后删除），后续如果有对同一个 Key 的 Value 的变更或删除也只会是新的 SSTable 中的一条记录(删除也是一条数据，只是它的类型是delete，称之为墓碑标记)，读取将只返回最新的 SSTable 中的 Key 的记录。SSTable 是一个持久化的、排序的、不可更改的 Map 结构。由于同一个 Key 可能在多个 SSTable 中出现，但只有最新的数据是有效的，为了减少磁盘空间和读取需要查找的文件数，将在后台对这些文件段进行合并。同时为了防止在刷写磁盘之前掉电丢失数据，设立了 WAL(Write-Ahead Log，预写日志)，WAL 是磁盘上的一个只追加文件，数据写入 Memtable 之前首先写入 WAL。这样的设计带来一些好处：

* 数据的新增、变更和删除都是一条新增的记录，会写入 WAL 和 Memtable 中，WAL 是对磁盘的追加写，Memtable 在内存中，这样的写入速度要远快于对磁盘的随机写；
* 通过 Memtable 进行写数据的集散，能够一次写入多条数据，减少寻址时间，同时如果某条数据先写入随后即删除，那么通过 Memtable 的集散它就不用写入这条数据到磁盘；
* 在 Memtable 中对数据按 Key 排序，这样能够保证每一个文件段都是内部有序的。于是我们在内存中维护索引时，不需要对每个 Key 都建立索引，只需要建立稀疏的索引即可，因此相同的内存索引空间可以对应更大的磁盘，存储更多的数据。同样在合并文件段时，由于每个文件段都是有序的，可以使用归并排序进行合并。
* 由于文件段的不可变更的特性，不需要考虑读写时文件段锁的问题，更好的支持了高并发。

Bigtable 的所有持久化存储均由 GFS（一种分布式文件系统，Hadoop 的 HDFS 是其 GFS 的开源实现），这样可以保证持久化存储层的高吞吐与高可用，同时，当某个 Tablet Server 故障时，其余的 Tablet Server 通过存储在 GFS 上的 WAL 和 SSTable 可以快速恢复原 Tablet Server 上的所有 Tablet。



**读**：设立读缓存 ，Bigtable 有 Scan Cache 和 BlockCache两级缓存，Scan Cache 缓存的是刚刚Scan的数据，对于经常要重复读取相同数据的应用程序来说，Scan Cache 非常有效；Block Cache 缓存的是从 GFS 读取的 SSTable 的 Block，Block 是 Bigtable 最小的存储单位， 当经常要读取刚刚读过的数据附近的数据时，BlockCache 很有用。如果读缓存未命中，需要扫描 Memtable，然后配合内存中的稀疏索引依次从最新的 SSTable 扫描到旧的 SSTable。从这里的描述中可以看出，由于索引是稀疏索引，所以当请求读取一个不存在的 Key 时，将会导致 Bigtable 需要扫描所有的 SSTable 后才能确定无此 Key，因此，Bigtable 引入了 Boom filter（布隆过滤器），通过 Boom 过滤器能够迅速的排除大部分的对不存在的 Key 的读取请求。



### 参考资料

\- [1] [Google Bigtable 论文 ](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/68a74a85e1662fe02ff3967497f31fda7f32225c.pdf)
