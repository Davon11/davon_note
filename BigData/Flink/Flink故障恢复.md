# Flink 故障恢复

​		当 task 故障时，Flink 需要重启该 task 以使 job 能够恢复到正常执行状态。

​		Flink 通过**重启策略**和**故障恢复策略**来控制 Task 重启：重启策略决定是否可以重启以及重启的间隔；故障恢复策略决定哪些 Task 需要重启。

## Restart Strategies

​		Flink 的 Restart Strategies 分为集群和job两个级别。Flink 集群的 Restart Strategies 在 flink-conf.yaml 文件中配置，如果没有配置，那么 CheckPoint 设置为关闭(默认)时，使用 No Restart Strategy ；CK 开启时使用 Fixed Delay Restart Strategy ，重试次数为 Integer.MAX_VALUE，间隔为 1s。

​		Flink 作业可以在`ExecutionEnvironment` 对象上调用 `setRestartStrategy` 方法来设置单独的重启策略。

![image-20210725004401687](https://i.loli.net/2021/07/25/TnIDRAOQjswHM8E.png)

​		RestartStrategyConfiguration 的四个子类：

* NoRestartStrategyConfiguration： 不重启

* FixedDelayRestartStrategyConfiguration：固定延迟重启，重启次数超过指定值时彻底失败

* FallbackRestartStrategyConfiguration(默认)： 使用集群重启策略，当 job 未设置重启策略时使用此策略。

* FailureRateRestartStrategyConfiguration：失败率重启
  ```java
  public static FailureRateRestartStrategyConfiguration failureRateRestart(
  			int failureRate, Time failureInterval, Time delayInterval) {
  		...
  	}
  // 在 job 失败后重启，但连续的两次重启之间会间隔 delayInterval，如果在 failureInterval 时间内重启次数超过 failureRate 次彻底失败。
  ```

* Exponential Delay Restart Strategy(1.13版本计划加入)：无限次重启，job 永远不会彻底失败。最开始两次重试之间的间隔会以指数增长，达到设置的最大上限后以该值为间隔。

  ## Failover Strategies 

  * Restart All Failover Strategy：全部重启

  * Restart Pipelined Region Failover Strategy: 部分重启

### 参考资料

\- [1] [Flink 官方文档: task故障恢复](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/execution/task_failure_recovery/#failure-rate-restart-strategy)

