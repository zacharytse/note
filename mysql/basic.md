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
