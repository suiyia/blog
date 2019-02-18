title: Redis 简单介绍与 Jedis 常用操作
author: caijicoder
tags:
  - redis
categories:
  - redis
date: 2019-02-18 21:02:00
toc: true 
---

### 介绍
> Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 

主要对 [Redis 中文网](http://www.redis.cn/) 文档内容进行总结，并使用 Jedis 实现一些基本操作。

### 使用 Jedis 实现 Redis 基本操作
[Jedis](https://github.com/xetorthio/jedis) 是一个小而精的 Redis 客户端，用 Java 实现
- strings 操作
```java
String result = jedis.set("name","chx"); // 添加 String ，result = "OK"
result = jedis.mset("name", "chenhaoxiang", "age", "20", "email", "chxpostbox@outlook.com"); // 同时设置多个键值对 result = "OK"
Boolean exists = jedis.exists("name"); // 是否存在 key=user 的记录
result = jedis.get("name"); // 获取数据，result = "chx"
Long appendres = jedis.append("name", " is my name;"); // 拼接，appendres = 15，拼接后字符串长度
Long delres = jedis.del("name");  //删除某个键值对， delres = 1
//键对应 value 的自增操作，具备原子性，如果键包含错误类型的值或包含无法表示为整数的字符串，则会返回错误。此操作限于64位有符号整数。ERR value is not an integer or out of range
//如果键不存在，则在执行操作之前将其设置为0。如果 age 不存在，操作后 age = 1；
Long incrres = jedis.incr("age");  // 用于将键的整数值递增1。 incrres = 21，递增后的值。
```
- lists 操作
```java
Long res1 = jedis.lpush("list","1"); // 头部插入一个元素，res1 = 1
res1 = jedis.lpush("list","2"); // res1 = 2
res1 = jedis.lpush("list","3"); // res1 = 3 返回该 List 的长度
jedis.rpush("list","5"); // 尾部插入元素
Long llen = jedis.llen("list"); // List 长度，llen = 4
List<String> list3 = jedis.lrange("list",0,-1); // list3 = [3,2,1,5] 按范围取出,第一个是key，第二个是起始位置，第三个是结束位置
```
- sets 操作
```java
res1 = jedis.sadd("set","1");
res1 = jedis.sadd("set","2");
res1 = jedis.sadd("set","3"); // 添加
Long srem = jedis.srem("set","2"); // 移除某个元素
Set<String> set1 = jedis.smembers("set"); // 获取 key=set 的 Set
Boolean sismember = jedis.sismember("set","1"); // key=set 的 Set 中是否存在元素 "1"
String srandmember = jedis.srandmember("set"); // 随机返回一个 set 元素
List<String> list4 = jedis.srandmember("set",2); // 随机返回指定个数个元素
Long scar = jedis.scard("set"); // set 的元素个数
```
- sorted sets 操作( TODO 用到再更新)
```java
jedis.zadd("zset",1,"1");
jedis.zadd("zset",2.0,"2");
jedis.zadd("zset",3,"3");
```

- Hashes 操作
```java
Map<String, String> map = new HashMap<String, String>();
map.put("name", "chx");
map.put("age", "100");
map.put("email", "***@outlook.com");
result = jedis.hmset("user", map); // 添加 Map，result = "OK"
String res = jedis.hget("user","age"); // 获取 Map 指定 key 的 value，只能获取单个 key，res = 100
List<String> list = jedis.hmget("user", "name", "age", "email"); // 获取 Map 指定 key 的 value，同时指定多个 key，list = [chx, 100, ***@outlook.com];
Long hdel = jedis.hdel("user", "age"); //删除 map 中的某个键值，hdel = 1
Long hlen = jedis.hlen("user"); // 获取 map 中的键值对个数，hlen = 2
Set<String> set = jedis.hkeys("user"); // 返回 map 中的所有 key
Iterator<String> iterator = jedis.hkeys("user").iterator(); // 迭代遍历
while (iterator.hasNext()){
    String key = iterator.next();
    String value = jedis.hmget("user",key).get(0);
}
List<String> list2 = jedis.hvals("user"); // 返回 map 中的所有 value
```

### Redis 其它名词
- **expire 过期**

Redis 允许为每一个 key 设置不同的过期时间，当它们到期时将自动从服务器上删除。

主动删除：client 主动访问，发现过期，立即删除

被动删除：Redis **定时随机**选择一些 key 进行检测，删除过期的 key，如果删除比例高于 25%，则继续选择一些 key 进行删除

- **管道(Pipelining)**

Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。
发送一个命令之后只有等待服务器返回之后才会执行后续命令。

管道则可以一次发送多条命令而不必立即等待服务器返回，收到命令之后，服务器会以队列形式回复命令，管道操作比较耗内存，要注意命令的大小。

```java
Pipeline pipeline = jedis.pipelined();
for (int i = 0; i < 100; i++) {
    pipeline.set("1","1");
}
pipeline.sync();
```
[Redis学习笔记7--Redis管道（pipeline）](https://blog.csdn.net/freebird_lb/article/details/7778919)

[分布式缓存Redis之Pipeline（管道）](https://blog.csdn.net/u011489043/article/details/78769428)


- **事务**

将一组 Redis 命令放到事务中执行，MULTI 、 EXEC 、 DISCARD 和 WATCH 是 Redis 事务相关的命令。

```java
String watch = jedis.watch("123");
Transaction multi = jedis.multi();
multi.set("123","123");
List<Object> list1 = multi.exec();
multi.discard();
jedis.unwatch();
```

multi 用于开启一个事务，exec 执行事务，watch 用于监测事务中 key 的变化，如果 key 被其它客户端改过了，那么整个事务会被取消，discard 用于取消事务

### 总结
这个总结主要是了解下 Jedis 常用操作。具体细节学习需要文档、项目结合学习。

博客参考：
[【Redis】Java中使用Jedis操作Redis(Maven导入包)、创建Redis连接池](https://blog.csdn.net/qq_26525215/article/details/60466222)