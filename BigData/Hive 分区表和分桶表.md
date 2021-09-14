# Hive 分区表、分桶表和分桶抽样



## 一、使用场景

​	分区表和分桶表都是为大规模数据集查询优化而设计出来了，通过在物理存储文件上的划分，防止查询时的全表扫描提高查询效率。实际生产中分区表使用较多，分桶表由于在建表时固定了桶的个数，不适合于持续不断膨胀的表，故使用较少，但仍有分区、分桶同时使用的。



## 二、创建表

​	分区表：

```sql
create table partition_table(
    id int,
    name string,
    age int,
    p int
)
partitioned by (dt string)
row format delimited
fields terminated by '\t'
lines terminated by '\n'
```

​	分桶表：

```sql
create table bucket_test(
    id int,
    name string,
    age int,
    p int
)
clustered by (age) into 4 buckets
row format delimited
fields terminated by '\t'
lines terminated by '\n'
```

​	分区分桶表创建同时使用``partitioned by`和 `clustered by`即可。



## 三、向表插入数据

​	文件形式(本地或HDFS)插入：

```sql
load data [local] inpath 'file_path' overwrite/insert into table table_name [partition(partiton_field = value)];
```

​	SQL 插入：

```sql
insert [overwrite] into table table_name [partition(partiton_field = value)] select ...;
```

分桶表默认只能使用 SQL 形式插入，如果想要使用文件形式需要更改参数设置`set hive.strict.checks.bucketing=false
hive.mapred.mode=nonstrict`



## 四、分桶函数

​		计算分桶编号的方法是用分桶字段值 HashCode 对总桶数取余。一般用于对数据进行样本抽样。**非分桶表也可以使用分桶函数抽样**。

​		分桶函数关键字为 `tablesample(bucket x out of y on bucket_field)` 它表示对分桶字段 bucket_field 使用分桶函数分桶，总共分 y 桶，最后取 x 桶里的数据，x 为1基（即从1开始到 y 结束)。分桶函数分出来的和分桶表本身已经有的桶完全无关，亲测网络上的 y 必须是分桶表本身分桶的因数说法错误。

![image-20210421205332467](https://i.loli.net/2021/07/08/p5elGoZ8MdgIY1R.png)



## 五、总结

1、表创建时，分区字段不是表字段，而分桶字段必须是表字段；

2、HDFS 上的存储不同

​	不同分区在不同的子目录中

![image-20210421202021500](https://i.loli.net/2021/07/08/Ss27qaRwEKpgeuh.png)

​	不同的分桶数据则是在不同的文件中，其中 000000_0_copy_1 000000_0_copy_2 和 000000_0 同属一个桶，是分属该桶的后续添加的数据。

![image-20210421202125757](https://i.loli.net/2021/07/08/2zwAG3TrkXOya4D.png)

-----------

附：`hive.mapred.mode`参数的作用

​	其值有 `nonstrict`和`strict`两种，设为`strict`时 hive 将拒绝包括以下三种 SQL 的执行

1、对于分区表，不加分区字段进行查询，不能执行。
2、对于order by语句必须使用limit语句。
3、限制笛卡尔积的查询（join的时候不适用on，而使用where的）

因为它们都会造成任务只有一个 reducer ，造成性能瓶颈。