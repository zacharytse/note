[TOC]
# Redis是什么
Redis是一种NoSql数据库，支持分布式，<key,value>存储系统，C/S通讯模型，单进程单单线程模型。
## 应用场景
缓存热点数据，计数器，限流器，发布订阅、排行榜，分布式锁，共享session,队列
# Redis的类型
共五种
- String(可以包含任何数据)
- List
底层是链表
- Hash(重要，类似Map<String,Object>)
- Set
通过HashTable实现
- Zset
每个元素都会关联一个double类型的分数，通过分数为集合中的成员进行从小到大的排序

# 常用命令
- scard:获取集合中元素个数
- srem:删除集合中的元素
- smembers:枚举集合元素
- sadd:向集合中添加元素
- srandmemeber:随机出几个元素

# 跳表实现(重点)
当我们的有序集合的节点个数>128或者节点的member长度大于64的时候，采用跳表，否则用ziplist(字节数组)