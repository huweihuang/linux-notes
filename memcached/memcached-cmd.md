# 1. Memcached 命令

## 1.1. 存储命令

### 1.1.1. 常用命令

| 命令    | 说明       |
| ------- | ---------- |
| set     | 新增或更新 |
| add     | 新增       |
| replace | 替换       |
| append  | 在后面追加 |
| prepend | 在前面追加 |
| cas     | 检查并设置 |

以上几个命令语法格式相似，以`set`为例：

```bash
set key flags exptime bytes [noreply] 
value 
```

参数说明如下：

- **key：**键值 key-value 结构中的 key，用于查找缓存值。
- **flags**：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- **exptime**：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）
- **bytes**：在缓存中存储的字节数
- **noreply（可选）**： 该参数告知服务器不需要返回数据
- **value**：存储的值（始终位于第二行）（可直接理解为key-value结构中的value）

实例：

- key → runoob
- flag → 0
- exptime → 900 (以秒为单位)
- bytes → 9 (数据存储的字节数)
- value → memcached

```bash
set runoob 0 900 9
memcached
STORED

get runoob
VALUE runoob 0 9
memcached

END
```

输出：

如果数据设置成功，则输出：

```
STORED
```

输出信息说明：

- **STORED**：保存成功后输出。
- **ERROR**：在保存失败后输出。

### 1.1.2. cas命令

Memcached CAS（Check-And-Set 或 Compare-And-Swap） 命令用于执行一个"检查并设置"的操作。

它仅在当前客户端最后一次取值后，该key 对应的值没有被其他客户端修改的情况下， 才能够将值写入。

检查是通过cas_token参数进行的， 这个参数是Memcach指定给已经存在的元素的一个唯一的64位值。

语法：

> 比以上命令多了一个`unique_cas_token`

```bash
cas key flags exptime bytes unique_cas_token [noreply]
value
```

参数说明如下：

- **key**：键值 key-value 结构中的 key，用于查找缓存值。
- **flags**：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- **exptime**：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）
- **bytes**：在缓存中存储的字节数
- **unique_cas_token**通过 gets 命令获取的一个唯一的64位值。
- **noreply（可选）**： 该参数告知服务器不需要返回数据
- **value**：存储的值（始终位于第二行）（可直接理解为key-value结构中的value）

**unique_cas_token**通过**gets**命令获取。

## 1.2. 查找命令

| 命令      | 说明                                          |
| --------- | --------------------------------------------- |
| get       | 获取一个或多个key                             |
| gets      | 获取一个或多个cas token                       |
| delete    | 删除已存在的key                               |
| incr/decr | 对已存在的 key(键) 的数字值进行自增或自减操作 |

## 1.3. 统计命令

| 命令        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| stats       | 用于返回统计信息例如 PID(进程号)、版本号、连接数等。         |
| stats items | 用于显示各个 slab 中 item 的数目和存储时长(最后一次访问距离现在的秒数)。 |
| stats slabs | 用于显示各个slab的信息，包括chunk的大小、数目、使用情况等。  |
| stats sizes | 用于显示所有item的大小和个数。                               |
| flush_all   | 用于清理缓存中的所有 **key=>value(键=>值)** 对。             |

参考文章：

- http://www.runoob.com/memcached/memcached-tutorial.html
