# 美团 Leaf 文章阅读笔记

文章链接：[Leaf——美团点评分布式ID生成系统](https://tech.meituan.com/2017/04/21/mt-leaf.html)

生成业务唯一id的常见解决方案:

1、使用 uuid 

​		128位，一般以 32 个16进制数表示，本地生成无序，可以保证全局唯一性，使用较多的是由 60 位的时间戳和 48 位 MAC 地址组成的版本。



2、snowflake 

​		官方是64位，可以根据需要调整。分布式的唯一ID生成方案，有递增趋势，不依赖第三方数据库。但要注意时钟回拨的问题，解决方案有：1. 检测到回拨，等待回拨时间过去后再提供服务；2. 将部分 bit 分配给时钟回拨，当发生回拨时该标识段进行对应变化；3. 将服务单独放在少量服务器上，关闭这些服务器的 NTP(Network Time Protocol) 等。



3、数据库生成

​		以 MySQL 举例：创建一个用来生成 ID 的表 `Tickets64`

```sql
CREATE TABLE `Tickets64` (
     `id` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT,	-- 设置 id 自增
     `stub` CHAR(1) NOT NULL DEFAULT '',
     PRIMARY KEY (`id`),
     UNIQUE INDEX `stub` (`stub`)	-- 设置 stub 为唯一索引
)
    COLLATE='utf8_general_ci'
    ENGINE=MyISAM;

-- 插入起始值
INSERT INTO Tickets64(stub) values ('A');
```

​		每次业务使用以下 SQL 得到 ID

```sql
begin;
-- REPLACE的运行与INSERT很相似。只有一点例外，假如表中的一个旧记录与一个用于PRIMARY KEY或一个UNIQUE索引的新记录具有相同的值，则在新记录被插入之前，旧记录被删除。
REPLACE INTO Tickets64 (stub) VALUES ('a');
SELECT LAST_INSERT_ID();	-- MySQL LAST_INSERT_ID 函数 获取 AUTO_INCREMENT 列的最后生成的序列号
commit;
-- 需要以事务方式执行
```

​		还可以分表并分别设置`auto_increment_increment`和`auto_increment_offset`来进行分布式架构扩展服务性能。



4、Leaf方案实现

​		美团的 Leaf 有两个方案，分别是 2 与 3 的改良。[GitHub地址](https://github.com/Meituan-Dianping/Leaf)