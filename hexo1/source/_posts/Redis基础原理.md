---
title: Redis基础原理
date: 2018-07-04
tags: 
categories: 框架

---



### Redis基础原理 ###

[TOC]

#### 1.概念 ####

**NoSQL**

非关系型数据库,Redis是非关系型数据库中的**键值存储数据库**

**应用**

处理高并发/海量数据的访问,内容缓存

**优点**

1. 快速查询,支持横向扩充(集群)和纵向扩充(加强设备)
2. 一主多从,读写分离
3. 哨兵机制,检测选举
4. 集群机制,多主多从,数据高可用,分布式存储

**缺点**

​    存储缺少结构化(难以构建关系型理数据库模型)



#### 2.数据类型 ####

##### String #####

最简单的KV存储,value可以是String也可以是数字等

**场景**

kv字符串结构等,非常普遍

**指令**

​	SET key value                   设置key=value 

​	GET key                         或者键key对应的值 

​	GETRANGE key start end          得到字符串的子字符串存放在一个键 

​	GETSET key value                设置键的字符串值，并返回旧值 

​	GETBIT key offset               返回存储在键位值的字符串值的偏移 

​	MGET key1 [key2..]              得到所有的给定键的值 

​	SETBIT key offset value         设置或清除该位在存储在键的字符串值偏移 

​	SETEX key seconds value         键到期时设置值 

​	SETNX key value                 设置键的值，只有当该键不存在 

​	SETRANGE key offset value       覆盖字符串的一部分从指定键的偏移 

​	STRLEN key                      得到存储在键的值的长度 

​	MSET key value [key value...]   设置多个键和多个值 

​	MSETNX key value [key value...] 设置多个键多个值，只有在当没有按键的存在时 

​	PSETEX key milliseconds value   设置键的毫秒值和到期时间 

​	INCR key                        增加键的整数值一次 INCRBY key increment            由给定的数量递增键的整数值 

​	INCRBYFLOAT key increment       由给定的数量递增键的浮点值

​	DECR key                        递减键一次的整数值 

​	DECRBY key decrement            由给定数目递减键的整数值 

​	APPEND key value                追加值到一个键 

​	--------操作管理----------

- DEL key                         如果存在删除键 

- DUMP key                        返回存储在指定键的值的序列化版本 

- EXISTS key                      此命令检查该键是否存在 

- EXPIRE key seconds              指定键的过期时间

- EXPIREAT key timestamp          指定的键过期时间。在这里，时间是在Unix时间戳格式 

- PEXPIRE key milliseconds        设置键以毫秒为单位到期 

- PEXPIREAT key milliseconds-timestamp        设置键在Unix时间戳指定为毫秒到期 

- KEYS pattern                    查找与指定模式匹配的所有键 

- MOVE key db                     移动键到另一个数据库 

- PERSIST key                     移除过期的键 

- PTTL key                        以毫秒为单位获取剩余时间的到期键。 

- TTL key                         获取键到期的剩余时间。 

- RANDOMKEY                       从Redis返回随机键 

- RENAME key newkey               更改键的名称 

- RENAMENX key newkey             重命名键，如果新的键不存在 

- TYPE key                        返回存储在键的数据类型的值。 




##### List(列表) #####

字符串列表,非常重要的Redis类型,本质是双向链表,支持反向查询和遍历,更加方便,但是会额外增加内存开销(存储双向链表),redis内部的**发送缓冲队列**使用的就是List结构

**场景**

如twitter的关注列表和粉丝列表,实现轻量级的消息队列等

**指令**

- BLPOP
  BLPOP key1 [key2 ] timeout 取出并获取列表中的第一个元素，或阻塞，直到有可用
- BRPOP
  BRPOP key1 [key2 ] timeout 取出并获取列表中的最后一个元素，或阻塞，直到有可用
- BRPOPLPUSH
  BRPOPLPUSH source destination timeout 从列表中弹出一个值，它推到另一个列表并返回它;或阻塞，直到有可用
- LINDEX
  LINDEX key index 从一个列表其索引获取对应的元素
- LINSERT
  LINSERT key BEFORE|AFTER pivot value 在列表中的其他元素之后或之前插入一个元素
- LLEN
  LLEN key 获取列表的长度
- LPOP
  LPOP key 获取并取出列表中的第一个元素
- LPUSH
  LPUSH key value1 [value2] 在前面加上一个或多个值的列表
- LPUSHX
  LPUSHX key value 在前面加上一个值列表，仅当列表中存在
- LRANGE
  LRANGE key start stop 从一个列表获取各种元素
- LREM
  LREM key count value 从列表中删除元素
- LSET
  LSET key index value 在列表中的索引设置一个元素的值
- LTRIM
  LTRIM key start stop 修剪列表到指定的范围内
- RPOP
  RPOP key 取出并获取列表中的最后一个元素
- RPOPLPUSH
  RPOPLPUSH source destination 删除最后一个元素的列表，将其附加到另一个列表并返回它
- RPUSH
  RPUSH key value1 [value2] 添加一个或多个值到列表
- RPUSHX
  RPUSHX key value 添加一个值列表，仅当列表中存在



##### Hash #####

Redis Hash对应Value内部实际就是一个HashMap，实际这里会有2种不同实现，这个Hash的成员比较少时Redis为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的HashMap结构，对应的value redisObject的encoding为zipmap,当成员数量增大时会自动转成真正的HashMap,此时encoding为ht。 

**场景**

可以方便的存储用户信息,用户ID为Key,用户信息序列化为value存储

**指令**

- HDEL
  HDEL key field[field...] 删除对象的一个或几个属性域，不存在的属性将被忽略
- HEXISTS
  HEXISTS key field 查看对象是否存在该属性域
- HGET
  HGET key field 获取对象中该field属性域的值
- HGETALL
  HGETALL key 获取对象的所有属性域和值
- HINCRBY
  HINCRBY key field value 将该对象中指定域的值增加给定的value，原子自增操作，只能是integer的属性值可以使用
- HINCRBYFLOAT
  HINCRBYFLOAT key field increment 将该对象中指定域的值增加给定的浮点数
- HKEYS
  HKEYS key 获取对象的所有属性字段
- HVALS
  HVALS key 获取对象的所有属性值
- HLEN
  HLEN key 获取对象的所有属性字段的总数
- HMGET
  HMGET key field[field...] 获取对象的一个或多个指定字段的值
- HSET
  HSET key field value 设置对象指定字段的值
- HMSET
  HMSET key field value [field value ...] 同时设置对象中一个或多个字段的值
- HSETNX
  HSETNX key field value 只在对象不存在指定的字段时才设置字段的值
- HSTRLEN
  HSTRLEN key field 返回对象指定field的value的字符串长度，如果该对象或者field不存在，返回0.
- HSCAN
  HSCAN key cursor [MATCH pattern] [COUNT count] 类似SCAN命令



##### Set #####

存储数据不重复,set 的内部实现是一个 value永远为null的HashMap，实际就是通过计算hash的方式来快速排重的，这也是set能提供判断一个成员是否在集合内的原因。 

**场景**

set和list的功能类似,但是set加载的列表自动排重,当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。 

在微博应用中，每个用户关注的人存在一个集合中，就很容易实现求两个人的共同好友功能。 

**指令**

- SADD
  SADD key member [member ...] 添加一个或者多个元素到集合(set)里
- SACRD
  SCARD key 获取集合里面的元素数量
- SDIFF
  SDIFF key [key ...] 获得队列不存在的元素
- SDIFFSTORE
  SDIFFSTORE destination key [key ...] 获得队列不存在的元素，并存储在一个关键的结果集
- SINTER
  SINTER key [key ...] 获得两个集合的交集
- SINTERSTORE
  SINTERSTORE destination key [key ...] 获得两个集合的交集，并存储在一个集合中
- SISMEMBER
  SISMEMBER key member 确定一个给定的值是一个集合的成员
- SMEMBERS
  SMEMBERS key 获取集合里面的所有key
- SMOVE
  SMOVE source destination member 移动集合里面的一个key到另一个集合
- SPOP
  SPOP key [count] 获取并删除一个集合里面的元素
- SRANDMEMBER
  SRANDMEMBER key [count] 从集合里面随机获取一个元素
- SREM
  SREM key member [member ...] 从集合里删除一个或多个元素，不存在的元素会被忽略
- SUNION
  SUNION key [key ...] 添加多个set元素
- SUNIONSTORE
  SUNIONSTORE destination key [key ...] 合并set元素，并将结果存入新的set里面
- SSCAN
  SSCAN key cursor [MATCH pattern][COUNT count] 迭代set里面的元素

##### Sorted Set #####

set的有序版,由HaspMap和跳跃表组成

**场景**

用户的积分排行榜需求就可以通过有序集合实现。还有上面介绍的使用List实现轻量级的消息队列，其实也可以通过Sorted Set实现有优先级或按权重的队列。 

**指令**

- ZADD
  ZADD key score1 member1 [score2 member2] 添加一个或多个成员到有序集合，或者如果它已经存在更新其分数
- ZCARD
  ZCARD key 得到的有序集合成员的数量
- ZCOUNT
  ZCOUNT key min max 计算一个有序集合成员与给定值范围内的分数
- ZINCRBY
  ZINCRBY key increment member 在有序集合增加成员的分数
- ZINTERSTORE
  ZINTERSTORE destination numkeys key [key ...] 多重交叉排序集合，并存储生成一个新的键有序集合。
- ZLEXCOUNT
  ZLEXCOUNT key min max 计算一个给定的字典范围之间的有序集合成员的数量
- ZRANGE
  ZRANGE key start stop [WITHSCORES] 由索引返回一个成员范围的有序集合（从低到高）
- ZRANGEBYLEX
  ZRANGEBYLEX key min max [LIMIT offset count]返回一个成员范围的有序集合（由字典范围）
- ZRANGEBYSCORE
  ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 返回有序集key中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员，有序集成员按 score 值递增(从小到大)次序排列
- ZRANK
  ZRANK key member 确定成员的索引中有序集合
- ZREM
  ZREM key member [member ...] 从有序集合中删除一个或多个成员，不存在的成员将被忽略
- ZREMRANGEBYLEX
  ZREMRANGEBYLEX key min max 删除所有成员在给定的字典范围之间的有序集合
- ZREMRANGEBYRANK
  ZREMRANGEBYRANK key start stop 在给定的索引之内删除所有成员的有序集合
- ZREMRANGEBYSCORE
  ZREMRANGEBYSCORE key min max 在给定的分数之内删除所有成员的有序集合
- ZREVRANGE
  ZREVRANGE key start stop [WITHSCORES] 返回一个成员范围的有序集合，通过索引，以分数排序，从高分到低分
- ZREVRANGEBYSCORE
  ZREVRANGEBYSCORE key max min [WITHSCORES] 返回一个成员范围的有序集合，以socre排序从高到低
- ZREVRANK
  ZREVRANK key member 确定一个有序集合成员的索引，以分数排序，从高分到低分
- ZSCORE
  ZSCORE key member 获取给定成员相关联的分数在一个有序集合
- ZUNIONSTORE
  ZUNIONSTORE destination numkeys key [key ...] 添加多个集排序，所得排序集合存储在一个新的键
- ZSCAN
  ZSCAN key cursor [MATCH pattern] [COUNT count] 增量迭代排序元素集和相关的分数



**Redis过期策略:https://www.jb51.net/article/103236.htm





#### 3.Redis运用 ####

实际开发中,经常会用到集合(list读取较快)来存储缓存数据,但是Redis只支持List<String>,不能直接支持List<Object>,因此需要制作工具类来转换Json类型读写

1. 导入POM
2. 导入JsonUtils工具类(将Object/List &lt;Object &gt;转化为字符串)
3. 导入Redis工具类接口/实现接口
4. 导入配置文件spring-redis/redis.properties
5. service中注入使用即可

**注意spring的加载机制:**

```
Caused by: java.lang.IllegalArgumentException: Could not resolve placeholder 'USER_LIST' in string value "${USER_LIST}"
```

Spring容器采用反射扫描的发现机制，在探测到Spring容器中有一个org.springframework.beans.factory.config.PropertyPlaceholderConfigurer的Bean就会停止对剩余PropertyPlaceholderConfigurer的扫描（Spring 3.1已经使用PropertySourcesPlaceholderConfigurer替代PropertyPlaceholderConfigurer了）

所以根据加载的顺序，配置的第二个property-placeholder就被没有被spring加载，我想引入的config-wxapp.properties就没有被引入，所以在使用@Value注入的时候占位符就解析不了...解决方法就是把2个property-placeholder注解配置合并在一起就好了 

