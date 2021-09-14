# Flink layered APIs

Flink 的 API 主要分为如下三层抽象：

<img src="https://flink.apache.org/img/api-stack.png" alt="img" style="zoom: 33%;" />



## DataStream API

### 基于时间和窗口的算子

#### 流 join

​	Flink 为流 join 提供了3个算子 `join`、`coGroup`、`intervalJoin`。

​	join 和 coGroup 都是基于窗口的，在同一个窗口中的两流元素才能 join 起来，对被分配至两个窗口的元素，即使他们之间的时间之隔 1ms 同样无法 join。join 算子功能类似于 inner join 两者都有才会收集，否则丢弃，coGroup 则类似于 full out join，都会收集。编程模型如下：

```scala
stream
	.join(otherStream)			// 或 coGroup(otherStream)
    .where(<KeySelector>)		// 指定 stream 的key
    .equalTo(<KeySelector>)		// 指定 otherStream 的key
    .window(<WindowAssigner>)	// 指定窗口分配器，滑动、滚动和会话窗口
    .apply()

```

​	可通过在其后的 process 函数中将其改造为 left join、right join 或 inner join [参考案例](https://blog.csdn.net/lmalds/article/details/52085794)

​	intervalJoin 则是根据时间范围进行 join，intervalJoin 同样类似于 inner join。编程模型如下：

```scala
stream
  .keyBy(<KeySelector>)
  .intervalJoin(otherStream.keyBy(<KeySelector>))	// intervalJoin 作用在两个 keyedStream 上
  .between(lowerBound, upperBound)					// 指定上下边界
  .process(ProcessJoinFunction)
```

​	Flink 目前没有提供基于时间范围的 left、 right 、out join。如果需要的话可以参考 intervalJoin 自行实现，[interval join的实现原理](https://www.jianshu.com/p/45ec888332df)