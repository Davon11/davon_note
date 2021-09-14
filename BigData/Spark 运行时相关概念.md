# Spark 运行时相关概念



## 一些主要的名词概念

**从集群的物理层面**

* Master 节点：部署 Cluster Manager的节点

* Slave 节点：部署 Worker 的节点，每个节点可以有多个 Worker 进程

**从进程层面(与所执行的应用无关)**

* Cluster Manager：管理集群的 CPU、内存等资源，为不同的应用分配所需的资源

* Worker： 接受 Cluster Manager 的调度安排，分配具体的资源给应用程序，生成 Executor。每个Worker可以有多个Executor，但默认值为1

**具体的APP执行**

* Driver：向集群提交 Application， 执行其中创建 SparkContext 的 mian 函数的进程
* Executor：执行切分好的 task ，并把结果缓存在节点的内存和磁盘上。每个 Executor 上可以有多个 task

**APP 执行概念的划分**

* job：每一个 action 算子会生成一个 job
* stage：每个 job 会被划分为多个 stage，划分依据是否发生了 shuffle（或者说宽窄依赖）
* task：被分配到 Executor 上执行的单位工作内容，一般有多少个 partition 就会有多少个task



## Spark 程序的一些对象

**SparkConf、SparkContext、SparkSession 和 StreamingContext**

* SparkConf：Spark运行的配置对象
* SparkContext：Driver 和集群进行连接和通信的上下文，RDD 编程的入口，Spark 中使用的大多数操作/方法或函数都来自 SparkContext，例如累加器、广播变量、并行化等等
* SparkSession：Spark 新的入口，内部封装了 SparkContext，其实计算也都由 SparkContext 完成，当需要使用 Spark SQL、Hive、DataFream、DataSet 时应使用 SparkSession 为入口
* StreamingContext：Spark Streming 的入口，内部封装了 SparkContext，Stream 相当于 unbound 的 RDD





## Spark 运行模式

**本地模式**

* 本地启动*多线程* 模拟集群工作，提供单机测试坏境，用于验证逻辑的正确性
* 提交时用参数`--master local[N]`启动

**本地集群模式**

* 本地启动*多进程* 模拟集群工作
* 提交时用`--master local-cluster[num_excutors, excutor_cores, excutor_memory]`启动

**StandAlone 模式**

* 自带的集群部署模式，不依赖其他资源调度框架，但资源的调度上不够灵活，适用于项目早期快速部署，通常配合 zk 配置 master 高可用
* 提交参数 `--master spark://ip:port`

**Spark On Yarn**（Mesos与之类似）

* 集群部署模式，借助 Yarn 进行资源管理
* `--master yarn`

在使用集群部署时还可以使用参数`--deploy-mode client\cluster`设置选用`client`还是`cluster`模式，

* client 模式：在提交App的节点启动 driver，App运行过程中该节点不可离线且应该能够与集群正常通信。此模式下可以在提交任务的终端上看到输出，应该只在调试和测试时使用此模式
* cluster 模式：driver 启动在集群中的某一节点，App 提交后节点可以离线，正常生产中使用此模式





### 参考资料

\- [1] [Spark 应用提交指南](https://colobu.com/2014/12/09/spark-submitting-applications/)
\- [2] [Cluster vs Client: Execution modes for a Spark application](https://blog.knoldus.com/cluster-vs-client-execution-modes-for-a-spark-application/)

