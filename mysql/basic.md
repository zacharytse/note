[TOC]
# 主从复制
指数据可以从一个mysql主节点复制到一个或多个从节点 。

**三种复制类型**
- 同步复制：用于银行，证券等要求数据事务处理一致性的单位。master收到所有slave节点的ack，才会commit
- 异步复制(mysql默认操作)：主机完成写操作后，直接commit，不等待slave的ack
  ![](https://gitee.com/zacharytse/image/raw/master/img/20201223201851.png)
  存在failover的情况，不能保证主从节点的数据一致性
- 半同步模式
  介于同步与异步之间的，master收到一个slave的ack，就直接commit,否则等待直到超时转为异步操作
  ![](https://gitee.com/zacharytse/image/raw/master/img/20201223202038.png)

## 用途
- 读写分离
  sql语句锁表时，该表不能被读。使用主从复制，**主库负责写**，从库负责读。即使主库被锁，从库还能读
- 数据实时备份，当系统中某个节点发生故障时，可以方便的故障切换
- 高可用HA
- 架构扩展
  将负载分布在多个从节点上，降低单机I/O的频率

## 形式
- 一主一从
- 一主多从(提供读性能)
- 多主一从(5.7开始支持)
  从节点可以是存储性能比较好的服务器
- 双主复制(自己既是master,也是slave)
- 级联复制
  ![](https://gitee.com/zacharytse/image/raw/master/img/20201223200733.png)
  可以缓解主节点压力，也不会影响数据一致性

## 原理
![](https://gitee.com/zacharytse/image/raw/master/img/20201223200854.png)
一共有3个线程
1. 1个log dump thread(主节点上运行)
   负责记录master的二进制更新日志，每创建一个从节点，master就会创建一个log dump thread
2. I/O thread(从节点) 
   读取master的二进制更新日志，并写入relay log
3. SQL thread(从节点)
   从relay log中读取操作，并执行
要实施复制，就需要打开master端的binary log(bin-log)功能
## bin-log格式
mysql有3种主从复制方式，分别对应3种bin-log文件
- 基于SQL语句的复制对应STATEMENT
  记录sql语句在binlog中，mysql5.1.4以及之前的版本使用的。优点:记录量少，节约IO。缺点:可能会因为sleep()，now()等语句造成主从节点数据不一致。
- 基于行的复制对应ROW
  只记录被修改的数据。优点：可以保证主从节点的数据一致性。缺点：产生大量的bin-log日志，也无法通过bin-log解析获取执行过的sql语句。
- 混合模式复制对应MIXED
  mysql ndb cluster 7.3和7.4使用。前面2种方式的混合版本，mysql会根据执行的sql语句选择合适的方式。适合用sql语句保存的操作就用sql语句保存，否则只保存修改过后的数据。

## GTID复制(mysql5.6以后的新特性)
Global Transaction IDentifier,全局事务标识，具有全局唯一性，一个事务对应一个GTID

在传统的复制中，主从切换时，服务器需要找到binlog和pos点，并设置新的slave为master开始复制。mysql5.6后会自动匹配GTID断点，不再寻找binlog和pos，只需要提供原master的ip，端口，账号密码。
### GTID组成
GTID=source_id:transaction_id。
- source_id:产生GTID的服务器
- transaction_id是每台mysql server从1开始自增的事务序列号。每产生一个事务，该序号都会自增1。
  
### 产生方式
其产生受GTID_NEXT控制。master中的GTID_NEXT默认值是AUTOMATIC，即每次事务commit都会产生一个GTID，并在该事务执行前，将GTID写入bin-log。slave从binlog读取该GTID，并用该GTID标识之后要执行的事务。

> GTID是不会重复的，拥有相同GTID的事务只会执行一个

### 使用步骤
- master更新数据，产生GTID，并在**执行前**写入bin-log
- slave的I/O thread读取GTID，并写入relay log
- SQL Thread从relay log中读取GTID，对比master的bin-log中是否有记录
- 有记录，则该GTID已经执行，slave忽略
- 没有记录(可能是别的slave产生的GTID)，SQL Thread执行该GTID事务，并记录到slave的bin-log中

# 隔离级别
## ACID
- 原子性
每一个事务之间的操作，要么一起完成，要么就不做
- 一致性
事务A修改的值，事务A接下来调用这个值时，要保证值和之前的修改是相同的
- 隔离性
事务之间的操作是相互隔离，互不影响的
- 持久性
事务操作完成后数据不会丢失

## 隔离性带来的问题
- 脏读
事务B修改了数据，但还没提交，此时事务A读取了事务B修改后的数据，这种称为脏读
- 不可重复读
事务B修改或者删除了数据，并且提交了，此时事务A读取了事务B修改后的数据，这种称为不可重复读
- 幻读
事务B插入了新数据，并且提交了，此时事务A读取了事务B新插入的数据，这种称为幻读。
## 解决方案
![](https://gitee.com/zacharytse/image/raw/master/img/20201230160432.png)
READ COMMITTED是Oracle默认使用的隔离级别，REPEATABLE READ是mysql使用的默认级别
> 注意：串行化是读写按顺序进行，所以完全避免了脏读，不可重复读和幻读，但这种实现方案并行效果差，所以一般不采用
### mysql具体实现
**只有InnoDB是支持事务的，InnoDB中实现了行锁和表锁来解决这些问题(LBCC:Lock-Based Concurrent Control)**

锁的分类

![](https://gitee.com/zacharytse/image/raw/master/img/20201230162431.png)

- 行锁
  - 共享锁(读锁)
  - 排它锁(写锁)
  - 一个事务加了读锁，其他事务也可以接着加读锁，但如果有一个事务加了写锁，其他事务就不能加锁了
- 表锁
  - 意向锁:意向共享锁，意向排它锁。这两个锁用来判断表中有没有加行锁，以此来提高加表锁的效率
```
锁定粒度:表锁>行锁
加锁效率:表锁>行锁
冲突概率:表锁>行锁
并发性能:表锁<行锁
```
- 锁的算法
  - 记录锁 where id = xxx,对xxx这个数据所在行进行上锁
  - 间隙锁 将表中的数据按区间进行划分，并且该区间全为开区间
  - 临键锁 记录锁与临键锁的合体版，即开区间变为闭区间

![](https://gitee.com/zacharytse/image/raw/master/img/20201230162747.png)

**MVCC:Multi-Version Concurrent Control**

多版本并行控制。
MySQL为每一个执行的事务都会赋予一个transaction_id。每一行的数据是有多个版本组成的，格式为row=transaction_id。事务在对每一行数据进行操作时，会将该行数据的最大transaction_id与自身事务的transaction_id进行比较，如果前者较大，则找下一个版本的数据，反之更新该行数据

# Mysql索引

## 索引实现
InnoDB和MyIsam采用B+树实现，理由如下:
> 先比较B+树和普通的二叉树。对于普通的平衡二叉树而言，因为每一个节点也只能有2个孩子，树的深度可能远大于B+树，因此二叉树的IO查找次数要比B+树多的多。
再来比较B+树和B树，B+树和B树最大的不同是B树的每个节点下都会存储data，而B+树只有叶子节点会存储data。考虑极端情况，data过大，导致一个节点只能保存1个key时，B树退化为二叉树了，而B+树非叶节点只保存索引值，因此B+树效率高。同时由于有链表结构，只需要找到首尾就可以通过链表将所有数据取出

Memory引擎采用Hash实现

## 索引的分类
- 主键索引(特殊的唯一索引)
- 唯一索引
- 普通索引
- 组合索引
- 全文索引

**为什么建表时一定要指定主键(为什么必须创建主键索引)**
> 不创建主键索引，执行sql时mysql加的是表锁而不是行锁，并发执行效率低

## InnoDB和MYISAM索引的区别
- 两者都采用B+树实现，但InnoDB的叶子节点中的data存储的是数据，而MyIsam叶子节点中的data存储的是数据的地址
- InnoDB的数据和索引都放在idx文件中存储，是聚簇索引(有且只有一个)，而MyISAM索引和数据分别存放在MYI和MYD文件中，是非聚簇索引

## 索引的用处
- 快速查找匹配WHERE字句的行
- 如果表具有多列索引，则优化器可以使用索引的任何最左前缀来查找行(最左匹配)
- 当有表连接的时候，从其他表检索行
- 查找特定索引列的min和max值
- 可以优化查询以检索值而无需查询数据行

## 一些常见名词
- 回表
使用普通索引查找时，先定位主键值，再定位行记录(两次查找B+树)
- 覆盖索引
只需要在一棵索引树上就能获取到SQL所需的数据，无需回表
- 最左匹配
where条件子句按照组合索引建立的顺序进行匹配，如索引<a,b>,select * from table_name where a = xxx and b = xxx会先比较索引a再比较索引b，而select * from table_name where b = xxx不会触发索引
- 索引下推
多条件查询时，将索引列由server向下沉，减少查找的结果









