# Hadoop



## Yarn

##### 调度器

社区实现的 Yarn 调度器有两种: Capacity Scheduler 和 Fair Scheduler，两者均支持多队列，同时在其他队列资源空闲时均可以进行队列间的资源共享，在生产中都有应用。

![image-20210705214731234](https://i.loli.net/2021/07/05/PzALH7YuVCUfOZy.png)

Capacity Scheduler：可为每个队列分配资源上限和下限（百分比），当 job 1 submit 时，即便队列 B 没有 job 运行，job1 也不会使用到集群所有的资源，这样当 job2 提交后就能够立即运行（如果它能得到的资源足够的话）。Capacity Scheduler 支持多队列，在同一个队列中采用的时FIFO策略。

<img src="https://i.loli.net/2021/07/05/84gmeEBwUbfiyPT.png" alt="image-20210705215510318" style="zoom: 67%;" />

Fair Scheduler：如图所示，当 队列 B 为空时，job1 可以使用集群的全部资源，当队列 B 的 job2 提交时，job2 不能立即得到需要的资源，需要等待 job1 释放资源后才能启动，当同属于队列 B 的 job3 提交后，job3 就会和 job2 分别得到 1/4 资源，资源在两个队列间实现公平。