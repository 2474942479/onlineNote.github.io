# Redis

Redis是基于**单线程IO多路复用**的架构实现的NoSql，意味着它的操作都是串行化的，所以在命令操作上不会出现线程安全问题

业务实现：

> 1.  **分布式锁。**利用Redis的串行化特性，可以轻松的实现分布式锁，其中用到的命令有：**setnx key value** , **expire key time** ,**del key** ，其中第一个setnx是指在key不存在时能赋值成功，expire 来设置key的存活时间来防止程序异常而没有及时del到key值的情况。但是程序也有可能在expire没有执行时就已经挂掉的时候，这是可以来一个增强版**set key value NX EX time**。这里的NX就是表示if not exist，而Ex表示时间单位秒，Px代表毫秒。
> 2. **分布式session**。这里仅仅是利用Redis的数据库功能，把分布式应用的session抽取到Redis中，普通的get、set，命令即可完成。
> 3. **商品秒杀实现。**把需要销售的商品提前放入Redis，通过redis的**incr**和**decr**命令安全的增加和减少库存。
> 4. **限时验证。** expire key time ，判断exists key在短信验证时，当redis中存在数据则不允许再次请求验证发送。

## 常用五大数据类型

* String	字符串：是 redis 最基本的类型，是二进制安全的，可以包含任何数据，最大能存储 512MB。

  * ```java
    set college szu
    ```

* List  列表：List 是有序列表，这个可以玩儿出很多花样。

  *  list 存储一些列表型的数据结构，类似粉丝列表、文章的评论列表之类的东西。

  * 可以通过 lrange 命令，读取某个闭区间内的元素，可以基于 list 实现分页查询，这个是很棒的一个功能，基于 Redis 实现简单的高性能分页，可以做类似微博那种下拉不断分页的东西，性能高，就一页一页走。

    ```java
    # 0开始位置，-1结束位置，结束位置为-1时，表示列表的最后一个位置，即查看所有。
    lrange mylist 0 -1
    ```

  * 可以搞个简单的消息队列，从 list 头怼进去，从 list 尾巴那里弄出来。

    ```java
    lpush mylist 1
    lpush mylist 2
    lpush mylist 3 4 5
    
    # 1
    rpop mylist
    ```

* Set  集合： 无序无重复(去重)的String类型的集合

  * 基于 set 玩儿交集、并集、差集的操作
  * 多机器上的数据去重

  ```java
  #-------操作一个set-------
  # 添加元素
  sadd mySet 1
  
  # 查看全部元素
  smembers mySet
  
  # 判断是否包含某个值
  sismember mySet 3
  
  # 删除某个/些元素
  srem mySet 1
  srem mySet 2 4
  
  # 查看元素个数
  scard mySet
  
  # 随机删除一个元素
  spop mySet
  
  #-------操作多个set-------
  # 将一个set的元素移动到另外一个set
  smove yourSet mySet 2
  
  # 求两set的交集
  sinter yourSet mySet
  
  # 求两set的并集
  sunion yourSet mySet
  
  # 求在yourSet中而不在mySet中的元素
  sdiff yourSet mySet
  ```

* ZSet(Sorted set) 有序无重复的集合 有序的set

  ```
  zadd board 85 zhangsan
  zadd board 72 lisi
  zadd board 96 wangwu
  zadd board 63 zhaoliu
  
  # 获取排名前三的用户（默认是升序，所以需要 rev 改为降序）
  zrevrange board 0 3
  
  # 获取某用户的排名
  zrank board zhaoliu
  ```

* Hash  哈希类似java中的Map的一种结构.通常用来存储结构话的数据,例如 一个对象（前提是**这个对象没嵌套其他的对象**）给缓存在 Redis 里，然后每次读写缓存的时候，可以就操作 hash 里的**某个字段**。

  * ```java
    hset person name bingo
    hset person age 20
    hset person id 1
    hget person name
    (person = {
      "name": "bingo",
      "age": 20,
      "id": 1
    })
    ```

# Redis 过期策略

Redis 过期策略是：**定期抽查删除+惰性删除(调用了再检查是否过期)**。

所谓**定期删除**，指的是 Redis 默认是每隔 100ms 就随机抽取一些设置了过期时间的 key，检查其是否过期，如果过期就删除。

假设 Redis 里放了 10w 个 key，都设置了过期时间，你每隔几百毫秒，就检查 10w 个 key，那 Redis 基本上就死了，cpu 负载会很高的，消耗在你的检查过期 key 上了。注意，这里可不是每隔 100ms 就遍历所有的设置过期时间的 key，那样就是一场性能上的**灾难**。实际上 Redis 是每隔 100ms **随机抽取**一些 key 来检查和删除的。

但是问题是，定期删除可能会导致很多过期 key 到了时间并没有被删除掉，那咋整呢？所以就是惰性删除了。这就是说，在你获取某个 key 的时候，Redis 会检查一下 ，这个 key 如果设置了过期时间那么是否过期了？如果过期了此时就会删除，不会给你返回任何东西。

> 获取 key 的时候，如果此时 key 已经过期，就删除，不会返回任何东西。

但是实际上这还是有问题的，如果定期删除漏掉了很多过期 key，然后你也没及时去查，也就没走惰性删除，此时会怎么样？如果大量过期 key 堆积在内存里，导致 Redis 内存块耗尽了，咋整？

答案是：**走内存淘汰机制**。

# Redis 内存淘汰机制

 `Redis` 提供了一个参数 `maxmemory` 来配置 `Redis` 最大使用内存

```java
maxmemory <bytes>
```



`Redis` 中提供了 `8` 种淘汰策略，可以通过参数 `maxmemory-policy` 进行配置：

| 淘汰策略          | 说明                                                         |
| :---------------- | :----------------------------------------------------------- |
| `volatile-lru`    | 根据 LRU (**最近最少使用**)算法删除**设置了过期时间的键**，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `allkeys-lru`     | 根据 LRU 算法删除所有的键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `volatile-lfu`    | 根据 LFU(**最近不经常使用**) 算法删除设置了过期时间的键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `allkeys-lfu`     | 根据 LFU 算法删除所有的键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `volatile-random` | 随机删除设置了过期时间的键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `allkeys-random`  | 随机删除所有键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `volatile-ttl`    | 根据键值对象的 `ttl` 属性， 删除最近将要过期数据。如果没有，则直接报错 |
| `noeviction`      | 默认策略，不作任何处理，直接报错                             |

PS：淘汰策略也可以直接使用命令 `config set maxmemory-policy <策略>`来进行动态配置

# LUA脚本操作Redis

## 为什么用LUA脚本操作Redis

- 减少网络开销：本来5次网络请求的操作，可以用一个请求完成，原先5次请求的逻辑放在redis服务器上完成。使用脚本，减少了网络往返时延。
- **原子操作：Redis会将整个脚本作为一个整体执行，中间不会被其他进程或者进程的命令插入**。（最重要）
- 复用：客户端发送的脚本会永久存储在Redis中，意味着其他客户端可以复用这一脚本而不需要使用代码完成同样的逻辑。

## redis执行lua

### eval

使用[EVAL](http://doc.redisfans.com/script/eval.html#eval)命令对 Lua 脚本进行求值

**EVAL script numkeys key [key ...] arg [arg ...]**

> [info] numkeys ： keys的数量有几个。这是一个必传的参数，即使没有keys也要传个0；

```
#  注意redis的计数是从1开始的
127.0.0.1:6379> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```

## lua函数

主要有两个函数来执行redis命令  

​	redis.call() – 出错时返回具体错误信息,并且终止脚本执行  

​	redis.pcall() –出错时返回lua table的包装错误,但不引发错误 

## 对应类型

| redis数据类型 | lua数据类型      |
| :------------ | :--------------- |
| integer       | number           |
| bulk          | string           |
| multi bulk    | table            |
| status        | 包含ok域的table  |
| error         | 包含err域的table |
| nil bulk      | false            |

从redis数据类型到lua数据类型或者从lua数据类型到redis数据类型,都有以上对应规则,但是从

*从lua转换到redis有一条额外的对应规则*

- lua boolean true –> redis 1  即,lua的true对应redis 的整型1.



## Redis锁的LUA脚本

```erlang
if redis.call('EXISTS', KEYS[1])==1 
then 
	local curValue = redis.call('GET', KEYS[1])
	if string.find(curValue, val)==1 
	then 
		local curTtl = redis.call('TTL', KEYS[1]) 
		redis.call('EXPIRE', KEYS[1], curTtl + ttl) 
		return true 
	else 
		return false 
	end 
else 
    return false 
end
```



```java
if (redis.call('EXISTS', KEYS[1]) == 1) {
  local curValue = redis.call('GET', KEYS[1]));
  if (string.find(curValue, val)==1) {
    	local curTtl = redis.call('TTL', KEYS[1]) 
			redis.call('EXPIRE', KEYS[1], curTtl + ttl) 
			return true 
  } else {
    	return false
  	} 
} else {
  return false
}
  	
```



