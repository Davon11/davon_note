FlinkCDC











## 坑

* Flink 开启CK时默认重启策略为：延迟重启，Integer.MAX_VALUE 次，间隔时间 1s。

* 使用 MySQL source 时参数错误将不会有额外的报错，配合上一条，将导致无限重试而不报原因

  ```scala
  MySQLSource.builder()
        .hostname("hadoop103").port(3306)
        .username("root").password("123456")
        .databaseList("gmall_2022").tableList("").build()
  // 务必确保这6个参数正确(如果有的话)
  // 如果打印出 Connected to hadoop103:3306 at mysql-bin.000003/120 (sid:5442, cid:195) 类似信息代表连接上 mysql 实例，请确保是正确的实例
  ```

* Windows 本地调试，程序中使用到 hdfs 或其他 Hadoop 组件，报如下错误，安装 winutils 并设置环境变量即可

  ```
  Could not locate executablenull\bin\winutils.exe in the Hadoop binaries
  ```


* Scala 编写 Flink 侧输出流，报错

  ```
  val outputTagDate = OutputTag[String]("Date-side-output")。程序写完编译的时候报
  type mismatch :
   found   : org.apache.flink.streaming.api.scala.OutputTag[String]
   required: org.apache.flink.util.OutputTag[java.io.Serializable]
  ```

  1. 检查 OutputTag 和 输出到侧输出流的元素类型是否一致
  2. 引入 java OutputTag，`import org.apache.flink.util.OutputTag`

* 