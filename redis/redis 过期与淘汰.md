## redis 过期与淘汰

过期策略：

定时删除+懒惰删除+定期删除

定时删除：

在设置键的过期时间的同时，创建一个定时器，让定时器在键的过期时间来临时，立即执行对键的删除操作。

1. 优点：对内存非常友好
2. 缺点：对CPU时间非常不友好

懒惰删除：

放任过期键不管，每次从键空间中获取键时，检查该键是否过期，如果过期，就删除该键，如果没有过期，就返回该键。

1. 优点：对CPU时间非常友好
2. 缺点：对内存非常不友好

定期删除：

每隔一段时间，程序对数据库进行一次检查，删除里面的过期键，至于要删除哪些数据库的哪些过期键，则由算法决定。

周期会对cpu、内存的友好程度有影响。

### 内存淘汰策略

noeviction：当内存不足以容纳新写入数据时，新写入操作会报错。**默认策略**

allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。

allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。

volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。

volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。

volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。



