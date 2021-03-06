# Redis 淘汰策略

 `Redis` 提供了一个参数 `maxmemory` 来配置 `Redis` 最大使用内存

```java
maxmemory <bytes>
```



`Redis` 中提供了 `8` 种淘汰策略，可以通过参数 `maxmemory-policy` 进行配置：

| 淘汰策略          | 说明                                                         |
| :---------------- | :----------------------------------------------------------- |
| `volatile-lru`    | 根据 LRU 算法删除设置了过期时间的键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `allkeys-lru`     | 根据 LRU 算法删除所有的键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `volatile-lfu`    | 根据 LFU 算法删除设置了过期时间的键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `allkeys-lfu`     | 根据 LFU 算法删除所有的键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `volatile-random` | 随机删除设置了过期时间的键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `allkeys-random`  | 随机删除所有键，直到腾出可用空间。如果没有可删除的键对象，且内存还是不够用时，则报错 |
| `volatile-ttl`    | 根据键值对象的 `ttl` 属性， 删除最近将要过期数据。如果没有，则直接报错 |
| `noeviction`      | 默认策略，不作任何处理，直接报错                             |

PS：淘汰策略也可以直接使用命令 `config set maxmemory-policy <策略>`来进行动态配置

