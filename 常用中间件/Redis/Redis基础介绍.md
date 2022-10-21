---

---

## Redis介绍

Redis是一个开源的使用C语言编写的key-value的单线程的存储系统，是跨平台的非关系型数据库。

## Redis基本数据类型内部编码

基础数据结构：

- 字符串String(3)种编码
- 哈希Hash(2)种编码
- 列表List(2)种编码
- 集合Set(2)种编码
- 有序集合Zset(2)种编码

### 字符串(String)

字符串是Redis种最简单的储存类型，它存储的值可以是字符串、整形或者浮点型，对整个字符串或者字符串其中的一部分执行操作对整数或者浮点数执行自增或者自减操作。

**字符串的基本操作**

- set key value  //存入字符串键值对

- mset key value [key value ....] //批量存储字符串键值对

- setnx key value   //存入一个不存在的字符串键值对

- get key      //获取一个字符串键值

- mget key [key ...]    //批量获取字符串键值

- del key       //删除一个键

- expoire  key  seconds   //设置一个键的过期时间(秒)

  **原子操作**

------

- incr  key  //将key中存储的数字值加1
- decr key  // 将key中存储的数字值减1
- incrby key increment   //将key所存储的值都加上increment
- decrby key decrement   //将key所存储的值减去decrement

### 哈希(Hash)

Redis hash 是一个String 类型的field(字段)和value(值)的映射表，hash特别适合用于存储对象。

**哈希的基本操作**

- Hset key field value   //存储一个哈希表的键值
- Hsetnx key field value   //存储一个不存在的哈希表的键值
- Hmset key field value [field value...]  //在一个哈希表中存储多个键值对
- Hget key field  //获取哈希表key对应的filed键值
- Hmget key field [field...]  //批量获取哈希表key中多个field键值
- Hdel key field [field...]  //删除哈希表key中的field键值
- Hlen key   //返回哈希表key中field的数量
- Hgetall key  //返回哈希表key中所有的键值
- Hincrby key field increment    //为哈希表key中的field键的值加上增量increment

### 列表（List)

Redis中的lists相当于Java中的LinkedList，实现原理是一个双向链表(其底层是一个快速列表)，即可以支持
反向查找和遍历，更方便操作，插入和删除操作非常快，时间复杂度为O(1)，但是索引定位很慢，时间复杂
度为O(n)  。

**列表的基本操作**

- lpush key value [value...]  //将一个或者多个值value插入到key列表的表头（最左边）
- rpush key value[value...]  //将一个或者多个值value插入到key列表的表尾（最右边）
- lpop key  //移除并返回key列表的头元素
- rpop key  //移除并返回key列表的尾元素
- blpop key [key...] timeout  //从key列表表头弹出一个元素，若列表中没有元素，阻塞等待
- brpop key [key...] timeout //从key列表表尾弹出一个元素，若列表中没有元素，阻塞等待timeout秒，如果timeout=0，则一直等待。
- lrange key start stop  //返回列表key中指定区间内的元素，区间以偏移量start和stop指定。

### 集合（Set）

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据  

**集合的常用操作**

- set常用操作

  ```java
  sadd key member [member...]  //往集合key中存入元素，元素存在则忽略，若key不存在则新建
  srem key member [member...]  //往集合key中删除元素
  smembers key   //获取集合key中所有元素
  scard  key   //获取集合key中元素个数
  sismember key  member //判断member元素是否存在于集合key中
  srandmember key [count]  //从集合中选出count个元素，元素不从key中删除
  spop key [count]  //从集合中选出count个元素，元素从key中删除
  ```

- Set运算操作

  

```java
sinter key [key...]  //交集运算
sintersotre destination key [key...]  //将交集结果存入新集合desttination中
sunion key [key...]  //并集计算
sunionstore destination key [key...]  //将并集存入新集合destination当中
sdiff key [key...]  //差集计算
sdiffsotre destination key [key...]   //将差集结果存入新集合destination当中
```



### 有序集合(Zset)

Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一
个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。有序集合的成员是唯
一的,但分数(score)却可以重复。  

```java
ZADD key score member [[score member]…]   //往有序集合key中加入带分值元素
ZREM key member [member …]   //从有序集合key中删除元素
ZSCORE key member   //返回有序集合key中元素member的分值
ZINCRBY key increment member   //为有序集合key中元素member的分值加上increment
ZCARD key    //返回有序集合key中元素个数
ZRANGE key start stop [WITHSCORES]    //正序获取有序集合key从start下标到stop下标的元素
ZREVRANGE key start stop [WITHSCORES] //倒序获取有序集合key从start下标到stop下标的元素
ZUNIONSTORE destkey numkeys key [key ...]   //并集计算
ZINTERSTORE destkey numkeys key [key …]   //交集计算
```

![QQ截图20220629174630](C:\Users\汤琛\Desktop\学习资料\常用中间件\Redis\images\QQ截图20220629174630.png)

## Redis存储原理深入剖析

**Redis数据存储原理**

redisDB

dict

dictht

dictEntry

![QQ截图20220630090307](C:\Users\汤琛\Desktop\学习资料\常用中间件\Redis\images\QQ截图20220630090307.png)

Key对存储效率的影响

dictEntry：16字节

RedisObject: 12字节

sds: 8字节+字符串长度

一个10个字节的key需要多少空间？

16+12+8+10+1=47

![QQ截图20220630090443](C:\Users\汤琛\Desktop\学习资料\常用中间件\Redis\images\QQ截图20220630090443.png)

## Redis高效率

![QQ截图20220630091038](C:\Users\汤琛\Desktop\学习资料\常用中间件\Redis\images\QQ截图20220630091038.png)
