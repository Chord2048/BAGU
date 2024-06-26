Redis 内存数据库 MySQL缓存

支持多种数据类型

持久化

集群

发布订阅

Lua脚本

事务





五种类型

string 字符串、整数或者浮点数

list 字符串的链表

set 无序集合

zset 有序存储键值对

hash tales 无序三列表





单线程：**CPU 并不是制约 Redis 性能表现的瓶颈所在**

IO多路复用

避免多线程竞争

都在内存中完成



### Redis持久化

**AOF日志** 每执行一条**写操作**命令，就把该命令以追加的方式写入到一个文件里；

​	先执行再写日志，不会阻塞和带来检查开销；但是数据可能丢失，或阻塞其他操作。

​	写回策略

- **Always**
- **Everysec**，每隔一秒将缓冲区里的内容写回到硬盘；
- **No** 由操作系统自己决定

**RDB快照** 将某一时刻的内存数据，以二进制的方式写入磁盘

**混合持久化**





## 缓存问题

### 缓存雪崩

大量缓存数据同时过期 新请求直接访问数据库

解决：时效时间加上一个随机值

设置缓存不过期，通过后台服务更新缓存



### 缓存击穿

热点数据过期，大量请求直接访问数据库。

* 互斥锁方案，保证同一时间只有一个业务请求缓存。

* 不给热点数据设置过期时间，或者热点过期前通知后台线程更新缓存重新设置过期时间



### 缓存穿透

访问的数据不在缓存中也不再数据库中

* **非法请求的限制，**  判断求请求参数是否合理，请求参数是否含有非法值、请求字段是否存在
* **设置空值或者默认值 ** 在缓存中设置空值
* 使用**布隆过滤器**



#### 热点数据动态缓存策略

热点数据动态缓存的策略总体思路：**通过数据最新访问时间来做排名，并过滤掉不常访问的数据，只留下经常访问的数据**。



以电商平台场景中的例子，现在要求只缓存用户经常访问的 Top 1000 的商品。具体细节如下：

- 先通过缓存系统做一个**排序队列**（比如存放 1000 个商品），系统会根据商品的**访问时间**，更新队列信息，**越是最近访问的商品排名越靠前**；
- 同时系统会**定期过滤掉队列中排名最后的 200 个商品**，然后再从数据库中**随机读取出 200 个商品加入队列中**；
- 这样当请求每次到达的时候，会先从队列中获取商品 ID，如果命中，就根据 ID 再从另一个缓存数据结构中读取实际的商品信息，并返回。



### 缓存更新策略

- **Cache Aside**（旁路缓存）策略； ： mysql 用的
- Read/Write Through（读穿 / 写穿）策略；
- Write Back（写回）策略；



Cache Aside

**写策略的步骤：**

- 先更新数据库中的数据，再**删除**缓存中的数据。

**读策略的步骤：**

- 如果读取的数据命中了缓存，则直接返回数据；
- 如果读取的数据没有命中缓存，则从数据库中读取数据，然后将数据写入到缓存，并且返回给用户。



**适合读多写少的场景，不适合写多的场景**



写多？

1. 更新数据时更新缓存
2. 给缓存一个短的过期时间



Read/Write Through（读穿 / 写穿）策略 客户端只和缓存交互。

Write Back（写回）策略。更新数据的时候，只更新缓存，同时将缓存数据设置为脏的，然后立马返回。适合写多的场景。但是有数据不一致和丢失的风险。





### RDB

save & bgsave 阻塞/fork一个子进程

