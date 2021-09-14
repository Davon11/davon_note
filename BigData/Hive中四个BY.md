# order by、sort by、distirbute by、cluster by

* Hive中的order by和其他SQL语言中定义的一样。会对查询结果集进行一个全局排序。这也就是说会有一个所有数据都通过一个reducer进行处理的过程。

* sort by只会在每个reducer中对数据进行排序，也就是说这是一个局部排序的过程。这可以保证每个reducer的输出数据都是有序的(局部有序，全局无序)。

* distribute by控制map的输出在reducer中如何划分。默认情况下，MapReduce框架会根据map输入的键计算相应的hash值，然后按照hash值将key-value均匀的分发到多个reducer中。distribute by可以保证对应的记录会分发到同一个reducer中处理，但不保证相同的key会出现在相邻的位置，例如：

  ```
  // 有如下5条记录，2个reducer任务：
  x1
  x2
  x4
  x3
  x1
  
  // reducer1
  x1
  x2
  x1
  
  // reducer2
  x4
  x3
  ```

* 如果在一个HQL中，distribute by的字段和sort by的字段完全相同，而且采用**升序**排列(asc，默认的排序方式)，这种情况下 `distribute a,b sort by a asc,b asc` 就可以缩写成 `cluster by a,b`。



ps: 每个reduce task会生成一个文件。distribute by和cluster by只能指定记录分配reduce task的key，而不能指定reduce task的个数。设置hive的map与reduce task个数见：[Hive设置map和reduce的个数](https://blog.csdn.net/B11050101/article/details/78754652) 	 [Hive设置map和reduce的个数](https://app.yinxiang.com/shard/s59/nl/30120116/dc958565-0bcd-46c1-9f7a-349b5678f077)