# 1. redis是什么？（what）

Redis是一个开源（BSD许可），内存存储的数据结构服务器，可用作数据库，高速缓存和消息队列代理。它支持字符串、哈希表、列表、集合、有序集合，位图，hyperloglogs等数据类型。内置复制、Lua脚本、LRU收回、事务以及不同级别磁盘持久化功能，同时通过Redis Sentinel提供高可用，通过Redis Cluster提供自动分区。

Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

# 2. 为什么使用redis？（why）

## 2.1. redis的特点

1. Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
2. Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
3. Redis支持数据的备份，即master-slave模式的数据备份。

## 2.2. redis的优势

1. 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。
2. 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
3. 原子 – Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。
4. 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

## 2.3. redis与其他key-value存储有什么不同

1. Redis有着更为复杂的数据结构并且提供对他们的原子性操作，Redis的数据类型都是基于基本数据结构的同时对程序员透明，无需进行额外的抽象。
2. Redis运行在内存中但是可以持久化到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，应为数据量不能大于硬件内存。
3. 相比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单，这样Redis可以做很多内部复杂性很强的事情。
4. 在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问。

# 3. 如何使用redis？（how）

## 3.1. redis的数据类型

| 数据类型                  | 概念                                       | 常用命令               |
| --------------------- | ---------------------------------------- | ------------------ |
| String(字符串)           | key-value型                               | SET ，GET           |
| Hash(哈希)              | field-value,适用于存储对象类型（对象名-对象属性值）         | HMSET，HEGTALL      |
| List(列表)              | string类型的有序列表，按照插入顺序排序                   | lpush，lrange       |
| Set(集合)               | string类型的无序集合                            | sadd，smembers      |
| zset(sorted set：有序集合) | string类型元素的集合,且不允许重复的成员。每个元素关联一个double值来进行排序，double值可以重复但元素不能重复。 | zadd，ZRANGEBYSCORE |

## 3.2. redis常用命令
